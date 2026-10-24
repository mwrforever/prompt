# java-springboot · 技术栈宪法条目

> **适用范围**：Java ≥21（条目按 JDK 25 LTS + Spring Boot 4.1 线沉淀，Boot 3.x 未覆盖即视为未命中）· Spring Boot ≥4.1 · MyBatis-Plus ≥3.5.17（spring-boot4-starter）· PostgreSQL ≥15 · Maven ≥3.9（经 mvnw wrapper）· Redisson ≥4.x
> **入库记录**：2026-10-24 维护者收录（内置路径），来源 img-create 项目 backend-java 宪法（含 2026-10-18～20 多轮用户钉版条款）
> **槽位覆盖**：A.1 / A.2 / A.3 / A.4（A.4.1–A.4.3）/ A.5 / A.7 / C.4；A.6 引用全局注释 / 日志 / 测试规范不入库；A.8 为前端体系内置不适用。CI 构建配置段见 [ci/java.md](../ci/java.md)。

## A.1 编码约束

- 依赖注入一律构造器注入且注解优先：构造器经 Lombok `@RequiredArgsConstructor` 生成（注入字段声明 `final`），仅注解无法表达时（需构造逻辑校验 / 多构造器）才手写构造器并以注释陈述理由；单构造器免写 `@Autowired`；字段注入禁用于新代码；构造器参数过多视为职责过重的重构信号
- 自定义配置树（`app.*`）一律 `@ConfigurationProperties` 类绑定（构造器绑定可用 record），前缀 kebab-case；禁散装 `@Value`；业务类注入配置类整体使用，禁把配置值拆散复制为业务类私有字段，字段类型转换在使用点就地完成
- 外部系统端点 URL 等环境相关配置一律配置文件承载、列表式统一管理（按渠道 / 系统分组），禁抽离到类 static 字段硬编码
- record 用于不可变数据载体（DTO / VO / 值对象），紧凑构造器做入参校验；需可变 / 继承的场景禁用 record
- switch 一律箭头标签 `case A, B ->`，禁冒号标签 fall-through；switch 表达式必须穷尽
- 业务代码禁全类名引用：类型一律 import 后以简单名书写；仅同名类冲突且无 import 别名时允许全类名（语言层唯一例外）
- 业务对象转换一律 MapStruct：层间转换统一由 convert 包转换器承担（编译期生成），禁手写逐字段 set 复制、禁 BeanUtils 等运行时反射拷贝
- Web 并发基线启用虚拟线程；CPU 密集型任务禁占用请求虚拟线程，须显式专用线程池
- 全部代码挂靠单一根包，禁平级新增语义重叠包
- 代码质量通则（2026-10-20 用户钉版）：可读性——循环条件内禁调用方法（提取局部变量）、命名达意、复杂条件拆名；算法——同功能择优时间 / 空间复杂度（不为微优化牺牲可读性）；扩展性——按明确具体场景设计接入点，杜绝无场景的过度设计
- Lombok 使用准则（2026-10-20 用户钉版）：**@Builder** 字段 ≥4 的对象组装启用，与 `@NoArgsConstructor` 共存必须显式加 `@AllArgsConstructor`（否则隐式全参构造器吞掉无参构造致反射实例化失败），继承体系用 `@SuperBuilder` 禁 `@Builder`；**@RequiredArgsConstructor** 依赖注入构造器唯一形态；**@Slf4j** 日志声明唯一形态；**@Data** 实体禁用（DTO / VO 可用，含敏感字段须 `@ToString(exclude=...)`）；**@SneakyThrows** 禁用；清单未列注解默认不使用，确需引入先记 CHANGELOG

## A.2 配置管理

- `application.yml` 共享基座，环境差异一律放 `application-{profile}.yml`（profile 专属覆盖非专属）
- 敏感值统一 `${VAR:default}` 环境变量占位注入；dev 兜底默认值必须无害，生产值必须外部注入强制覆盖；密钥仅经环境 / secrets 注入，禁落代码与配置文件明文
- `.env` 非 Spring 原生机制：仅可 `spring.config.import=optional:file:.env[.properties]`，且 `.env` 禁入 git（模板入库）

## A.3 API 设计

- 服务前缀域显式钉死（如用户链 / 管理链双前缀域），禁新增未声明前缀域
- 全部 REST 接口返回统一包装 `{code, message, data}`（code 为字符串：`"0"` 成功、业务错误分段码）；新增接口先补响应样例文档再实现
- 请求体一律绑定 DTO 并施加 jakarta.validation 约束注解，Controller 形参标注 `@Valid`；入参校验字段级承载，禁业务方法内散装手写校验
- 全局异常 `@RestControllerAdvice` + `@ExceptionHandler` 集中；业务异常必须携带错误码，与统一包装 code 对齐
- Controller 薄层：仅参数绑定、校验、调 service、组装 VO 返回；禁业务逻辑 / 事务 / SQL
- springdoc 为 API 契约唯一事实源：接口 / 模型变更必须同步再生成 spec 提交物供前端生成类型消费；spec 中 id 一律 `type: string`（雪花 ID 字符串出端）

