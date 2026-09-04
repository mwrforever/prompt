# 风格示例（分层宪法文档体系）

> agent 模式样例（AGENTS.md 宪法实体 + CLAUDE.md 根索引）；claude 模式命名整体对调。章节骨架与填写规则见 `template.md`；`{...}` 生成时替换为目标项目实测事实，项目专属内容（定位、目录、技术栈版本）禁止照抄。

---

## 1. 根 AGENTS.md（定位层，≤ 200 行）

````markdown
# AGENTS.md

> 本文件为 AI 编码代理提供仓库级指引，是**定位层**：只描述项目定位、**根目录一层**结构与子项目宪法索引，不承载子项目内部深度约束。
> **强制阅读路由**：写任一子项目代码前必读其宪法（见「子项目宪法索引」）；子项目内的深度约束以子宪法为准。
> **约束效力**：本定位层（含跨切总则）与全部子项目宪法对所有开发行为强制生效，冲突代码 / 设计不得合入主分支（总则见下文「约束效力与遵从总则」）。
> 注释 / 日志 / 测试规范见全局 `~/.zcode/AGENTS.md`（§一/§二/§六），本项目不重复正文，强制生效。

## 项目定位

{项目名} 是 {一句话业务定位}。后端提供 {核心能力，如"带沙箱执行与工具扩展的 agent 运行时"}，前端为 {Web 界面一句话}。

## 运行形态

| 服务     | 端口/入口    | 职责                             |
| -------- | ------------ | -------------------------------- |
| 反向代理 | `{公网端口}` | 统一公网入口：服务前端并转发 API |
| 网关 API | `{内部端口}` | REST API + agent 运行时          |
| 前端     | `{内部端口}` | Web 界面                         |

反向代理是唯一公网入口，内部端口不对外发布；路由与运行细节见 [backend/AGENTS.md](backend/AGENTS.md)。

## 仓库地图（仅一层）

```
{项目名}/
├── Makefile                  # 根编排命令（见「命令总览」）
├── AGENTS.md / CLAUDE.md     # 定位层 / 根索引（全项目仅此一份索引）
├── TASK.md                   # 待办与登记台（{待调研项} / 待决策 / TODO）
├── CHANGELOG.md              # 工程变更记录（先记再改正文）
├── backend/                  # Python 后端 —— 规范见 backend/AGENTS.md
├── frontend/                 # TypeScript 前端 —— 规范见 frontend/AGENTS.md
├── docker/                   # 容器编排与反向代理
├── docs/
│   ├── specs/                # 功能设计与业务数据契约（宪法只引用）
│   └── agmds-research/       # 宪法调研报告
└── scripts/                  # 根编排脚本
```

（一层为准：backend / frontend 内部结构不展开，只留指针。）

## 命令总览（根 vs 子项目）

```bash
make setup    # 交互式初始化（安装依赖 + 生成配置模板）
make dev      # 全栈开发（网关 + 前端 + 反向代理，热重载）
make test     # 全栈测试
```

口诀：**根命令 = 整个应用；子项目内命令 = 单模块工作**。各子项目命令清单见其 AGENTS.md（C.4）。

## 子项目宪法索引

| 子项目    | 宪法                                     | 深度范围                                             |
| --------- | ---------------------------------------- | ---------------------------------------------------- |
| backend/  | [backend/AGENTS.md](backend/AGENTS.md)   | 运行时与路由、架构分层、持久化与迁移、配置、测试布局 |
| frontend/ | [frontend/AGENTS.md](frontend/AGENTS.md) | 页面结构、状态与数据流、视觉令牌、代码风格与命令     |

## 约束效力与遵从总则

1. **强制生效**：本定位层（含跨切总则）与全部子项目宪法对一切开发行为（编码 / 设计 / 脚本 / CI / 文档）强制生效；注释 / 日志 / 测试全局规范（`~/.zcode/AGENTS.md`）同样强制生效。
2. **不得违背**：冲突以宪法为准，冲突产物不得合入主分支；禁止「临时 / 紧急」绕过——修宪先记 `CHANGELOG.md` 再改正文，一次性事项登记 `TASK.md` 待决策并限定范围。
3. **遵从路径**：开发任一子项目前必读其宪法；跨子项目行为同时遵守本文档跨切总则。
4. **裁决顺序**：单子项目事项以其子宪法为准；跨子项目事项以本文档为准；未规定事项遵全局约束文件；全局与本项目宪法冲突时以本项目宪法（更特化）为准。

## 跨切约定

