# 风格示例（分层宪法文档体系）

> 本文件为 `constitution-generator` 产出物的**书面风格与结构示例**：根 `AGENTS.md` 定位层 + 各技术子项目 `AGENTS.md` 宪法 + 每份 AGENTS.md 的配套 `CLAUDE.md`（md 链接索引）。示例内容为通用样例，`{...}` 占位在生成时替换为目标项目事实；段落骨架与技术栈示例可参照，**项目专属内容（定位、目录、技术栈版本等）必须整体替换为目标项目实测事实，禁止照抄**。

---

## 1. 根 AGENTS.md（定位层，正文 ≤ 200 行）

````markdown
# AGENTS.md

> 本文件为 AI 编码代理提供仓库级指引，是**定位层**：只描述项目定位、**根目录一层**结构与子项目宪法索引，不承载任何子项目内部的深度约束。
> **强制阅读路由**：写后端代码必读 [backend/AGENTS.md](backend/AGENTS.md)，写前端代码必读 [frontend/AGENTS.md](frontend/AGENTS.md)；本文档只是地图与索引，子项目内的任何深度约束以子宪法为准。
> 子项目宪法索引：
> - [backend/AGENTS.md](backend/AGENTS.md) —— API 网关与 agent 运行时、架构分层、持久化与迁移、配置系统、测试布局
> - [frontend/AGENTS.md](frontend/AGENTS.md) —— 页面结构、状态与数据流、视觉令牌、代码风格与命令

## 项目定位

{项目名} 是 {一句话业务定位}。后端提供 {核心能力，如"带沙箱执行与工具扩展的 agent 运行时"}，前端为 {Web 界面一句话}，{外部接入 / 可选服务一句话，无则省略}。

## 运行形态

| 服务     | 端口/入口    | 职责                             |
| -------- | ------------ | -------------------------------- |
| 反向代理 | `{公网端口}` | 统一公网入口：服务前端并转发 API |
| 网关 API | `{内部端口}` | REST API + 内嵌 agent 运行时     |
| 前端     | `{内部端口}` | Web 界面                         |

反向代理是唯一公网入口，内部端口不对外发布；{原则级协作关系 1–2 句}。路由与运行细节见 [backend/AGENTS.md](backend/AGENTS.md)。

## 仓库地图（仅一层）

```
{项目名}/
├── Makefile                  # 根编排命令（见下「命令总览」）
├── config.example.yaml       # 模板 → 复制为 config.yaml（根运行时配置，实文件 gitignore）
├── AGENTS.md / CLAUDE.md     # 定位层 / 其 md 链接索引
├── TASK.md                   # 待办与登记台（{待调研项} / 待决策 / TODO 工单）
├── CHANGELOG.md              # 工程变更记录（宪法修订先记此处再改正文）
├── backend/                  # Python 后端 —— 规范见 backend/AGENTS.md
├── frontend/                 # TypeScript 前端 —— 规范见 frontend/AGENTS.md
├── docker/                   # 容器编排与反向代理配置
├── scripts/                  # 根编排脚本
├── docs/                     # 跨切文档（agmds-research 调研报告在此）
└── tests/                    # 根级测试
```

（仅一层；backend / frontend 内部结构一个字不展开，只留指针。）

## 命令总览（根 vs 子项目）

```bash
make setup    # 交互式初始化（安装依赖 + 生成配置模板）
make dev      # 全栈开发（网关 + 前端 + 反向代理，热重载）
make test     # 全栈测试
```

口诀：**根命令 = 整个应用；子项目内命令 = 单模块工作**。各子项目命令清单见其 AGENTS.md。

## 子项目宪法索引

| 子项目    | 宪法                                     | 深度范围                                             |
| --------- | ---------------------------------------- | ---------------------------------------------------- |
| backend/  | [backend/AGENTS.md](backend/AGENTS.md)   | 运行时与路由、架构分层、持久化与迁移、配置、测试布局 |
| frontend/ | [frontend/AGENTS.md](frontend/AGENTS.md) | 页面结构、状态与数据流、视觉令牌、代码风格与命令     |

## 跨切约定

repo 级总则，细节归对应子宪法：
- **配套文件职责** —— [TASK.md](TASK.md) 待办与登记台（{待调研项} / 待决策 / TODO 工单，回填后删除对应行）；[CHANGELOG.md](CHANGELOG.md) 工程变更记录（宪法修订与重大变更，**先记变更再改正文**）；`docs/` 过程文档（含 agmds-research 调研报告）。
- **文档同步策略** —— 用户可见变更更 `README.md`，架构/开发规范变更更对应 `AGENTS.md`，同一变更集内完成。
- **全栈门禁** —— 合入主干前各子项目 lint + format + 测试全过；工具链细节见各子宪法。
- **跨子项目契约** —— 统一 API 前缀 `{/api/v1}`；跨组件数据契约放 `contracts/`，细节归对应子宪法。

