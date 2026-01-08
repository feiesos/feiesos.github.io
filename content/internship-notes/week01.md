+++
date = '2026-01-08T11:19:29+08:00'
draft = false
title = 'Week01'
toc = true
tocBorder = true
+++
# 规范
## Java编码规范
### 编码风格
1. 代码风格
    - 使用4个空格代替Tab缩进
    - 控制单行代码长度不超过120字符，否则遵循一下原则换行：
        - 在逗号后换行
        - 在操作符前换行
        - 换行后缩进四个空格（方法调用链多缩进一层以区分）
    - 方法、类、代码块之间用1个空行隔开，连续空行不超过2行
    - 遵循K&R风格（左大括号不单独成行）
2. 注释规范
    - 类、接口、枚举的注释必须包含类级Javadoc，说明功能、作者、创建日期
        > 比如@author xxx @data 2025-11-18
    - 方法必须包含Javadoc，说明入参、返回值、异常
        > 比如@param name 用户名 @return 是否存在 @throws NullPointerException 当name为null时
    - 复杂逻辑（如分支条件、算法步骤）需加单行注释，避免冗余注释

### 命名规范
1. 基本原则
    - 命名需见名知意，不使用拼音，可使用国际通用缩写（如UserDTO）
    - 不使用单字母命名（局部循环变量除外），不使用`_`、`$`等特殊字符
2. 具体规则
    - 类/接口/枚举：使用 UpperCamelCase（大驼峰）
    - 方法/参数/局部变量：使用 lowerCamelCase（小驼峰）
    - 常量：使用全大写+下划线分隔，如MAX_RETRY_COUNT
        > 注：常量指public static final修饰的字段
    - 包名：使用全小写，多级包用`.`分隔，不包含下划线，如com.company.user.service
    - 抽象类：
        - 以Abstract开头，如AbstractUserDAO
        - 接口不加前缀，如UserService而非IUserService
    - 异常类：以Exception结尾，如UserNotFoundException
    - 集合类：使用复数形式或明确包含 "List/Map/Set"，如userList、idToUserMap
    - 布尔变量：避免使用is前缀（POJO 类的布尔字段除外），推荐用hasXXX、canXXX、isXXX（如hasPermission、canEdit）

### 常量与变量
1. 常量定义
    - 常量必须通过`public static final`修饰，集中定义在类的顶部（枚举类除外）
    - 不在接口中定义常量（接口仅用于定义方法契约）
    - 避免**魔法值**（未定义的常量），如`if (type == 1)`应改为`if (type == ORDER_TYPE_NORMAL)`
    - 跨类共享的常量需放在专门的常量类中（如CommonConstants），同类内部使用的常量直接定义在当前类
2. 变量使用
    - 成员变量需明确访问修饰符（private优先，避免public），必要时通过get/set方法访问
    - 局部变量作用域最小化，如仅在循环体内使用的变量不定义到循环体外
    - 基本类型与包装类型选择：
        - 局部变量、方法参数优先使用**基本类型**（int而非Integer）
        - 集合、POJO属性使用**包装类型**（允许null表示未赋值）
    - 不使用null初始化变量，未赋值的引用类型变量默认值为null

### 集合处理
1. 初始化
    - 集合初始化时指定初始容量[如new ArrayList<>(10)]，避免频繁扩容（初始容量 = 预计元素数 / 负载因子 + 1，HashMap 负载因子默认 0.75）
    - 不使用new ArrayList(10)[原始类型，需指定泛型new ArrayList<String>(10)]
2. 遍历与操作
    - 遍历集合优先使用增强 for 循环[for (String s : list)]，需要索引时使用普通for循环
    - 不在**foreach**循环中修改集合（会抛出ConcurrentModificationException），如需修改使用迭代器（Iterator）的remove方法
    - 集合转数组使用toArray(T[])，如list.toArray(new String[0])，避免返回Object[]强转导致ClassCastException