- **配套文件职责** —— [TASK.md](TASK.md) 待办与登记台（{待调研项} / 待决策 / TODO，回填后删除对应行）；[CHANGELOG.md](CHANGELOG.md) 工程变更记录（**先记变更再改正文**）；`docs/` 过程文档（含调研报告）。
- **文档同步** —— 用户可见变更更 `README.md`，规范变更更对应 `AGENTS.md`，同一变更集内完成。
- **全栈门禁** —— 合入主干前各子项目 lint + format + 测试全过；工具链细节见各子宪法。
- **CI 链总则** —— GitHub Actions 统一流水线：合入主干前各子项目流水线全过；各子项目阶段 / 产物 / 部署细则见其 AGENTS.md（C.5）。
- **跨子项目契约** —— 统一 API 前缀 `{/api/v1}`；跨组件数据契约放 `contracts/`，细节归对应子宪法。

## 去哪里深入

- 各子项目开发（写代码前必读其宪法）→ 见「子项目宪法索引」
- 功能设计与数据契约 → [docs/specs/](docs/specs/)；待办登记 → [TASK.md](TASK.md)；变更记录 → [CHANGELOG.md](CHANGELOG.md)
````

---

## 2. 子宪法示例（backend/AGENTS.md，≤ 500 行）

````markdown
# backend 宪法

> 本子项目最高规范：**只存工程原则与约束**。功能实现与业务数据契约见代码与 `specs/`，待办见 `TASK.md`，变更记录见 `CHANGELOG.md`（先记再改），本文档均不重复。
> 任何与本文档冲突的代码或设计不得合入主分支。
> 仓库定位层见根 `../AGENTS.md`：跨切总则不在此重复，说明性内容不复制，也不引用兄弟子宪法替代成文（与前端相同的约束在本文件完整成文）。
> 注释 / 日志 / 测试覆盖规范见全局 `~/.zcode/AGENTS.md`（§一/§二/§六），本项目不重复正文，强制生效。

配套文件职责（specs/ / TASK.md / CHANGELOG.md 边界）由根定位层声明，本文件只留指向、不复述。

**三段结构**：Part A Python+FastAPI 通用 / Part B 架构分层 / Part C backend 实际。

# Part A — Python+FastAPI 通用规范

## A.1 编码约束
1. UTF-8 编码，行尾 LF；Python 3.12+：type hint、async/await。
2. 异步操作用 async/await，禁止异步上下文内同步阻塞；禁止 `print()`，统一 `logging`。

## A.2 配置管理
1. 双层：`config.yaml`（非敏感，版本控制）+ `.env`（敏感，禁止提交）；敏感值以 `$VAR` 占位注入。
2. 提供并维护 `.env.example` / `config.example.yaml` 模板。

## A.3 API 设计
1. 版本化前缀 `/api/v1/...`。
2. 请求 / 响应用 Pydantic model，禁止裸 dict；统一错误响应格式与 HTTP 状态码语义。

## A.4 数据库操作
1. 表结构变更必须经 Alembic 迁移（upgrade + downgrade）。
2. 异步 session，禁止请求内同步 session；查询走索引，禁止全表扫描。

## A.5 基础设施生命周期
1. 外部连接资源（DB / 缓存 / 对象存储）进程级单例 + 显式生命周期（fail-fast），禁止请求级创建。
2. 依赖经工厂注入获取，禁止直接 import 单例变量。
3. 后台任务持强引用，防 GC 丢任务；消息消费用消费组 + 幂等 + 有界重试。

## A.6 注释 / 日志 / 测试
见全局 `~/.zcode/AGENTS.md` §一（注释）/ §二（日志）/ §六（测试覆盖）。本项目强制生效。

## A.7 跨层数据对象与传参约束
1. 公开函数 / 方法形参数量 > 3 时，必须定义参数对象（Pydantic BaseModel / dataclass）整体传参，禁止逐参罗列；仅业务功能确需灵活传参（可选配置聚合等）可例外并陈述业务理由。
2. 业务数据返回必须定义响应模型（Pydantic model），禁止裸 dict / tuple；API 请求与响应分别建模（请求模型 ≠ 响应模型），禁止同一模型双向复用。
3. 不同业务职责的对象禁止复用：即使字段完全相同也必须按职责各自建模（查询参数 ≠ 响应 VO ≠ 存储 DO），禁止万能 dict 承载多职责数据、禁止模型跨层直传。
4. 仅业务功能确需动态结构（动态字段透传等）可偏离上述条款，偏离点须能陈述业务理由；无业务理由的违反视为缺陷，不得合入。

# Part B — 架构分层

## B.1 目录职责边界
| 目录                | 边界                                         |
|---------------------|----------------------------------------------|
| `app/api/`          | 路由层：解析 + 校验 + 调 service，禁止业务逻辑 |
| `app/services/`     | 业务逻辑与事务边界                            |
| `app/repositories/` | 数据访问：单表 / 关联查询，禁止业务逻辑       |
| `app/agents/`       | agent 编排：调 service 与网关，禁直接操作 DB  |

## B.2 层级依赖（强制）
```
api -> services -> repositories -> models
agents -> services / llm
```
禁止跨层调用与反向依赖；禁止循环 import；事务边界在 service 层。

