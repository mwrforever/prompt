# python-fastapi · 技术栈宪法条目

> **适用范围**：CPython ≥3.13 · FastAPI ≥0.140（0.143.3 实测）· uvicorn ≥0.54 · pydantic-settings ≥2.16 · SQLAlchemy 2.x（async ORM，2.1.5 实测）+ asyncpg ≥0.32 · redis-py ≥8（async 客户端）· httpx（AsyncClient）· uv ≥0.13 · ruff ≥0.17 · pytest ≥9
> **入库记录**：2026-10-24 维护者收录（内置路径），来源 img-create 项目 backend-ai 宪法（含 2026-10-18～20 多轮用户钉版条款）
> **槽位覆盖**：A.1 / A.2 / A.3 / A.4 / A.5 / A.7 / C.3 目录职责骨架 / C.4；Part B（agent 编排 / LLM 网关 / 技能系统）属项目架构主题不收录；A.6 引用全局注释 / 日志 / 测试规范不入库；A.8 为前端体系内置不适用。CI 构建配置段见 [ci/python.md](../ci/python.md)。

## A.1 编码约束

- 全部函数参数 / 返回值显式类型标注，容器用内置泛型语法（`list[str]`、`str | None`、`dict[str, float]`）
- 路由 async/def 选择规则：内部有 `await`（httpx AsyncClient、redis async 等）必须 `async def`；调用同步阻塞库的路由声明普通 `def`（自动进外部线程池）
- `async def` 上下文内禁止同步阻塞调用（time.sleep、requests、同步 DB、重 CPU 计算）；确需调同步库时用 `fastapi.concurrency.run_in_threadpool` 包装
- httpx 一律 AsyncClient 且复用实例（统一基础设施封装管理），禁每次请求新建客户端
- 依赖注入统一 `Annotated[T, Depends(fn)]` 写法；`Depends()` 传函数本身不加括号
- 业务异常必须自定义异常类分类并经 `@app.exception_handler` 全局注册；业务侧 4xx 用 `raise HTTPException(...)`，禁止 return 错误响应；覆盖 `RequestValidationError` handler 时逐条提取 `exc.errors()` 的 loc/msg，禁 `str(exc)` 直返（防泄露文件路径与行号）
- 【全局铁律】设计类型显式优于隐式（2026-10-19 用户钉版）：类型、契约、配置一律显式声明，禁隐式约定与魔法默认，健壮性与可维护性优先
- 代码质量通则（2026-10-20 用户钉版）：可读性——循环条件内禁函数调用（提取局部变量）、命名达意、复杂条件拆名；算法——同功能择优时间 / 空间复杂度（不为微优化牺牲可读性）；扩展性——按明确具体场景设计接入点，杜绝无场景的过度设计

## A.2 配置管理

- 配置唯一入口为 BaseSettings 子类（pydantic-settings）；分层源与优先级固定：环境变量 > `.env` > `config.yaml` > 字段默认值（config.yaml 承载非敏感灵活项；敏感项只存占位注释，实值仅经环境注入）
- Settings 单例：`@lru_cache` 包装获取函数，禁每请求重建；测试经 `app.dependency_overrides` 覆盖
- 配置对象整体注入：业务代码注入 Settings 对象整体使用（Depends / 显式传参），禁把配置字段拆散复制为业务模块级常量或类字段；类型换算（秒→timedelta 等）在使用点就地完成（2026-10-20 用户钉版）
- 敏感值字段一律 `SecretStr`（供应商 api_key、对象存储 secret key 等），禁 repr / 日志泄露
- 验签公钥经环境注入并支持轮换（新旧公钥按密钥标识并存验证）；签发私钥永不出现在本侧任何配置
- 复杂类型字段（list / dict / 嵌套模型）的环境变量按 JSON 字符串解析，禁自造逗号分隔等私有约定
- 默认 dev 凭据（内置 token / 空口令）必须在启动校验中拒绝，不得带病启动；`.env` 真实值禁入 git（仅 `.env.example` 模板入库）

## A.3 API 设计