3. 工具类
    - 创建空集合使用`Collections.emptyList()`/`emptyMap()`（不可变），或`new ArrayList<>()`（可变），不返回null
    - 集合判空用`CollectionUtils.isEmpty(collection)`（需引入`commons-collections`工具类），避免`collection == null || collection.size() == 0`
    - 集合拷贝使用`new ArrayList<>(sourceList)`而非手动循环添加

### 控制语句
1. 分支语句
    1. if/else
        - 条件表达式必须用括号包裹（如if (a > b)，不if a > b）
        - 多分支条件优先使用if-else if而非多个独立if（避免重复判断）
        - 复杂条件提取为布尔变量，如：
            ```java
            boolean isOverdue = endTime < System.currentTimeMillis();
            if (isOverdue) {
                ...
            }
            ```
        - 不在if后省略大括号（即使单行代码），如：
            ```java
            // 错误
            if (flag) doSomething();
            // 正确
            if (flag) {
                doSomething();
            }
            ```
    2. switch/case
        - switch参数支持byte/short/int/char/String/enum，不使用 long/float/double
        - 每个case块必须以break/return结束，或注释说明穿透逻辑（如// fallthrough）
        - 必须包含default分支（即使为空，需注释说明）

2. 循环语句
    - 避免死循环，循环条件需有明确退出机制
    - 不在循环内定义新对象，如：
        ```java
        for (...) {
            String s = new String();
        }
        ```
        应改为循环外定义
    - 多层循环嵌套不超过 3 层，超过时提取为方法

### 异常处理
1. 异常抛出
    - 方法抛出的异常必须在**Javadoc**中声明（@throws）
    - 不抛出Exception/Throwable等**宽泛异常**，应抛具体异常（如IOException而非Exception）
    - 不手动抛出NullPointerException，通过代码逻辑避免空指针（如提前判空）
    - 业务异常需自定义（继承RuntimeException），如UserNotExistException
2. 异常捕获
    - try-catch块范围最小化，仅包裹可能抛出异常的代码
    - 不捕获异常后不处理（空catch块），至少打印日志（log.error("错误信息", e)）
    - 捕获异常时需先捕获子类异常，再捕获父类异常（如先catch (IOException)再catch (Exception)）
    - 不在finally块中使用return（会覆盖try/catch中的返回值）

### 面向对象规范
1. 类设计
    - 类名体现职责，一个类只做一件事（单一职责原则）
    - POJO 类（实体类）属性必须私有，提供get/set方法，重写toString()（便于日志打印）
    - 工具类（如StringUtil）应私有构造方法（不实例化），所有方法静态
    - 不在类中定义public static可变字段[如`public static List<String> list = new ArrayList<>()`]
2. 继承与实现
    - 继承遵循"is-a"关系，重写方法时加@Override注解
    - 接口中只定义方法签名，不包含实现（默认方法default谨慎使用）
    - 抽象类中可包含抽象方法和具体方法，作为模板骨架
3. equals与hashCode
    - 重写equals()必须同时重写hashCode()（否则HashMap等集合可能无法正常工作）
    - equals()判断需避免空指针，推荐`Objects.equals(a, b)`而非a.equals(b)
    - 基本类型用==比较，引用类型用equals()（除String常量池比较外）

### 并发处理
1. 线程安全
    - 多线程环境下共享变量需加锁（synchronized或Lock），或使用线程安全类（如ConcurrentHashMap）
    - `synchronized`优先修饰方法或代码块（范围最小化），避免修饰整个类
    - 不在多线程中使用`SimpleDateFormat`（线程不安全），改用`DateTimeFormatter`（Java 8+）
2. 线程池
    - 不使用Executors创建线程池（默认参数可能导致 OOM），需手动创建`ThreadPoolExecutor`并指定核心参数
    - 线程池命名需包含业务标识（如orderProcessThreadPool），便于问题排查
    - 线程池使用后需调用`shutdown()`关闭，避免资源泄漏

### 其他规范
1. 日志
    - 日志级别使用：
        - error（错误，影响功能）
        - warn（警告，不影响功能）
        - info（重要业务节点）
        - debug（调试信息，生产环境关闭）
    - 日志打印需包含上下文信息（如用户ID、订单号），便于定位问题
    - 不使用System.out.println()，统一使用日志框架（如Logback、Log4j2）