## 去哪里深入

- 后端开发（写代码前必读）→ [backend/AGENTS.md](backend/AGENTS.md)
- 前端开发（写代码前必读）→ [frontend/AGENTS.md](frontend/AGENTS.md)
- 待办登记 → [TASK.md](TASK.md)；变更记录 → [CHANGELOG.md](CHANGELOG.md)
- 安装 / 贡献 / 安全策略 → README.md / CONTRIBUTING.md / SECURITY.md
````

---

## 2. 子宪法示例（以 backend/AGENTS.md 为例，正文 ≤ 500 行）

````markdown
# backend 宪法

> 本子项目最高规范：**只存工程原则与约束**。功能实现与业务数据契约见代码与 `specs/` 设计文档，待办登记见 `TASK.md`，工程变更记录见 `CHANGELOG.md`（宪法修订先记 CHANGELOG 再改正文），本文档均不重复。
> 任何与本文档冲突的代码或设计不得合入主分支。
> 仓库定位层（项目定位、一层目录地图、跨切总则）见根 `../AGENTS.md`，本文档只管本子项目深度。
> 注释 / 日志 / 测试覆盖规范见全局 `~/.zcode/AGENTS.md`（§一/§二/§六），本项目不重复正文，强制生效。

**配套文件职责**：
- `specs/` —— 功能设计与业务数据契约的唯一权威来源，宪法不重复；
- `TASK.md` —— 待办与登记台：调研不可得项、待决策项、TODO 工单；
- `CHANGELOG.md` —— 工程变更记录（根级，各子宪法共用），先记变更再改正文。

**三段结构**：Part A Python+FastAPI 通用 / Part B 架构分层 / Part C backend 实际。

# Part A — Python+FastAPI 通用规范

## A.1 编码约束
1. UTF-8 编码，行尾 LF；Python 3.12+：type hint、async/await。
2. 异步操作用 async/await，禁止异步上下文内同步阻塞调用。
3. 禁止 `print()`，统一 `logging`。

## A.2 配置管理
1. 双层：`config.yaml`（非敏感，版本控制）+ `.env`（敏感，禁止提交）；敏感值以 `$VAR` 占位注入。
2. 提供并维护 `.env.example` / `config.example.yaml` 模板。

## A.3 API 设计
1. 版本化前缀 `/api/v1/...`。
2. 请求/响应用 Pydantic model，禁止裸 dict；统一错误响应格式与 HTTP 状态码语义。

## A.4 数据库操作
1. 表结构变更必须经迁移工具（含 upgrade + downgrade）。
2. 异步 session，禁止请求内同步 session；查询走索引，禁止全表扫描。

## A.5 基础设施生命周期
1. 外部连接资源（DB / 缓存 / 对象存储）进程级单例 + 显式生命周期（fail-fast），禁止请求级创建。
2. 依赖经工厂注入获取，禁止直接 import 单例变量。
3. 后台任务持强引用，防 GC 丢任务；消息消费用消费组 + 幂等 + 有界重试。

## A.6 注释 / 日志 / 测试
见全局 `~/.zcode/AGENTS.md` §一（注释）/ §二（日志）/ §六（测试覆盖）。本项目强制生效。

# Part B — 架构分层

## B.1 目录职责边界
| 目录 | 边界 |
|------|------|
| `app/api/`          | 路由层：解析 + 校验 + 调 service，禁止业务逻辑 |
| `app/services/`     | 业务逻辑与事务边界 |
| `app/repositories/` | 数据访问：单表/关联查询，禁止业务逻辑 |
| `app/agents/`       | agent 编排：调 service 与网关，禁止直接操作 DB |

## B.2 层级依赖（强制）
```
api -> services -> repositories -> models
services -> llm / core
agents -> services / llm
```
1. 禁止跨层调用与反向依赖；2. 禁止循环 import；3. 事务边界在 service 层。

## B.3 运行时原则
1. 单会话单 run，并发控制保证同会话仅一个活跃 run。
2. 取消、重试、过期清理遵循框架运行时语义，禁止自建并行执行栈（以调研为准）。

## B.4 LLM 网关
1. 模型以注册表静态声明、按职责分类，禁止硬编码模型地址与密钥。
2. 禁止业务代码直接调用模型 SDK，统一走网关并记录请求/响应摘要日志。

# Part C — backend 实际

## C.1 子项目定位
仓库的 API 网关与 agent 运行时：对外提供 REST API，内嵌 {框架} 工作流运行时。