## B.3 运行时原则
单会话单 run，并发控制保证同会话仅一个活跃 run；取消、重试、过期清理遵循框架运行时语义，禁止自建并行执行栈。

## B.4 LLM 网关
模型以注册表静态声明、按职责分类，禁止硬编码模型地址与密钥；业务代码禁止直接调用模型 SDK，统一走网关并记录请求 / 响应摘要日志。

# Part C — backend 实际

## C.1 子项目定位
仓库的 API 网关与 agent 运行时：对外提供 REST API，内嵌 {框架} 工作流运行时。

## C.2 技术栈选型
| 职责          | 技术               | 版本   |
|---------------|--------------------|--------|
| Web 框架      | FastAPI            | ≥0.110 |
| ORM           | SQLAlchemy async   | 2.0+   |
| 迁移          | Alembic            | 最新稳定 |
| 数据库        | PostgreSQL         | ≥15    |
| 缓存 / 消息   | Redis              | ≥7     |
| LLM 网关      | litellm            | 最新稳定 |

（版本必须来自用户输入或勘察证据，禁止编造。）

## C.3 目录结构
（backend 内部 tree：入口、app/ 各层目录、tests/；根目录地图归定位层，不在此重复。）

## C.4 常用命令
| 命令                                   | 用途     |
|----------------------------------------|----------|
| `uv run uvicorn app.main:app --reload` | 本地启动 |
| `uv run pytest`                        | 测试     |
| `uv run ruff format .`                 | 格式化   |
| `uv run ruff check --fix .`            | lint 修复 |
| `uv run alembic upgrade head`          | 数据库迁移 |

## C.5 CI 生产落地方案
1. GitHub Actions：push / PR 触发，`main` 分支保护要求检查全过。
2. 流水线硬门禁：`ruff check` → `ruff format --check` → `pytest` → `uv build`，任一失败阻断合入。
3. 依赖缓存（uv cache）加速流水线。

## C.6 永久环境约束
1. {编码 locale 例外、依赖被阻塞的替代方案等硬约束}
2. 首次 clone 后 `pre-commit install` + `pre-commit install --hook-type pre-push`；合入主干前全测全过（CI 门禁见 C.5）。
````

---

## 3. 前端子宪法的实例化差异（frontend/AGENTS.md）

同一三段骨架，差异仅在内容实例化：

- **Part A**：框架版本、TypeScript 严格度、UI 组件栈、视觉令牌隔离（多端禁止混用）；A.3 整节省略（请求 / 响应建模由 A.7 条款 2 承载）。A.7 以 TS 习语成文：
  1. 导出函数形参数量 > 3 必须定义参数 interface / type 整体传参（组件 props 天然为对象，本条约束工具 / 服务 / hook 函数）；仅业务确需灵活传参可例外并陈述理由。
  2. API 请求与响应分别声明业务 interface / type，禁止 `any`、裸 `Record<string, unknown>` 等无结构类型承载业务数据；同一类型禁止请求 / 响应双向复用。
  3. 不同业务职责的类型禁止复用：字段完全相同也分别声明（`UserListItem` ≠ `UserCardProps` ≠ 表单模型），禁止宽泛类型兜底多职责数据。
- **Part B**：页面 / 组件 / 状态管理的架构分层（目录职责边界、数据流方向、状态管理选型约束）。
- **Part C**：C.4 覆盖 dev / build / test / lint --fix / format；C.5 前端流水线（lint → format 校验 → 测试 → build 产物 → 静态部署，与全项目 CI 链同源）；C.6 永久环境约束。
- **双端前端（如管理端 + 用户端）**：共享组件栈约束、各自门禁细则分别完整写入两份子宪法（宁可重复），禁止「共享」落款或互引。

---

## 4. 根索引（agent 模式 = `<根>/CLAUDE.md`，与宪法实体同时生成）

```markdown
# CLAUDE.md

> 本文件是编码工具入口索引，不承载规范；项目宪法唯一权威来源为 AGENTS.md 体系：
> **写后端代码必读 [backend/AGENTS.md](backend/AGENTS.md)，写前端代码必读 [frontend/AGENTS.md](frontend/AGENTS.md)**。

- 根定位层：[AGENTS.md](AGENTS.md)
- 子项目宪法：[backend/AGENTS.md](backend/AGENTS.md) · [frontend/AGENTS.md](frontend/AGENTS.md)
- 配套文件：[TASK.md](TASK.md)（登记台）· [CHANGELOG.md](CHANGELOG.md)（变更记录，先记再改正文）
```

claude 模式命名整体对调（索引为 `<根>/AGENTS.md`，链接指向 CLAUDE.md 实体）。根目录仅此一份索引，子项目目录内禁止任何索引文件；整仓单应用省去子项目宪法行。