2. 工具依赖
    - 字符串处理优先使用`StringUtils`（org.apache.commons.lang3），避免null判断
    - 日期处理使用Java 8+的`LocalDateTime`/`Instant`，替代`Date`/`Calendar`
    - 集合操作使用`CollectionUtils`/`MapUtils`（commons-collections）简化判空和操作
3. 代码提交
    - 提交代码前需格式化（IDE自动格式化），通过静态检查工具（如SonarQube）扫描
    - 不提交调试代码（如System.out、TODO、注释掉的代码）
    - 代码注释需与逻辑同步更新，避免注释与代码不一致

---

## Git分支策略规范
### 核心分支定义（长期存在）
1. master分支
    - **用途**：存放正式发布的代码，始终保持可部署状态（生产环境代码来源）
    - **权限**：不直接提交，仅通过release或hotfix分支合并
    - **保护机制**：配置分支保护（如GitLab/GitHub中设置 “不强制推送”“需要代码审核才能合并”）
2. develop分支
    - **用途**：开发主分支，集成已完成的功能，作为下一个版本的开发基线（测试环境代码来源）
    - **权限**：不直接提交，仅通过feature分支合并
    - **特性**：包含最新开发成果，保持相对稳定（可随时构建为测试版本）

### 临时分支定义（完成后删除）
1. feature分支（功能分支）
    - 用途：开发新功能、优化或非紧急 Bug 修复（基于develop分支创建）
    - 命名规范：feature/[issue编号]-[功能描述]（小写，用连字符分隔）示例：`feature/123-user-login`、`feature/456-order-optimize`
    - 流程：
        - 从develop分支checkout新分支：`git checkout -b feature/xxx develop`
        - 开发完成后，提交Merge Request（MR）到develop分支
        - 需通过代码审核（至少1名团队成员Approve），通过后合并，删除该feature分支
2. release分支（发布分支）
    - 用途：准备版本发布，仅修复发布前的minor bug（不开发新功能），基于develop分支创建
    - 命名规范：release/v[主版本].[次版本].[修订号]示例：`release/v1.2.0、release/v2.0.1`
    - 流程：
        - 当develop集成的功能满足发布条件时，从develop分支创建：`git checkout -b release/vx.y.z develop`
        - 仅修复测试中发现的Bug，修复后同步到develop分支（避免遗漏）
        - 发布完成后，合并到master分支（并打Tag）和develop分支，删除该release分支
3. hotfix分支（紧急修复分支）
    - 用途：修复生产环境（master 分支）出现的紧急Bug，基于master分支创建
    - 命名规范：hotfix/[issue编号]-[bug描述] 或 hotfix/v[主版本].[次版本].[修订号]示例：`hotfix/789-login-error、hotfix/v1.1.1`
    - 流程：
        - 从master分支checkout新分支：`git checkout -b hotfix/xxx master`
        - 修复完成后，提交MR到master分支（发布修复版本）和develop分支（同步修复到开发主线）
        - 合并后，删除该hotfix分支

### 分支合并规则
1. **不直接推送**：master、develop 分支不允许直接`git push`，必须通过MR合并
2. **代码审核**：所有MR需经过至少1名团队成员审核（重点检查逻辑、风格、测试覆盖）
3. **冲突处理**：合并前需在本地解决分支冲突（`git pull --rebase`优先，避免冗余合并提交）
4. **自动化校验**：MR触发CI流程（如单元测试、代码规范检查、构建验证），全部通过才能合并

### 版本号规范（语义化版本）
遵循 主版本号.次版本号.修订号（如 v1.2.3）：
- **主版本号（X）**：不兼容的API变更（如架构重构），X.0.0
- **次版本号（Y）**：向后兼容的功能新增，1.Y.0
- **修订号（Z）**：向后兼容的问题修复（含hotfix），1.2.Z
- **Tag标记**：合并到master分支后，必须打Tag标记版本：`git tag -a v1.2.0 -m "Release v1.2.0"`，并推送Tag到远程