## C.2 技术栈选型
| 职责 | 技术 | 版本 |
|------|------|------|
| Web 框架 | FastAPI | ≥0.110 |
| ORM | SQLAlchemy async | 2.0+ |
| 迁移 | Alembic | 最新稳定 |
| 数据库 | PostgreSQL | ≥15 |
| 缓存 / 消息 | Redis | ≥7 |
| LLM 网关 | litellm | 最新稳定 |

（版本必须来自实测证据，禁止编造。）

## C.3 目录结构
（backend 内部 tree：配置文件、入口、app/ 各层目录、tests/；根目录一层地图归定位层，不在此重复。）

## C.4 永久环境约束 + 门禁落地
1. {编码 locale 例外、依赖被阻塞的替代方案等硬约束}
2. 首次 clone 后 `pre-commit install` + `pre-commit install --hook-type pre-push`；合入主干前 lint + format + 全测全过。
````

---

## 3. 配套 CLAUDE.md（每份 AGENTS.md 一份，纯 md 链接索引）

### 3.1 根 CLAUDE.md（`<根>/CLAUDE.md`）

```markdown
# CLAUDE.md

> 本文件是编码工具入口索引，不承载规范；项目宪法唯一权威来源为 AGENTS.md 体系，经以下链接进入：
> **写后端代码必读 [backend/AGENTS.md](backend/AGENTS.md)，写前端代码必读 [frontend/AGENTS.md](frontend/AGENTS.md)**。

- 根定位层（项目定位、一层目录地图、子项目宪法索引、跨切总则）：[AGENTS.md](AGENTS.md)
- 子项目宪法：[backend/AGENTS.md](backend/AGENTS.md) · [frontend/AGENTS.md](frontend/AGENTS.md)
- 配套文件：[TASK.md](TASK.md)（待办与登记台）· [CHANGELOG.md](CHANGELOG.md)（工程变更记录，先记变更再改正文）
```

### 3.2 子项目 CLAUDE.md（如 `<根>/backend/CLAUDE.md`）

```markdown
# CLAUDE.md

> 本文件是编码工具入口索引，不承载规范；本子项目宪法唯一权威来源：[AGENTS.md](AGENTS.md)——**写本子项目代码前必读**。
> 仓库定位层见 [../AGENTS.md](../AGENTS.md)；根级配套文件见 [../TASK.md](../TASK.md) 与 [../CHANGELOG.md](../CHANGELOG.md)。
```

---

## 4. 前端子宪法的实例化差异

前端子宪法套用同一三段骨架（见 `template.md` 文档头 ※ 注），差异仅在内容实例化：
- **Part A** = 前端技术栈通用：框架版本、TypeScript 严格度、UI 组件栈、**视觉令牌隔离**（多端时禁止混用）；A.3 API 设计整节省略。
- **Part B** = 页面 / 组件 / 状态管理的架构分层（组件目录职责边界、数据流方向、状态管理选型约束）。
- **Part C** 同构：前端子项目定位、技术栈选型表、前端内部目录 tree、lint / format / 测试门禁（ESLint + Prettier + Vitest 等，与后端工具链对齐）。

---

## 风格要点小结

1. 文档体系只有一种形态：根定位层（项目定位、运行形态、**一层**仓库地图、命令总览、子项目宪法索引、跨切总则、去哪里深入）+ 每个技术子项目一份子宪法（Part A 技术栈通用 / Part B 架构分层 / Part C 子项目实际）+ 每份 AGENTS.md 一份配套 CLAUDE.md。
2. 定位层文档头必含**强制阅读路由**（写后端代码必读 backend/AGENTS.md、写前端代码必读 frontend/AGENTS.md，逐子项目列全）；CLAUDE.md 以 md 链接复现该路由，保证任何以 CLAUDE.md 为入口的工具都能进入宪法体系。
3. 定位层禁止展开子项目内部结构，技术子项目行尾只留「规范见 <子项目>/AGENTS.md」指针；子项目内部目录树、深度约束、全量命令一律归子宪法。
4. 强制条款用编号列表 + 禁令表述（"禁止…""必须…""违反任一均为缺陷"）；表格化：运行形态、目录职责边界、技术栈选型、子项目宪法索引。
5. 依赖方向用 text 图；引用链双向互通（定位层索引 ↔ 子宪法头注反指）但同一规则只落一处，引用（含章节号与路径）不算重复正文。
6. 子宪法只存工程原则与约束：业务数据模型、图模型、状态机等设计细节归 `specs/`（只留职责引用行）；变更记录归 `CHANGELOG.md`（正文不设变更控制章节）；TASK.md / CHANGELOG.md 作为根级配套文件在定位层仓库地图与跨切约定中标注职责。
7. 跨切约定只写总则并声明「细节归对应子宪法」；约束不串写——前端约束只进前端子宪法，后端约束只进后端子宪法。