- 服务前缀域钉死单一前缀（反代直通、内网绑定解除对外暴露面），禁新增第二前缀域
- 路由按业务域拆分为 `api/v1/` 各包内 APIRouter（版本为目录级分包，URL 前缀不加版本段、对外契约不变）；`main.py` 只做组装（include_router + 全局依赖 + 异常 handler + lifespan），业务路由禁堆积在 main.py
- 请求 / 响应契约模型统一落 `schemas/` 包，禁路由文件内联定义
- 请求体一律 Pydantic BaseModel 禁裸 dict 接收；响应一律声明响应模型（优先返回类型标注写法）过滤输出字段，防敏感字段泄露
- 入参校验一律字段级承载（`Field` 约束 / `Annotated` / `field_validator` / `model_validator`），禁业务函数内散装手写 if 校验（2026-10-20 用户钉版）
- 输入 / 输出模型分离，禁同一模型双向复用；id 字段一律 `str` 类型（雪花 ID 字符串出端铁律）
- 错误响应统一 `{"detail": ...}` 语义（校验错误 422 为 `{detail: [{loc, msg, type}]}`），此契约为对外冻结约定
- FastAPI 自动 OpenAPI spec 为契约唯一事实源：接口 / 模型变更必须同步 spec 提交物，纳入防漂移门禁（见 ci/python.md）
- 长连接端点（SSE / WebSocket）禁内联执行异步任务——受理与执行分离，执行一律归 worker

## A.4 数据库访问（SQLAlchemy async + PostgreSQL）

- 选型钉版：ORM = SQLAlchemy 2.x async——`models/` 仅定义 ORM 结构、`repositories/` 承载数据访问；驱动 asyncpg（psycopg 线不采用，替换须重新实测 × SQLAlchemy async 方言装载 + 真实连通）
- DDL 唯一属主：单服务项目迁移入口唯一（Alembic）；多服务共享库时 DDL 收敛唯一属主服务，本侧禁一切 DDL 与迁移脚本，只按已迁移的表结构读写
- 多服务共享库按业务功能域划分写域：本服务只写属主域数据，他属主域仅限契约明确的只读访问；新表落地先明确属主侧并同步契约，禁跨域写
- 分层与事务铁律：事务边界唯一落 services 层（显式开 / 提交 / 回滚，异常路径回滚，禁依赖隐式提交）；repositories 只做单表 / 关联查询与结果组装，禁业务逻辑、禁自带事务边界；api / workers 禁直调 repository 之外的路径（禁绕过 repo 直写 SQL / ORM）
- SQL 一律经 SQLAlchemy 表达式 / 绑定参数，禁字符串拼接 / 格式化拼 SQL
- 连接经 core/database 统一引擎与会话工厂（进程级单例 + 显式生命周期 + fail-fast）；跨请求 / 跨任务禁共享事务
- ORM 模型禁直接上浮出口：repository 出口一律映射为业务对象（Pydantic / dataclass，见 A.7）后再上浮，禁裸 ORM 实例跨 service 流转或直接序列化出 API
- 主键分布式 ID：雪花 ID 生成器统一封装（机器号分段 + 启动注册校验，冲突拒绝启动），禁散装手动生成；【全局铁律】表关系一律逻辑关联、禁止物理关联（外键等物理约束，2026-10-19 用户钉版），完整性由唯一约束与应用层保障

## A.5 基础设施生命周期

- 异步任务流一律 Redis Stream 消费组制（禁 BLMOVE / List 轮询队列模式）；任务处理必须幂等（重复投递不产生重复副作用）；重试必须有界（次数上限 + 指数退避 + 抖动）；停机语义「TERM 后完成在途任务再退出」，禁直接强杀丢弃在途任务
- Redis 客户端进程级复用（统一基础设施封装），禁每次操作新建实例；响应解码全项目钉死一种（`decode_responses=True`）；阻塞命令必须用独立连接
- 重资源（模型会话、缓存句柄、磁盘缓存等）进程级懒加载单例，禁每次调用现场加载
- FastAPI 生命周期管理 lifespan 优先（禁新增 `on_event`）；lifespan 内统一 init / shutdown 基础设施单例，关闭钩子统一释放 HTTP 客户端 / 长连接组件 / 连接池，禁泄漏到进程退出
- uvicorn `--reload` 仅限本地开发，生产禁用；多进程用 `--workers N`；容器形态每容器单进程；worker 与应用进程独立启停，均不对外暴露 HTTP 端点（出站请求经统一网关）
- 有状态组件跨实例扩展须经 Redis 共享态解耦，禁进程内存直连假设