### 协作场景示例
1. 新功能开发：develop → feature/xxx（开发）→ MR合并到develop
2. 版本发布：develop → release/vx.y.z（测试修复）→ 合并到 master（打Tag）和develop
3. 生产Bug修复：master → hotfix/xxx（修复）→ 合并到master（打Tag）和develop

---

## 代码提交规范
### 提交信息格式
每次提交需包含**类型（type）、范围（scope，可选）、描述（subject）**，并可附加**详细说明（body，可选）**和**关闭的issue（footer，可选）**

基本格式：
```text
<type>(<scope>): <subject>

[body]

[footer]
```

示例：
```text
feat(user): 新增用户注册验证码功能

- 集成短信验证码接口
- 增加验证码有效期校验

Closes #123
```

### 核心字段说明
1. 类型（type）
必须指定，说明提交的性质，取值范围：
| 类型 | 含义 | 适用场景 |
| :--- | :--- | :--- |
| **feat** | 新功能开发 | feature 分支开发新功能 |
| **fix** | 修复 Bug | feature (普通 Bug)、hotfix (生产 Bug) |
| **docs** | 文档更新 | 接口文档、注释调整 (仅修改文档，不涉及代码) |
| **style** | 代码格式调整 | 缩进、命名规范、删除多余空格等 (不影响代码逻辑) |
| **refactor** | 代码重构 | 逻辑优化、变量重名、函数拆分 (既非新功能也非修复 Bug) |
| **perf** | 性能优化 | 减少数据库查询、优化算法复杂度 |
| **test** | 新增 / 修改测试用例 | 单元测试、集成测试 |
| **build** | 构建配置修改 | 依赖版本调整、打包脚本修改 |
| **ci** | CI 流程配置修改 | GitLab CI、Jenkins 脚本调整 |
| **chore** | 其他操作 | 清理临时文件、修改 .gitignore (不影响代码逻辑的操作) |
2. 范围（scope）
可选，说明提交影响的模块/服务，建议使用**服务名**、**模块名**或**功能点**，如：
    - 服务名：user（用户服务）、order（订单服务）
    - 模块名：dao（数据访问层）、controller（接口层）
    - 功能点：login（登录）、pay（支付）
3. 描述（subject）
    - 简要描述提交内容，不超过 50 个字符
    - 以动词开头（如 “新增”“修复”“优化”），首字母小写，结尾不加句号
    - 示例：
        > - feat(order): 支持批量取消订单
        > - fix(redis): 修复缓存过期时间计算错误
4. 详细说明（body）
可选，用于补充subject未说明的细节，可分点描述（使用`-`或者`*`开头）：
- 说明“为什么做这个修改”、“实现思路”、“与之前的区别”等
- 示例：
    > fix(login): 修复密码错误次数超限后未锁定账号的问题  
    > - 原逻辑仅记录错误次数，未触发锁定机制  
    > - 新增判断逻辑：错误次数≥5时，调用账号锁定接口
5. 关闭的issue（footer）
可选，用于关联并关闭相关的任务/缺陷编号（需与项目管理工具联动，如Jira、GitLab Issues）：
- 格式：**Closes #issue编号**或**Fixes #issue编号**
- 示例：**Closes \#456**（关闭编号456的issue）

### 禁止性规范
1. 禁止一次提交包含多个不相关的修改（如同时开发新功能 + 修复另一个Bug），需拆分多次提交
2. 禁止使用模糊描述（如“修改了一些问题”、“更新代码”），必须明确具体内容
3. 禁止提交无关文件（如IDE配置、本地日志），通过`.gitignore`过滤

### 提交记录示例
| 分支类型 | 提交信息示例 |
| :--- | :--- |
| **feature** | feat(pay): 新增支付宝支付接口 |
| **feature** | refactor(order): 拆分订单创建逻辑为独立服务 |
| **release** | fix(logger): 修复生产环境日志打印乱码问题 |
| **hotfix** | fix(user): 紧急修复用户登录时密码解密报错 |
| **develop** | docs(api): 更新用户服务接口文档 |