+++
date = '2026-01-12T18:05:48+08:00'
draft = false
title = 'Week03'
toc = true
tocBorder = true
+++
# 全局配置规范
## 配置文件命名规范
1. **基础配置文件**：以application-base-[模块名].yml命名，如application-base-db.yml（数据库基础配置）、application-base-storage.yml（存储基础配置）
2. **环境配置文件**：以application-[环境名].yml命名，支持的环境包括：
- application-dev.yml：开发环境
- application-stg.yml：测试布环境
- application-test.yml：现场测试环境
- application-beta.yml：预生产环境
- application-prod.yml：生产环境
3. **主配置文件**：application.yml用于指定激活环境和包含的基础配置

## 配置层级规范
1. **主配置文件（application.yml）**：仅配置spring相关核心配置，包括：
- 应用名称（spring.application.name）
- 激活环境（spring.profiles.active）
- 包含的基础配置（spring.profiles.include）
2. **基础配置文件**：按功能模块拆分配置，每个模块配置独立文件，包括但不限于：
- 数据库配置（base-db）
- 存储配置（base-storage）
- 系统管理配置（base-system-manager）
- 事务配置（base-trans）
- 分布式锁配置（base-dlocker）
- 序列配置（base-sequence）
- 服务保护配置（base-guard）
3. **环境配置文件**：仅配置当前环境特有的参数，覆盖基础配置中的变量，包括：
- 端口号（server.port）
- 数据库连接信息（db.sql.\*）
- 缓存配置（db.redisson.\*）
- 第三方服务地址（如 system-manager.sysServer）
- 环境特定开关（service-guard.\*.switch）

## 配置参数命名规范
1. 使用小写字母，单词之间使用短横线`-`分割（如max-http-header-size）
2. 层级结构使用缩进表示，避免使用下划线
3. 同类配置使用统一前缀：
- 数据库相关：db.sql.\*
- 缓存相关：db.redisson.\*
- 存储相关：db.storage.\*
- 服务保护：service-guard.\*
- 系统管理：system-manager.\*

## 配置内容规范
1. 环境隔离原则：不同环境的配置参数严格区分，避免在基础配置中硬编码环境特征值
2. 变量引用原则：基础配置中使用`${变量名:默认值}`形式引用环境中的变量，如`url: jdbc:mysql://${db.sql.ip-port}/${db.sql.database-name}`
3. 敏感信息处理：密码等敏感信息直接配置（当前规范，后续可考虑加密）
4. 开关配置：功能开关统一使用isRun、enabled或switch作为关键字，如systemmanager.config.isRun、service-guard.pressure.switch
5. 路径配置：URL路径统一使用`/`开头，多个路径用逗号分隔，如`exclusions: /custom/circuitbreaker,/spel`
6. 超时配置：时间单位明确，毫秒级配置建议添加ms后缀（如expire: 259200000表示3天，259200000ms）

## 配置加载顺序规范
1. 主配置文件（applicaton.yml）优先加载
2. 基础配置文件，按spring.profiles.include指定的顺序加载
3. 环境配置文件（如application-dev.yml）最后加载，覆盖基础配置中的同名参数

## 特殊模块配置规范
1. **数据库配置**：
- 连接池参数统一配置在base.datasource节点
- MyBatis-Plus配置统一放在mybatis-plus节点
- 逻辑删除字段统一使用isDelete
2. **存储配置**：
- MinIO配置统一放在dromara.x-file-storage.minio节点
- 区分内部域名（domain-inner）和外部域名（domain-outer）
3. **服务保护配置**：
- 按功能模块划分配置（限流、熔断、缓存等）
- 统一使用service-guard作为根节点
- 同意路径统一配置在exclude列表
4. **日志配置**：
- 日志相关配置放在logback和logging节点
- 区分控制台日志级别和文件日志级别

## 版本控制规范
1. 所有配置文件纳入版本控制
2. 环境配置文件中的敏感信息如需保护，可考虑使用配置中心
3. 配置变更需要通过代码审评，确保符合配置规范

## 扩展规范
1. 新增配置模块时，需创建对应的application-base-[模块名].yml文件
2. 新增环境时，需创建对应的application-[环境名].yml文件，并在主配置文件中支持切换
3. 公共配置参数优先放在基础配置文件，环境特定参数放在对应环境配置文件

## 检查清单
1. **文件命名规范检查**
- 基础配置文件均以application-base-[模块名].yml命名（如application-base-db.yml）
- 环境配置文件均以application-[环境名].yml命名（支持dev/test/dev-test/beta/prod）
- 主配置文件为application.yml且仅包含核心激活配置
2. **配置层级规范检查**
- 主配置文件仅包含spring.application.name、spring.profiles.active、spring.profiles.include
- 基础配置文件按功能模块拆分（db/storage/system-manager等）
- 环境配置文件仅包含当前环境特有参数（端口/连接地址/开关值等）
3. **参数命名规范检查**
- 所有参数使用**小写字母**+**短横线**分隔（如max-http-header-size）
- 同类配置使用统一前缀（如db.sql.\*/service-guard.\*）
- 避免使用下划线（特殊第三方依赖除外）
4. **配置内容规范检查**
- 基础配置中使用${变量名:默认值}引用环境变量（如url: jdbc:mysql://${db.sql.ip-port}/...）
- 敏感信息（密码/密钥）直接配置（按当前规范）
- 功能开关统一使用isRun/enabled/switch关键字
- URL路径以`/`开头，多路径用逗号分隔
- 超时配置明确时间单位（毫秒级建议加ms后缀）
- 不同环境配置参数严格隔离，无硬编码环境值
5. **模块配置专项检查**
- 数据库配置
    - 连接池参数在base.datasource节点配置
    - MyBatis-Plus配置在mybatis-plus节点
    - 逻辑删除字段使用isDelete
- 存储配置
    - MinIO配置在dromara.x-file-storage.minio节点
    - 区分内部域名（domain-inner）和外部域名（domain-outer）
- 服务保护配置
    - 根节点为service-guard
    - 按功能模块划分（pressure/secret 等子节点）
    - 排除路径配置在exclude列表
- 分布式锁配置
    - Redisson参数引用db.redisson.\*环境变量
    - 连接池参数统一配置（master-connection-pool-size 等）
6. **环境配置一致性检查**
    - 各环境配置文件中同类型参数结构一致（如db.sql在dev/test/prod中包含相同子节点）
    - 服务保护开关（service-guard.\*.switch）在各环境按需求正确设置
    - 第三方服务地址（如system-manager.sysServer）按环境正确配置
7. **版本控制检查**
    - 所有配置文件已纳入版本控制
    - 配置变更已通过代码评审
    - 环境配置中敏感信息如需保护已计划迁移至配置中心

---