## A.7 跨层数据对象与传参约束（本栈习语）

1. **参数对象化**：服务函数形参 > 3 必须参数对象整体传参——纯内部用 `@dataclass`（默认 `frozen=True`），需构造期校验用 `@pydantic.dataclasses.dataclass`；禁逐参罗列。仅业务确需灵活传参可例外。
2. **返回业务对象**：业务返回必须 Pydantic 模型或 dataclass 业务对象，禁裸 dict / 无结构 list 跨业务层流转；请求与响应分别建模，禁同一模型双向复用。
3. **职责隔离**：请求模型 ≠ 响应模型 ≠ 内部实体（dataclass）≠ ORM 模型（models/）≠ 消息模型 ≠ 事件模型，字段全同也按职责各自建类；禁万能对象承载多职责、禁跨层直传。
4. **例外边界**：对外边界（HTTP 入出口、消息契约、事件流、跨服务共享契约）一律 Pydantic BaseModel；日志上下文、trace 透传、JSON 序列化中间态允许 dict，须在代码注释声明业务理由；无理由的违反视为缺陷，不得合入。
5. **模型等值与字符串表示显式管控（2026-10-20 用户钉版）**：Pydantic 敏感字段（密钥 / 口令哈希 / 令牌）一律 `Field(..., repr=False)`；dataclass 等值显式管控——生成 id 字段 `field(compare=False)`、按业务自然键比较；禁默认全字段 repr / eq 泄露敏感字段或引入 id 漂移。

## C.3 目录职责骨架

> 通用分层目录骨架，按项目实际裁剪（无对应层的整目录省略，禁凑数）；目录职责唯一载体 = 目录树行尾注释（Part B 不另设目录职责清单，与 template B / C.3 约定同源）；agent / LLM 网关 / 技能系统等项目架构域（agents / llm / middleware 等）按项目架构调研补齐，不入本条目通用骨架。

```text
<项目>/
├── app/
│   ├── main.py          # 应用组装：实例、中间件、路由挂载、lifespan 生命周期；禁业务逻辑
│   ├── api/v1/          # 路由层：解析 + 校验 + 调 service；禁业务逻辑，禁直调 repository / model
│   ├── schemas/         # Pydantic 契约：HTTP 请求 / 响应、消息、事件；仅定义契约，零业务逻辑
│   ├── services/        # 业务逻辑与事务边界（编排 repositories / 网关）
│   ├── repositories/    # 数据访问：单表 / 关联查询与结果组装；禁业务逻辑、禁自带事务边界
│   ├── models/          # SQLAlchemy ORM：仅定义结构，禁查询与业务逻辑
│   ├── core/            # 基础设施单例与横切工具（database / redis / config / auth 等）
│   └── workers/         # 后台任务 worker：独立进程，业务编排调 services；数据读写经 repositories
├── tests/               # pytest（单测 + integration/ 集成层）
├── scripts/             # 运维脚本（启停 / spec 导出）
├── pyproject.toml       # 依赖与工具配置（uv 管理）
└── uv.lock              # 锁文件（入库，禁手改）
```

## C.4 常用命令骨架

```bash
uv sync                                   # 安装 / 同步依赖（按 uv.lock）
uv run uvicorn app.main:app --port 8000   # 启动服务（dev 可加 --reload）
uv run python -m app.workers.<worker>     # 启动 worker（独立进程）
uv run pytest tests/                      # 全量测试
uv run pytest tests/integration/          # 集成层（须连本地依赖服务）
uv run ruff check --fix .                 # lint 并安全修复
uv run ruff format .                      # 格式化
uv run python scripts/export_openapi.py   # spec 再生成提交物（防漂移与契约更新时必跑）
```

## 来源

- img-create 项目 `backend-ai/AGENTS.md`（Part A / C.4 / C.6，含用户钉版条款）及其 C.2 版本表实测记录
- 关键官方文档：FastAPI（lifespan / exception_handler / Annotated Depends）、pydantic-settings（配置源优先级）、SQLAlchemy 2.x async + asyncpg 方言、redis-py Stream 消费组、uv（依赖组与锁文件）