## A.4 数据库操作（PostgreSQL）

### A.4.1 迁移与 Schema

- Schema 唯一来源 = Flyway：全部 DDL（含索引）只经版本化迁移；禁 schema.sql / data.sql、禁自动 DDL、禁手工改库
- 迁移单目录显式声明（禁通配符）；命名 `V{n}__{snake_case}.sql`；跨环境数据修复用 `R__` 前缀
- 迁移版本按业务域分段号段管理（各域百位段续接），号段调整先记 CHANGELOG；号段归属与版本唯一性并入 CI 校验
- 禁止修改已应用迁移（checksum 校验必失败），错误修正只能追加新版本迁移（前向修复制）；outOfOrder=false、baseline-on-migrate=false
- 业务唯一性约束必须落库层强制（含部分唯一索引 `UNIQUE(...) WHERE ...`），禁仅应用层校验
- 【全局铁律】表关系一律逻辑关联，禁止物理关联：禁 FOREIGN KEY 物理外键，关联经 id + 域判别列逻辑表达，完整性由唯一约束与应用层保障（2026-10-19 用户钉版）
- jsonb 字段经统一 JSON TypeHandler 映射为强类型模型，禁裸 String / Map 直读直写

### A.4.2 连接、事务与数据规范

- 多服务共享库时按业务功能域划分写域：本服务只写属主域数据，他属主域仅限契约明确的只读访问；新表落地先明确属主侧并同步契约，禁跨域写
- 连接池 HikariCP 固定大小（minimumIdle=maximumPoolSize），压测校准且多实例总连接数低于库 max_connections 余量；leakDetectionThreshold 分环境（禁 0）；JDBC URL 开启 reWriteBatchedInserts=true；禁自建 DataSource bean
- 事务边界：`@Transactional` 放 service 实现层方法级最小边界，禁标注接口（CGLIB 代理下静默失效）、禁 controller 开事务、禁业务代码手动 commit / rollback；默认只回滚 RuntimeException / Error（按需显式 rollbackFor）；查询型方法标注只读事务；警惕同类自调用事务不生效
- 事务内禁止远程调用、消息发送与人工等待；对外交互在事务提交后执行（确需原子性时 outbox 模式先行评审）
- 金额 / 计费类数值一律整数（分）存储与运算，禁 float / double；分↔元换算收敛统一工具层，禁业务代码散落 ×100 / ÷100
- 审计字段 created_at / updated_at 由数据库维护（DEFAULT now() + updated_at 触发器）
- 敏感字段应用层列加密（AES-GCM）后落库，密钥经环境注入；需等值检索的敏感字段配 HMAC 盲索引辅助列

### A.4.3 MyBatis-Plus 使用规范

- 依赖定稿：mybatis-plus-spring-boot4-starter + mybatis-plus-jsqlparser（父 POM 显式锁版本）；Boot 4 适配须手动补新版 mybatis-spring（官方 #7009 内嵌版本过低）；除此之外禁再引入裸 mybatis / mybatis-spring-boot-starter（重复引入冲突）
- Service 形态（层级依赖钉版，2026-10-20）：接口 `I{业务域}Service`、实现 `{业务域}ServiceImpl` 落 service/impl/；有主表实体的业务域必须实现 MP 层级依赖（extends IService / ServiceImpl），无实体的业务域用普通接口 + 实现；两级分层铁律——单表简单 SQL 一律 MyBatis-Plus 链式，复杂查询（连表 / 分组 / 聚合 / 复杂条件）一律 mapper 接口 + XML；infrastructure/ 仅存放无业务语义的基础设施能力，禁放业务域逻辑
- 单表链式：主表操作一律 `this.lambdaQuery()` / `this.lambdaUpdate()`；查非本 service 主表才用 `Wrappers` 静态工厂；wrapper 禁跨层 / 跨线程传递
- 按需取列：查询必须 `select()` 精确投影，禁 SELECT *；循环内单查改 in 批量，拒绝 N+1
- 复杂 SQL 走 mapper 接口 + XML 映射文件；禁业务层拼 SQL、禁 JdbcTemplate 字符串拼接
- ID 生成：`@TableId(ASSIGN_ID)` + 自定义 IdentifierGenerator（机器号分段 + 注册校验防冲突形态），禁散装手动生成；批量插入 saveBatch 须在事务内
- 分页必须经 `PaginationInnerInterceptor`（maxLimit 显式）；分页 / 取前 N 必带唯一顺序 ORDER BY；大 OFFSET 深翻页改 Keyset；前端可传排序列时必须列名白名单校验（防 SQL 注入）
- 逻辑删除统一 `@TableLogic` + 全局配置；并发敏感实体 `@Version` 乐观锁；BlockAttackInnerInterceptor 拦截无 WHERE 全表 UPDATE / DELETE；插件集中注册于单一 Interceptor Bean，禁散落配置
- entity 与 mapper 接口不出 service 边界；跨 service 复用查询经注入对方 service 公开方法，禁直接操作他人 mapper

