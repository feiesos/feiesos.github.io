+++
date = '2026-01-09T14:55:45+08:00'
draft = false
title = 'Week02'
toc = true
tocBorder = true
+++
# 核心基础

## Hutool和Hutool常用工具集
Hutool是一个小而全的国产开源Java工具类库，通过静态方法封装，降低相关API的学习成本，提高工作效率，使Java拥有函数式语言般的优雅

### 包含组件
一个Java基础工具类，对文件、流、加密解密、转码、正则、线程、XML等JDK方法进行封装，组成各种Util工具类，同时提供以下组件：

| 模块名称 | 功能介绍 |
| :--- | :--- |
| hutool-aop | JDK 动态代理封装，提供非 IOC 下的切面支持 |
| hutool-bloomFilter | 布隆过滤，提供一些 Hash 算法的布隆过滤 |
| hutool-cache | 简单缓存实现 |
| hutool-core | 核心，包括 Bean 操作、日期、各种 Util 等 |
| hutool-cron | 定时任务模块，提供类 Crontab 表达式的定时任务 |
| hutool-crypto | 加密解密模块，提供对称、非对称和摘要算法封装 |
|hutool-db | JDBC 封装后的数据操作，基于 ActiveRecord 思想 |
| hutool-dfa | 基于 DFA 模型的多关键字查找 |
| hutool-extra | 扩展模块，对第三方封装（模板引擎、邮件、Servlet、二维码、Emoji、FTP、分词等） |
| hutool-http | 基于 HttpUrlConnection 的 Http 客户端封装 |
| hutool-log | 自动识别日志实现的日志门面 |
| hutool-script | 脚本执行封装，例如 Javascript |
| hutool-setting | 功能更强大的 Setting 配置文件和 Properties 封装 |
| hutool-system | 系统参数调用封装（JVM 信息等） |
| hutool-json | JSON 实现 |
| hutool-captcha | 图片验证码实现 |
| hutool-poi | 针对 POI 中 Excel 和 Word 的封装 |
| hutool-socket | 基于 Java 的 NIO 和 AIO 的 Socket 封装 |
| hutool-jwt | JSON Web Token (JWT) 封装实现 |

可以根据需要对每个模块进行单独引入，也可以通过引入`hutool-all`方式引入所有模块

### 安装
- Maven
    在项目的pom.xml的dependencies中加入以下内容：
    ```xml
    <dependency>
        <groupId>cn.hutool</groupId>
        <artifactId>hutool-all</artifactId>
        <version>5.8.16</version>
    </dependency>
    ```
- Gradle
    ```properties
    implementation 'cn.hutool:hutool-all:5.8.16'
    ```

