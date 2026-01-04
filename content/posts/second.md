+++
date = '2025-12-12T21:04:52+08:00'
draft = false
title = 'Netdisk项目技术说明（面向实现）'
toc = true
tocBorder = true
tags = ["项目说明", "对象存储", "Spring Boot", "MinIO", "Kafka"]
+++
## 技术栈（固定约定）

* 后端：Spring Boot4 (Java 17), Maven
* MongoDB：文件元数据 / 目录树
* MySQL：用户 / 权限 / 强一致业务（可选）
* Redis：缓存、Token、进度、锁、队列
* 存储：MinIO (S3 API) / 本地磁盘（开发）
* 异步队列：Redis Streams / Kafka
* 运维：Docker Compose / Kubernetes，Prometheus + Grafana，ELK/Loki 日志

---

## 主数据模型（MongoDB 文档示例）

每个文件或目录一条文档（`files` 集合）：

```json
{
  "_id": ObjectId("..."),
  "fileId": "uuid-1234",
  "userId": "uid1",
  "parentId": "uuid-parent",      // "root" 表示根目录
  "name": "简历.pdf",
  "type": "FILE",                 // FILE | DIR
  "size": 102400,
  "hash": "md5sha1",
  "storagePath": "s3://bucket/objpath",
  "status": "MERGED",             // UPLOADING | MERGING | MERGED | DELETED
  "chunks": {
    "total": 256,
    "chunkSize": 4*1024*1024
  },
  "meta": { ... },                // tags, version 等扩展字段
  "createdAt": ISODate,
  "updatedAt": ISODate
}
```

**索引建议**

* `{"userId":1, "parentId":1}` — 目录列出（最常用）
* `{"fileId":1}` — 快速定位
* `{"hash":1}` — 秒传查询（唯一或非唯一视需求）
* TTL 索引用于临时分享/临时状态集合（如果有）

---

## Redis Key 设计（前缀 + 用途）

```
login:token:{token}        -> userId (TTL e.g. 30m)
file:hash:{md5}            -> fileId
upload:progress:{fileId}   -> BITMAP or SET of chunkIndex
lock:file:merge:{fileId}   -> lock owner (use Redisson/RLock)
download:token:{uuid}      -> fileId (TTL e.g. 5m)
user:list:{userId}:files   -> cached paged lists (optional)
upload:queue               -> task queue (list/stream) for merges
```

**Bitmap vs Set**

* **Bitmap**（`SETBIT/GETBIT/BITCOUNT`）适合连续且上限确定的 chunkIndex：内存低、判断是否完成 O(1)（通过 BITCOUNT）。
* **Set**（`SADD/SCARD`）适合稀疏或 chunkIndex 无上限场景，但内存更高。

参数建议：默认 chunkSize = 4MB, maxChunks ~ 65536 => Bitmap 合理。

---

## API 设计（最核心）

（仅列关键接口）

1. `POST /api/upload/check`
   Input: `{ fileName, fileSize, fileHash, chunkSize }`
   Logic:

   * 查 `file:hash:{fileHash}` in Redis -> if exists and storage exists -> return `SECONDPASS` + fileId
   * Else create/ensure `FileDocument` in Mongo with status `UPLOADING`, return `fileId` + `chunkSize` + `expectedChunks`.

2. `POST /api/upload/chunk`
   Multipart/bytes: `{fileId, chunkIndex, chunkHash, chunkBytes}`
   Logic:

   * Validate token/auth
   * Verify chunkHash
   * Save chunk to MinIO at `/{fileId}/chunk_{index}` (或临时本地)
   * Mark Redis: `SETBIT upload:progress:{fileId} {chunkIndex} 1` (或 `SADD`)
   * If BITCOUNT == totalChunks -> push merge task to queue
   * Return success

3. `POST /api/upload/merge` (can be invoked by client or worker)
   Input: `{fileId}`
   Logic:

   * Try acquire `lock:file:merge:{fileId}` (Redisson RLock recommended)
   * Validate all chunks present (Bitmap/Set check)
   * Merge stream from MinIO chunks into final object (server-side compose if storage supports; otherwise stream and write)
   * Compute final hash & verify
   * Update `files` doc: `status=MERGED`, `storagePath=...`, delete chunk objects and Redis progress key
   * Release lock, produce event/notification

4. `GET /api/files?parentId=` — list directory by `userId` + `parentId` using index.

5. `POST /api/files/copy`
   Input: `{sourceId, targetParentId}` — see “拷贝实现”下文。

---

## 分片上传与合并（重点实现细节）

### 1. 前端约定

* 客户端先计算 `fileHash`（MD5/SHA1）并 `POST /check`
* 根据 `chunkSize` 切片并并发上传 `chunk`，并在每片完成后上报

### 2. 服务端存储策略

* 分片直接存对象存储（路径 `{fileId}/chunk_{index}`）避免后端 I/O 瓶颈
* 对象存储若支持 server-side compose（MinIO 不直接有合并 API，但可用 S3 Multipart Complete 或在 server 端合并流）

### 3. 进度判断（示例：Bitmap）

* 写入：`SETBIT upload:progress:{fileId} {index} 1`
* 判断完成：`BITCOUNT upload:progress:{fileId}` == `expectedChunks`
* 原子性：单片上传写入后立即检查 BITCOUNT；若等于 expected -> submit merge task