## A.5 基础设施生命周期

- Redis 访问统一收敛基础设施封装：Stream 操作 / 分布式锁 / 注册校验用 Redisson，简单缓存用 Spring Data Redis；业务代码禁散操裸客户端
- Redis Lua 脚本一律 resources/lua/ 文件承载（一功能一文件、KEYS/ARGV 显式传参），禁 Java 代码内嵌字符串 Lua；调用统一预注册单例工具包 evalsha（NOSCRIPT 自动重载）（2026-10-20 用户钉版）
- 缓存分轨：简单缓存一律 Spring Cache 注解；复杂业务缓存（Lua 原子操作、多键协同）按领域抽取单一具体类 `{领域}Cache`（不做接口 / 实现分离、不套 IService 层级），禁业务 service 内散写 Redis 命令；领域缓存允许只读回源，禁经其写库
- 与他端共享的 Redis 键 / Stream 命名以契约文档为唯一契约，变更双侧评审、先记 CHANGELOG
- 多前缀域多 SecurityFilterChain 时，各链匹配边界与 order 必须显式声明，禁隐式叠加；JWT 一律非对称签发（RS256 / ES256），私钥仅签发方持有、公钥分发验签，密钥经环境注入支持轮换
- 生产启用 `server.shutdown=graceful` + shutdown-phase 超时；Actuator + micrometer 指标为运行时观测基线，暴露端点范围与 security 联动收敛

## A.7 跨层数据对象与传参约束（本栈习语）

1. **参数对象化**：Controller 形参 > 3 或含可选参数必须收拢为请求 DTO（record 优先 + 约束注解 + `@Valid`）整体传参；mapper 多参数查询必须参数对象 / Query 条件对象，禁 `Map` 散传。仅业务确需灵活传参可例外。
2. **返回业务对象**：对外返回一律独立 VO（字段最小暴露）置于统一包装 data 内，禁 Controller 直接返回 Entity（表结构泄露）；请求 DTO（dto/）与响应 VO（vo/）分目录存放，禁同一对象双向复用；id 类字段 VO 层一律 String（雪花 ID 字符串出端铁律），Entity 层保持整型主键。
3. **对象职责四分（2026-10-20 用户钉版）**：①枚举一律 enums/ 包（enum 类型 + code 双向映射）②DTO = 请求入参 + 业务流转核心实体（类形态 Lombok @Data）③VO = 返回前端的业务对象（类形态 Lombok，禁用 record 承载 DTO/VO 职责）④record 仅限「需要不可变能力且不需要多对象之间转换」的业务对象（多值返回、临时聚合、查询投影，落 record/ 或组件包内私有载体）——不得因省事混用；层间转换一律经 convert 层 MapStruct。
4. **实体注解组合（2026-10-20 用户钉版）**：实体禁 `@Data`——一律 `@TableName + @Getter @Setter @NoArgsConstructor @AllArgsConstructor + @EqualsAndHashCode(of = 业务自然键)（排除自动生成 id）+ @ToString(exclude = 敏感字段)`；入参校验用 jakarta.validation 注解承载，禁业务方法内散装手写校验。
5. **例外边界**：例外须能陈述业务理由；无理由的违反视为缺陷，不得合入。

## C.4 常用命令骨架

```bash
./mvnw clean verify        # 全量门禁（格式化校验 + 全部测试 + 打包，合入前必过）
./mvnw spring-boot:run     # 本地启动
./mvnw test -Dtest=Class   # 单类测试
./mvnw spotless:apply      # 格式化修复
./mvnw spotless:check      # 格式化校验（CI 快层消费，可增量限定文件）
./mvnw clean package       # 打包可执行 jar
```

> 注：mvn/mvnw 命令前须 `export JAVA_HOME=<钉版 JDK 路径>`（系统默认版本低于项目要求为常见陷阱）。

## 来源

- img-create 项目 `backend-java/AGENTS.md`（Part A / C.4 / C.6，含用户钉版条款）及其 C.2 版本表实测记录
- 关键官方文档：Spring Boot 4.1（模块化 starter：restclient / flyway / webmvc-test 迁移）、MyBatis-Plus #7009、springdoc 3（3.1.x 才兼容 Boot 4.1 线）、Flyway 12