### 具体使用
参考[`官方文档`](https://hutool.cn/docs/)中**Hutool-core**部分

---


## MySQL索引设计规范
### 索引设计核心原则
1. 必要性原则
- 仅为**查询**、**排序**、**分组频繁**的字段创建索引，禁止为低频查询字段（如一年仅查询几次的归档字段）建索引
- 索引需匹配业务查询场景，避免 “为建而建”（如仅存在UPDATE/DELETE条件但无查询的字段无需索引）
2. 适度性原则
- 单表索引数量**不超过5个**（除非表数据量极小且查询复杂），过多索引会导致：
    - 写入/更新性能下降（每次变更需同步维护索引树）
    - MySQL 优化器选择索引的计算成本升高，可能选错索引
3. 平衡性原则
- **查询密集表**（如订单查询表、用户信息表）可适当增加索引（≤8 个）
- **写入密集表**（如日志表、消息表）索引宜少（≤3 个），优先保证写入性能
- 大表（100 万行以上）索引需更严格控制，避免索引维护开销过大
4. 时效性原则
- 定期（如每季度）清理无效索引（长期未使用、冗余索引），避免索引膨胀
### 索引类型设计规范
1. 主键索引（PRIMARY KEY）
- **强制要求**：每张表必须设置主键，且主键字段禁止为 NULL
- **类型选择**：
    - 优先使用**自增整数型**（INT/BIGINT），优势：
        - 连续写入，减少索引页分裂
        - 索引树结构紧凑，查询效率高
    - 禁止使用UUID/字符串作为主键（离散写入导致大量页分裂，性能下降 50%+）
    - 禁止使用业务字段（如订单号、用户手机号）作为主键（业务变更可能导致主键修改）
- **特性约束**：主键字段禁止更新，避免破坏索引树结构
2. 普通索引（INDEX）
- **适用场景**：单字段查询（如WHERE status = 1）、排序（如ORDER BY create_time）、分组（如GROUP BY user_id）
- **命名规范**：idx_字段名（如idx_status、idx_create_time）
- **设计要点**：
    - 字段长度较长时（如VARCHAR(100)），评估是否必要（索引存储成本高）
    - 避免对高频更新字段（如last_login_time）建普通索引（更新会触发索引树重建）
3. 联合索引
- **适用场景**：多字段组合查询（如WHERE user_id = 100 AND status = 2）
- **最左前缀原则**：查询条件需包含联合索引的左侧字段，否则索引失效例如：
    - 联合索引(a, b, c)可匹配：
        - WHERE a = 1
        - WHERE a = 1 AND b = 2
        - WHERE a = 1 AND b = 2 AND c = 3
    - 不可匹配：
        - WHERE b = 2（缺少最左字段a）
        - WHERE a = 1 AND c = 3（跳过b，仅a部分生效）
- **字段顺序规则**：
    1. **过滤性优先**：将过滤性强的字段放在左侧（过滤性 = 字段去重后的值数量/总记录数，如手机号 > 性别）
    2. **查询频率优先**：常作为单字段查询的字段放在左侧（可复用索引，如(user_id, status)可同时支持WHERE user_id = ?和WHERE user_id = ? - AND status = ?）
    3. **长度优先**：短字段放左侧（索引树更紧凑，如(status, username)优于(username, status)）
- **命名规范**：idx_字段1_字段2_字段3（如idx_userid_status）
- **冗余避免**：已有联合索引(a, b)时，无需单独创建(a)索引（联合索引前缀可被复用）
4. 唯一索引（UNIQUE）
- **适用场景**：保证字段唯一性（如手机号、邮箱、订单号），替代业务层校验（数据库层面更可靠）
- **命名规范**：uniq_字段名（单一字段）或uniq_字段1_字段2（联合唯一）
- **逻辑删除场景适配**（核心补充）：
    - 若表使用逻辑删除（如is_deleted字段，0 = 未删，1 = 已删），**唯一索引必须包含删除标记**，形成联合唯一索引，避免 “已删除记录阻塞新记录插入”
    - 示例：用户表phone字段需唯一且支持逻辑删除，应创建：
        > ```sql
        > CREATE UNIQUE INDEX uniq_phone_deleted ON user (phone, is_deleted);
        > ```
        - 原理：已删除记录（is_deleted=1）与新记录（is_deleted=0）的索引值不同（(phone,1) vs (phone,0)），允许插入同手机号的新记录，同时保证未删除记录的唯一性
    - 禁忌：禁止仅对业务字段建唯一索引（如UNIQUE INDEX uniq_phone (phone)），否则删除的记录会导致新记录无法插入
- **特殊场景**：若业务要求 “即使记录被删除，也不允许再次插入同值数据”（如身份证号），则无需包含is_deleted，保持单一字段唯一索引
5. 前缀索引
- **适用场景**：长字符串字段（如VARCHAR(255)、TEXT前 N 位），需加速前缀匹配查询（如WHERE title LIKE 'Java%'）
- **设计方法**：
    - 通过SELECT COUNT(DISTINCT LEFT(字段名, N))/COUNT(*) FROM 表名计算前缀选择性，N 取 “选择性接近 1 且长度最小” 的值（通常 20-50 字符）
    - 示例：对title字段建前缀索引：
        > ```sql
        >    CREATE INDEX idx_title_prefix ON article (title(30)); -- 取前30字符
        > ```
- **限制**：前缀索引无法用于ORDER BY、GROUP BY，也无法通过=精确匹配（仅支持前缀匹配）

### 索引设计禁忌
1. 低价值索引禁止
- **低基数字段**：禁止对基数 < 10 的字段建索引（如性别gender（男/女）、状态status（0/1/2）），索引选择性低，查询可能全表扫描更快
- **高频更新字段**：避免对更新频率 > 查询频率的字段建索引（如last_modify_time），每次更新会触发索引树重构，影响性能
- **大字段全索引**：禁止对TEXT、BLOB类型字段建全字段索引（存储成本极高），如需索引必须用前缀索引
2. 索引失效场景避免
- **函数/表达式操作**：索引字段参与函数或表达式运算会导致索引失效，例如：
    - 错误：`WHERE SUBSTR(phone, 1, 3) = '138'`（函数操作）
    - 错误：`WHERE create_time + INTERVAL 1 DAY > NOW()`（表达式）
    - 正确：`WHERE phone LIKE '138%'`（前缀匹配，可命中索引）
- **隐式类型转换**：字段类型与查询值类型不匹配会触发转换，导致索引失效，例如：
    - 错误：`WHERE phone = 13800138000`（phone为VARCHAR，值为数字）
    - 正确：`WHERE phone = '13800138000'`（类型一致）
- **负向查询**：NOT IN、!=、<>、NOT EXISTS、IS NOT NULL通常无法使用索引，建议重构为正向查询，例如：
    - 错误：`WHERE status != 0`
    - 正确：`WHERE status IN (1, 2, 3)`（若状态值有限）
- **模糊查询前缀** wildcard：LIKE '%xxx'或LIKE '%xxx%'会导致索引失效，仅LIKE 'xxx%'可命中前缀索引，例如：
    - 错误：`WHERE username LIKE '%张三'`
    - 正确：`WHERE username LIKE '张三%'`
3. 重复与冗余索引禁止
- **重复索引**：同一**字段**/**字段组合**创建多个索引（如同时存在idx_a和idx_a），需删除重复项
- **冗余索引**：联合索引已包含的前缀字段无需单独建索引（如已有(a, b)，则(a)为冗余索引）

### 索引维护与优化规范
1. 索引监控
- **工具使用**：
    - 通过SHOW INDEX FROM 表名查看索引信息，关注Cardinality（基数），基数越接近表行数，索引选择性越好
    - 利用慢查询日志（slow log）和EXPLAIN分析查询计划，识别未使用的索引（type: ALL表示全表扫描）
    - MySQL 8.0 + 可通过sys.schema_unused_indexes视图直接查询未使用的索引
- **监控指标**：
    - 索引使用率（使用次数/总查询次数）：低于 10% 的索引需评估删除
    - 索引维护成本（写入时索引更新耗时）：写入频繁且维护成本高的索引优先删除
2. 索引优化操作
- **无效索引清理**：对连续 3 个月未使用的索引，经业务确认后删除（删除前备份表结构）
- **索引重建**：当索引碎片率 > 30%（SHOW TABLE STATUS LIKE '表名'查看Data_free），执行：
> ```sql
> ALTER TABLE 表名 FORCE INDEX (索引名); -- 重建指定索引
> -- 或重建全表索引（适用于MyISAM，InnoDB建议用OPTIMIZE TABLE）
> OPTIMIZE TABLE 表名;
> ```
- **大表索引操作**：
- 表数据量 > 100 万行时，创建/删除索引需在业务低峰期执行
- MySQL 8.0 + 使用ALTER TABLE ... ALGORITHM=INPLACE减少锁表时间（避免全表锁）
- 批量导入数据前，先删除索引，导入后重建（减少索引维护开销）

### 典型场景索引设计示例
1. 用户表
user，含逻辑删除
| 字段名 | 类型 | 索引设计 | 说明 |
| :--- | :--- | :--- | :--- |
| create_time | DATETIME | INDEX idx_createtime (create_time) | 用于按注册时间排序、筛选 |
| id | BIGINT | PRIMARY KEY | 自增主键，唯一标识用户 |
| is_deleted | TINYINT | (无单独索引，包含在联合唯一索引中) | 逻辑删除标记 (0 = 未删，1 = 已删) |
| phone | VARCHAR(20) | UNIQUE INDEX uniq_phone_deleted (phone, is_deleted) | 同上，保证手机号唯一 |
| user_type | TINYINT | INDEX idx_usertype (user_type) | 用于按用户类型筛选（如会员/非会员） |
| username | VARCHAR(50) | UNIQUE INDEX uniq_username_deleted (username, is_deleted) | 联合唯一索引，适配逻辑删除 |

2. 订单表
order
| 字段名 | 类型 | 索引设计 | 说明 |
| :--- | :--- | :--- | :--- |
| id | BIGINT | PRIMARY KEY | 自增主键 |
| order_no | VARCHAR(32) | UNIQUE INDEX uniq_orderno (order_no) | 订单号唯一，无逻辑删除需求 |
| user_id | BIGINT | INDEX idx_userid_createtime (user_id, create_time) | 支持“查询用户的订单并按时间排序” |
| status | TINYINT | INDEX idx_status_createtime (status, create_time) | 支持“查询特定状态的订单并按时间筛选” |
| pay_time | DATETIME | INDEX idx_paytime (pay_time) | 用于按支付时间统计/筛选 |

### 索引设计检查清单
1. 每张表是否设置自增整数型主键？
2. 单表索引数量是否超过5个（特殊场景小于等于8个）？
3. 联合索引是否遵循**最左前缀原则**，字段顺序是否合理？
4. 逻辑删除表的唯一索引是否包含is_deleted字段？
5. 是否存在**低基数字段**（如性别）、**高频更新字段**的索引？
6. 是否存在**重复索引**或**冗余索引**？
7. 索引是否被有效使用（通过EXPLAIN验证，type为ref/range/const）？

---

## SQL编写规范
### 基础格式规范
1. 关键字与函数
- 所有SQL关键字（如SELECT、INSERT、UPDATE、WHERE）和内置函数（如COUNT、DATE_FORMAT）必须大写，提升可读性
    ```sql
    -- 正确
    SELECT id, username FROM user WHERE status = 1;
    
    -- 错误
    select id, username from user where status = 1;
    ```
- 关键字和函数名后不加空格，与参数之间用空格分隔（如COUNT(id)而非COUNT(id)）
2. 换行与缩进
- 一条SQL语句过长（超过120字符）时，按逻辑拆分换行：
    - SELECT子句：每个字段单独换行，逗号置于行尾
    - FROM、JOIN、WHERE、GROUP BY、ORDER BY 等子句单独换行，且缩进4个空格
        ```sql
        -- 正确
        SELECT
            u.id,
            u.username,
            o.order_no,
            o.create_time
        FROM
            user u
        INNER JOIN
            `order` o ON u.id = o.user_id
        WHERE
            u.status = 1
            AND o.create_time >= '2025-01-01'
        ORDER BY
            o.create_time DESC;
        
        -- 错误（单行过长，可读性差）
        SELECT u.id, u.username, o.order_no, o.create_time FROM user u INNER JOIN `order` o ON u.id = o.user_id WHERE u.status = 1 AND o.create_time >= '2025-01-01' ORDER BY o.create_time DESC;
        ```
- 子查询需用括号包裹，并在括号内缩进（与外层保持一致）：
    ```sql
    SELECT
        t.user_id,
        t.order_count
    FROM (
        SELECT
            user_id,
            COUNT(*) AS order_count
        FROM
            `order`
        GROUP BY
            user_id
    ) t
    WHERE
        t.order_count > 5;
    ```
3. 命名规范
- **表名**/**字段名**：使用小写字母，下划线分隔（如user_info、order_no），禁止使用驼峰命名（如userInfo）
- **别名**：简洁明了，表别名用**1-2个小写字母**（如u代表user，o代表order），列别名用有意义的名称（如order_count而非c）
- **关键字冲突处理**：若**表名**/**字段名**与SQL关键字冲突（如order、user），必须用反引号`` ` ``包裹：
    ```sql
    -- 正确
    SELECT * FROM `order` WHERE `status` = 1;
    
    -- 错误（未处理关键字冲突）
    SELECT * FROM order WHERE status = 1;
    ```

### 查询语句（SELECT）规范
1. 字段查询
- 禁止使用`SELECT *`，必须明确指定所需字段（避免返回冗余数据，减少 IO 开销，且防止表结构变更导致的异常）
    ```sql
    -- 正确
    SELECT id, username FROM user;
    
    -- 错误
    SELECT * FROM user;
    ```
- 避免重复查询相同字段（如`SELECT id, id FROM user`）
2. WHERE条件
- 优先使用`=`、`IN`、`BETWEEN` 等高效条件，减少`OR`、`NOT IN`的使用（可能导致全表扫描）
    ```sql
    -- 推荐
    WHERE status IN (1, 2)
    WHERE create_time BETWEEN '2025-01-01' AND '2025-12-31'
    
    -- 不推荐（可用IN替代）
    WHERE status = 1 OR status = 2
    WHERE create_time >= '2025-01-01' AND create_time <= '2025-12-31'
    ```
- 逻辑删除表必须包含删除标记条件（如is_deleted = 0），避免查询到已删除数据：
    ```sql
    -- 正确
    SELECT id FROM user WHERE is_deleted = 0 AND status = 1;
    
    -- 错误（未过滤已删除数据）
    SELECT id FROM user WHERE status = 1;
    ```
3. JOIN操作
- 明确指定JOIN类型（INNER JOIN、LEFT JOIN等），禁止省略（如用JOIN代替INNER JOIN可能导致歧义）
- LEFT JOIN时，右表的过滤条件需放在`ON`后，左表的过滤条件放在`WHERE`后：
    ```sql
    -- 正确（右表过滤放 ON，左表过滤放 WHERE）
    SELECT u.id, o.order_no
    FROM user u
    LEFT JOIN `order` o 
        ON u.id = o.user_id 
        AND o.status = 1  -- 右表条件：只关联状态为1的订单
    WHERE u.is_deleted = 0;  -- 左表条件：过滤未删除用户
    
    -- 错误（右表条件放 WHERE，会导致 LEFT JOIN 退化为 INNER JOIN）
    SELECT u.id, o.order_no
    FROM user u
    LEFT JOIN `order` o ON u.id = o.user_id
    WHERE u.is_deleted = 0 AND o.status = 1;
    ```
- 避免**超过3张表**的JOIN操作（多表JOIN会增加查询复杂度和性能开销，可拆分为**子查询**或**分步查询**）
4. GROUP BY与ORDER BY
- GROUP BY必须包含所有非聚合字段（与 SELECT 子句中的非聚合字段一致），避免隐式分组导致的结果不可控
    ```sql
    -- 正确
    SELECT user_id, status, COUNT(*) 
    FROM `order` 
    GROUP BY user_id, status;
    
    -- 错误（SELECT 包含非聚合字段 order_no，未在 GROUP BY 中）
    SELECT user_id, order_no, COUNT(*) 
    FROM `order` 
    GROUP BY user_id; 
    ```
- ORDER BY排序字段需确保有索引支持（如`ORDER BY create_time`需有`idx_create_time`索引），避免大表排序导致的性能问题
- 禁止在ORDER BY中使用表达式或函数（如`ORDER BY YEAR(create_time)`），会导致索引失效

5. 子查询与分页（优先使用Mabatis-plus分页插件）
- 子查询结果集较大时，建议用LIMIT 限制行数，或转为 JOIN 操作（子查询可能导致临时表创建）
- 分页查询必须使用LIMIT，且优先通过索引字段分页（如`WHERE id > 100 LIMIT 20`优于`LIMIT 100, 20`，避免全表扫描）：
    ```sql
    -- 推荐（基于索引字段分页，高效）
    SELECT id, username FROM user
    WHERE is_deleted = 0 AND id > 100 
    LIMIT 20;
    
    -- 不推荐（大偏移量会扫描大量无用数据）
    SELECT id, username FROM user
    WHERE is_deleted = 0 
    LIMIT 100, 20;
    ```

### 写入与更新语句（INSERT/UPDATE/DELETE）规范
1. INSERT语句
- 必须指定插入的字段名（避免表结构变更导致的字段顺序问题）：
    ```sql
    -- 正确
    INSERT INTO user (username, phone, create_time)
    VALUES ('test', '13800138000', NOW());
    
    -- 错误（未指定字段，依赖表结构顺序）
    INSERT INTO user VALUES (null, 'test', '13800138000', NOW());
    ```
- 批量插入时，优先使用多值插入（减少SQL执行次数）：
    ```sql
    -- 推荐（一次插入多条）
    INSERT INTO user (username, phone)
    VALUES
        ('user1', '13800138001'),
        ('user2', '13800138002');
    
    -- 不推荐（多次单条插入）
    INSERT INTO user (username, phone) VALUES ('user1', '13800138001');
    INSERT INTO user (username, phone) VALUES ('user2', '13800138002');
    ```
- 大表批量插入（>1000 条）时，需分批次插入（每批 500-1000 条），避免占用过多连接资源
2. UPDATE与DELETE语句
- 必须包含 WHERE 条件（除非明确要操作全表），禁止无条件更新/删除（避免误操作导致全表数据变更）：
    ```sql
    -- 正确（有明确条件）
    UPDATE user SET status = 0 WHERE id = 100;
    DELETE FROM `order` WHERE is_deleted = 1 AND create_time < '2024-01-01';
    
    -- 错误（无条件，风险极高）
    UPDATE user SET status = 0;
    DELETE FROM `order`;
    ```
- 逻辑删除优先于物理删除：通过UPDATE设置is_deleted = 1替代DELETE，保留数据用于追溯：
    ```sql
    -- 推荐（逻辑删除）
    UPDATE user SET is_deleted = 1, delete_time = NOW() WHERE id = 100;
    
    -- 不推荐（物理删除，数据无法恢复）
    DELETE FROM user WHERE id = 100;
    ```
- 大表**更新**/**删除**时，需加限制条件分批次操作（如每次处理 1000 条），避免长时间锁表：
    ```sql
    -- 分批次更新（每次 1000 条）
    UPDATE user
    SET status = 0 
    WHERE is_deleted = 0 AND create_time < '2024-01-01'
    LIMIT 1000;
    ```

### 性能与安全规范
1. 性能优化
- 避免在 WHERE 条件中对索引字段做**函数**/**表达式**操作（导致索引失效）：
    ```sql
    -- 错误（函数操作导致索引失效）
    SELECT * FROM user WHERE SUBSTR(phone, 1, 3) = '138';
    SELECT * FROM `order` WHERE DATE(create_time) = '2025-11-19';
    
    -- 正确（改写为索引友好的条件）
    SELECT * FROM user WHERE phone LIKE '138%';
    SELECT * FROM `order` WHERE create_time >= '2025-11-19' AND create_time < '2025-11-20';
    ```
- 避免使用SELECT DISTINCT（可能导致临时表和排序），优先通过GROUP BY去重或优化查询逻辑
- 禁止在事务中执行耗时SQL（如大表全表扫描），避免长时间占用锁资源
2. 安全防护
- 禁止拼接SQL字符串（存在SQL注入风险），必须使用**参数化查询**（如`SELECT * FROM user WHERE username = ?`）
- 限制`SELECT ... FOR UPDATE`的使用范围（会加行锁或表锁），避免锁冲突和死锁
- 避免使**HINT**强制索引（如 FORCE INDEX），除非明确优化器选择了错误索引（优化器通常更智能）

### 注释规范
- 复杂SQL需添加注释，说明业务目的、逻辑要点（单行注释要用`--`，多行注释用`/**/`）：
    ```sql
    -- 统计近30天各用户的有效订单数（排除测试订单和已删除订单）
    SELECT
        user_id,
        COUNT(*) AS valid_order_count
    FROM
        `order`
    WHERE
        is_deleted = 0
        AND is_test = 0  -- 排除测试订单
        AND create_time >= DATE_SUB(NOW(), INTERVAL 30 DAY)
    GROUP BY
        user_id;
    ```
- 临时调试SQL需标注：`TODO：调试用，上线前删除，避免误提交到生产环境`

### 检查清单
1. SQL关键字是否大写？格式是否符合换行与缩进规范？
2. 是否避免了`SELECT *`，明确指定了所需字段？
3. 逻辑删除表的**查询**/**更新**是否包含is_deleted条件？
4. UPDATE/DELETE是否包含WHERE条件？是否优先逻辑删除？
5. 多表JOIN是否明确类型？是否**超过3张表**？
6. 是否存在索引失效场景（如函数操作、隐式类型转换）？
7. 批量操作是否分批次执行？是否避免了大事务？

---

## 数据库表设计规范
### 基础设计原则
1. **业务向导原则**：表结构需精准映射业务实体，遵循“一事一表”，避免“万能表”（如同时存储用户、订单、商品信息的混合表）
2. **原子性原则**：字段需具备原子性（不可再分），例如禁止用user_info字段存储“姓名+电话”（应差分为username和phone）
3. **扩展性原则**：预留合理扩展字段（如ext_json存储非核心动态属性），避免频繁`ALTER TABLE`
4. **一致性原则**：相同含义的字段在全库保持命名、类型、长度一致（如user_id统一为BIGINT，order_no统一为VARCHAR(64)）

### 必选字段规范（强制要求）
所有表必须包含以下6个字段，按顺序排列，用于审计追踪和数据管理：
| 字段名 | 类型 | 长度 | 不是 Null | 默认值 | 注释 | 字符集 | 排序规则 | 其他约束 |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| id | BIGINT | 20 | true | - | 主键 (唯一标识) | - | - | 必须为主键，默认必须为自动递增 (AUTO_INCREMENT)，除非其他业务情况无法自增另行想办法 |
| create_user | VARCHAR | 32 | true | - | 创建人 (操作人 ID) | utf8mb4 | utf8mb4_bin | 存储创建者账号或 ID |
| create_time | BIGINT | 20 | true | | 创建时间 (时间戳) | - | - | 记录数据插入时的秒级时间戳，默认为当前时间戳 |
| update_user | VARCHAR | 32 | true | - | 更新人 (操作人 ID) | utf8mb4 | utf8mb4_bin | 存储最后更新者账号或 ID |
| update_time | BIGINT | 20 | true | | 更新时间 (时间戳) | - | - | 自动更新为最后修改时的秒级时间戳 |
| dept_id | VARCHAR | 255 | true | - | 创建人所属部门 ID | utf8mb4 | utf8mb4_bin | 关联部门表主键 |
| is_delete | TINYINT | 4 | true | 0 | 逻辑删除标识 | - | - | 0 = 未删除，1 = 已删除 |

必选字段配置说明：
- id：强制作为主键，类型为BIGINT，非空且自动递增，禁止使用复合主键
- create_time/update_time：统一使用BIGINT类型存储**秒级时间戳**（公司特殊规范）
- 字段顺序强制要求：dept_id必须位于is_delete之前（按上表顺序排列）
- is_delete：默认值为 0（未删除），删除操作时更新为 1（逻辑删除），禁止物理删除数据

### 字段设计规范
1. 命名规范
- 表名：使用小写字母+下划线，前缀体现业务模块（如user_、order_），例如user_info（用户信息表）、order_detail（订单详情表）
- 字段名：使用小写字母+下划线，见名知意（如user_name、order_amount），禁止使用拼音（如yonghuming）或缩写（如ord_amt）
- 禁止使用SQL关键字（如order、user、status），若必须使用需用反引号`` ` ``包裹（如`status`）
- 状态字段必须用code结尾，比如 status_code, grade_code等
2. 类型选择规范
- **主键**：统一为id，类型BIGINT，非空、自动递增（AUTO——INCREMENT），禁止使用INT（可能溢出）、VARCHAR（性能差）
- **状态字段**：强制使用VARCHAR类型（公司特殊规范），长度根据状态值数量设定（建议2-10），例如order_status
- **时间类型**：除必选字段create_time/update_time外，其他时间相关字段（如pay_time、login_time）也必须使用BIGINT存储秒级时间戳，禁止使用DATETIME/DATE
- **字符串类型**：
    - 短字符串（如手机号、邮箱）用VARCHAR，长度精确匹配最大需求（如手机号VARCHAR(20)，邮箱VARCHAR(128)）
    - 长文本（如备注、描述）用TEXT或LONGTEXT，避免VARCHAR(255)存储超长篇内容
    - 字符集统一使用utf8mb4（支持 emoji 和特殊字符），排序规则用utf8mb4_bin（区分大小写，适合精确匹配）
- **数值类型**：
    - 整数：优先用INT32位）或BIGINT（64位），避免TINYINT、SMALLINT（扩展性差）；无负数场景加UNSIGNED（如BIGINT UNSIGNED）
    - 小数：金额、数量等用DECIMAL（精确计算），禁止用FLOAT/DOUBLE（存在精度丢失），例如order_amount DECIMAL(10,2)（最大 99999999.99）
3. 约束规范
- **非空约束**：核心业务字段（如user_id、order_no）必须设为`NOT NULL`，避免NULL值导致的查询异常（如NULL != NULL）
- **默认值**：
    - 数值型：默认 0（如sort默认 0，quantity默认 0）
    - 字符串：非空字符串默认空串`''`（避免NULL），如`remark VARCHAR(255) DEFAULT ''`
    - 时间戳字段：默认值统一为`UNIX_TIMESTAMP()`（当前秒级时间戳）
- **唯一约束**：业务唯一字段需加唯一索引（如order_no、phone），逻辑删除表需包含is_delete（见[`索引规范`](#mysql索引设计规范)）

### 索引设计规范（关联补充）
1. **主键索引**：id字段自动创建主键索引，无需额外定义
2. **唯一索引**：
- 单一唯一字段：`UNIQUE INDEX uniq_字段名 (字段名)`（如`uniq_order_no (order_no)`）
- 逻辑删除表的唯一索引：需包含is_delete，如`UNIQUE INDEX uniq_phone_delete (phone, is_delete)`（避免删除记录阻塞新数据插入）
3. **普通索引**：查询频繁的字段（如user_id、create_time）创建普通索引，命名格式idx_字段名
4. **联合索引**：多字段查询场景创建联合索引，遵循最左前缀原则，命名格式idx_字段1_字段2（如idx_user_id_create_time）

### 表结构示范（符合规范）
```sql
CREATE TABLE `user_info` (
  `id` BIGINT NOT NULL AUTO_INCREMENT COMMENT '主键',
  `user_name` VARCHAR(64) NOT NULL DEFAULT '' COMMENT '用户名',
  `phone` VARCHAR(20) NOT NULL DEFAULT '' COMMENT '手机号',
  `user_status_code` VARCHAR(2) NOT NULL DEFAULT '0' COMMENT '用户状态：0-正常，1-冻结',
  `age` INT UNSIGNED NOT NULL DEFAULT 0 COMMENT '年龄',
  `balance` DECIMAL(10,2) NOT NULL DEFAULT 0.00 COMMENT '账户余额',
  `last_login_time` BIGINT NOT NULL DEFAULT 0 COMMENT '最后登录时间（时间戳）',
  `create_user` VARCHAR(32) NOT NULL COMMENT '创建人',
  `create_time` BIGINT NOT NULL DEFAULT UNIX_TIMESTAMP() COMMENT '创建时间（时间戳）',
  `update_user` VARCHAR(32) NOT NULL COMMENT '更新人',
  `update_time` BIGINT NOT NULL DEFAULT UNIX_TIMESTAMP() ON UPDATE UNIX_TIMESTAMP() COMMENT '更新时间（时间戳）',
  `dept_id` VARCHAR(255) NOT NULL COMMENT '创建人部门ID',
  `is_delete` INT NOT NULL DEFAULT 0 COMMENT '逻辑删除标识：0-未删，1-已删',
  PRIMARY KEY (`id`),
  UNIQUE INDEX `uniq_phone_delete` (`phone`, `is_delete`) COMMENT '手机号唯一（含逻辑删除）',
  INDEX `idx_user_status_code` (`user_status_code`) COMMENT '用户状态索引',
  INDEX `idx_create_time` (`create_time`) COMMENT '创建时间索引',
  INDEX `idx_last_login_time` (`last_login_time`) COMMENT '最后登录时间索引'
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_bin COMMENT='用户信息表';
```
### 设计检查清单
1. 表是否包含id、create_user、create_time、update_user、update_time、dept_id、is_delete7 个必选字段？
2. id是否为BIGINT类型且为主键、自动递增？
3. create_time和update_time是否为BIGINT类型，且update_time配置自动更新？
4. 状态字段是否为VARCHAR类型？
5. 字段命名是否符合“小写+下划线”规则，无拼音或关键字？
6. 核心字段是否设置NOT NULL约束，默认值是否合理？
7. 字符串字段是否使用utf8mb4字符集，排序规则是否为utf8mb4_bin？
8. 逻辑删除表的唯一索引是否包含is_delete字段？
9. 表名是否包含业务模块前缀，是否符合“一事一表”原则？