### 4. 合并注意点

* 使用 **分布式锁** 保证只有一个 worker/实例在合并（Redisson 推荐）
* 合并必须幂等：检查 `status` 字段，若 `MERGED` 直接返回
* 合并失败要可重试（任务记录中加 `attempts`, backoff）
* 合并完成后删除 chunk objects 或按策略保留短期备份

### 5. 秒传的双重校验

* Redis `file:hash:{md5}` 命中后，仍需验证 MongoDB `files` 有对应 `storagePath` 且对象存储可读取（避免缓存污染）

---

## 目录树、列出、重命名、拷贝（重点：多层拷贝实现）

### 存储表示

* 父引用模型（`parentId`）是主方案：简单、查询快，支持 pagination
* 对于需要“一次拿到整棵树”的操作：用 Mongo `$graphLookup`（聚合）或递归查询（应用端）

### 拷贝目录（算法，伪码）

1. 获取源目录的所有后代（`descendants = graphLookup(sourceId)`）按层级排序（从上到下或 BFS）
2. 构建 ID 映射 `oldId -> newId`（新 UUID）
3. 对每个节点按原来的顺序插入新文档：

   * new.parentId = (old.parentId == sourceId) ? targetParentId : idMap[old.parentId]
   * 对文件节点不复制物理对象，直接指向同一 `storagePath`（实现为“软拷贝”）
   * 如果需要“物理拷贝”，则对对象存储执行 copyObject（代价大）
4. 批量写入（分批提交）保证效率
5. 返回新根 id

**复杂度与事务**

* 大树复制可能很大，采用分批且异步完成（返回任务 id，异步 worker 完成）
* 幂等：记录任务 id 并用 `status` 字段避免重复执行

**避免环与验证**

* 移动时需校验 `targetParentId` 不是 `source` 的子孙（避免环）

---

## 并发控制与幂等性（关键点）

* **幂等**：所有写操作加 `opId` 或检查 `status`（例如合并 `status != MERGED` 才允许）
* **锁**：使用 Redisson `RLock`（支持超时与自动续租）
* **Redis 超时策略**：关键短生命周期 Key 需配置合理 TTL（例如 token 30m、download token 5m、upload progress 可长期存在直到合并完成）
* **异常回滚**：合并中断要能恢复：合并前状态写为 `MERGING` + `mergeTaskId`，worker 恢复时可检测并继续

---

## MongoDB vs MySQL 分工（决策）

* **MongoDB**：文件元数据、目录树、灵活扩展字段、快速目录查询（推荐）
* **MySQL**：用户表、权限、计费、强一致业务、审计日志（推荐）
* 可采用混合存储：用户/权限在 MySQL，文件与目录在 MongoDB。业务跨库需要谨慎设计（尽量弱耦合，异步同步必要信息）。

---

## 迁移策略（从原模板）

* 阶段化：先引入 Mongo 做新功能（双写策略或同步层），验证后逐步切换查询路由
* 数据迁移脚本：批量读取旧 MySQL 表，生成 Mongo 文档（map oldId->newId）
* 回退：短期保留 MySQL 副本，确保回退路径

---

## 测试与压测（要点）

* 单元测试：controller/service/repository 层
* 集成测试：in-memory Mongo / Redis，或 docker-compose 环境
* 并发压测：k6 / JMeter，重点场景：

  * 并发分片上传（并发数 100~1000 测试）
  * 并发秒传判断
  * 多并发合并任务（确保锁）
* 故障注入场景：Redis 宕机、MinIO 权限错误、网络延迟

---

## 监控与运维（简要）

* 指标：上传速率、合并失败率、Redis HIT/MISS、Mongo QPS、对象存储延迟
* 警告：合并失败率阈值、Redis RTT 升高、Mongo 索引扫描过多
* 备份：Mongo 定期备份（rsync / mongodump），MySQL binlog，MinIO 对象生命周期与异地备份

---

## 部署建议（开发 → 测试 → 生产）

* 开发：docker-compose (app, mongodb, redis, minio)
* 测试/生产：Kubernetes + Horizontal Pod Autoscaler，Redis 主从或托管服务，Mongo 副本集
* CI/CD：自动 build + 测试 + 镜像推送 + 自动部署（GitHub Actions / GitLab CI）

---

## 关注的坑与防范（必须说明）

* Redis 缓存污染（秒传命中但对象被删除）——二次校验对象存在
* 合并幂等与重复消费——必须用锁 + status + opId
* 大文件复制（物理复制开销大）——优先软拷贝（引用同 storagePath）
* Bitmap 大小评估不当导致内存浪费——估算 maxChunks 与动态切换策略
* 跨库一致性（MySQL 与 Mongo）——避免同步强依赖，使用事件最终一致性

---

## 最小实现优先级（短任务清单）

1. 环境：Docker Compose (Mongo, Redis, MinIO, app)
2. 基本 CRUD：Mongo 文件文档 + list by parentId
3. 上传 check + chunk upload (单片上传、保存 chunk、Redis 标记)
4. 合并 worker + Redisson 锁 + status 更新
5. 秒传：file:hash Redis 记录 + 二次校验
6. 目录拷贝：实现异步任务，软拷贝元数据
7. 限流与 token、临时下载链接
8. 监控与压测
