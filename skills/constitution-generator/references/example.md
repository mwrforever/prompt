# 风格示例（源自真实项目 MindSoar 的 AGENTS.md）

> 本文件为 `constitution-generator` 产出物的**书面风格与结构示例**，内容源自用户提供的 MindSoar 项目宪法（真实项目，2026-06-28 定稿四段结构，后持续修订）。
> 生成新宪法时：Part A–D 的段落骨架通用（Part B/C 按技术栈裁剪），但 Part D 为「项目实际」——其中的定位、目录结构、数据模型、图模型、变更记录等**项目专属内容必须整体替换为目标项目事实，禁止照抄**。结构要点见文末小结。

---

# MindSoar 项目宪法

> 项目最高规范：**只存原则、约束与数据契约**。功能实现细节见代码与 `TASK.md`，本文档不重复。
> 任何与本文档冲突的代码或设计不得合入主分支。
> 注释 / 日志 / 测试覆盖规范见全局 `~/.Codex/AGENTS.md`（§一/§二/§六），本项目不重复正文，强制生效。

**四段结构**：Part A Python+FastAPI 通用 / Part B agent 项目 / Part C 前端 / Part D MindSoar 实际。

---

# Part A — Python+FastAPI 通用规范

## A.1 编码约束
1. UTF-8 编码，行尾 LF，禁止 CRLF。
2. Python 3.12+：type hint、async/await、match。
3. SQLAlchemy 2.0+ 风格（`Mapped`/`mapped_column`/`select()`），禁止遗留 Query API。
4. 异步操作用 async/await，禁止异步上下文内同步阻塞调用。
5. 禁止 `print()`，统一 `logging`。

## A.2 配置管理
1. 双层：`config.yaml`（非敏感，版本控制）+ `.env`（敏感，禁止提交）。
2. 敏感值以 `$VAR` 占位，pydantic-settings 从 `.env`/环境变量替换。
3. `.env.example` / `config.example.yaml` 提供模板。

## A.3 API 设计
1. 版本化前缀 `/api/v1/...`。
2. 请求/响应用 Pydantic model，禁止裸 dict。
3. 统一错误响应格式；HTTP 状态码语义（200/201/400/404/500）。

## A.4 数据库操作
1. 表结构变更必须经 Alembic 迁移（含 upgrade + downgrade）。
2. 异步 session，禁止请求内同步 session。
3. 查询用索引优化，禁止全表扫描。

## A.5 基础设施生命周期
基础设施层（DB / Redis / Neo4j / MinIO / 媒体生成池）遵循以下原则，违反任一均为缺陷：
1. **进程级单例**：每个外部连接资源在进程内单例，禁止请求级创建。
2. **显式生命周期**：FastAPI lifespan 启动调 `init_infra()`、关闭调 `shutdown_infra()`；未初始化被访问抛 RuntimeError（fail-fast），禁止惰性 init。
3. **Depends 工厂 DI**：经 `Depends(get_xxx)` 获取，禁止 endpoint 直接 import 单例变量。
4. **专用线程池**：同步 SDK 经 `asyncio.to_thread` 调度到专用 `ThreadPoolExecutor`，禁止复用默认 event loop executor。
5. **provider 直连厂商 SDK**：图像/视频生成直接调用厂商 SDK，不经 litellm；litellm 仅服务文本/视觉提示词（见 Part B.4）。
6. **后台任务强引用**：`asyncio.create_task` 须持强引用集合 + 完成 callback 剔除，防 GC 丢任务。

## A.6 注释 / 日志 / 测试（引用全局）
见 `~/.Codex/AGENTS.md` §一（注释）/ §二（日志）/ §六（测试覆盖）。本项目强制生效。

---

# Part B — agent 项目规范（LangGraph 工作流通用）

## B.1 目录职责边界
| 目录 | 边界 |
|------|------|
| `app/agents/` | Agent 工作流编排（多步骤业务流程，调 service + LLM 网关）。Agent 间经共享 DB 状态协作，禁止直接互调。 |
| `app/middleware/` | LangGraph 横切中间件（提示词/压缩/HITL/引用/技能/循环/沙箱/审计）。每关注点一个中间件，禁止业务编排。 |
| `app/workers/` | Redis Stream 异步 Worker（独立进程，消费组水平扩容）。调 Agent/Service，禁止直接操作 DB。 |
| `app/llm/` | LLM 网关（litellm 文本/视觉 + provider 直连图像/视频）。禁止硬编码模型地址与密钥。 |

非 agent 层（core/models/schemas/repositories/services/api）见 Part D.3。

## B.2 层级依赖（强制）
```
api/v1 -> services -> repositories -> models
services -> llm / core
agents -> services / llm / schemas / middleware
workers -> agents / services
```
1. 禁止跨层调用（API 禁直接调 repository/model）。
2. 禁止反向依赖（低层禁导入高层）。
3. 禁止循环 import。
4. 事务边界在 service 层（repository 只做单表/关联查询）。

## B.3 运行时原则
平台核心是**工作区驱动的运行循环**：工作区发起 run，Worker 驱动图，事件流式推 SSE，interrupt 处暂停等用户决定。
1. **工作区 = 图会话**：作用域 `(user, graph_type, world_id, project_id)`；不同工作区 run + SSE 流天然隔离。
2. **投递 / 流式 / 取消走 Redis Stream + 消费组**：worker 独立进程水平扩容；全异步，无 arq/Celery。
3. **run 生命周期**：API 投递 → worker 驱动图 → 事件流式推 SSE；LLM 消息在消息边界落库。
4. **单会话单 run + 崩溃交接防双跑**：并发控制保证同会话同时仅一个活跃 run；worker 崩溃经 fencing 交接，防同 run 双跑。
5. **两类 interrupt**：节级续跑（workflow 节点）+ 步级确认（ConfirmationMiddleware），单一 `mode`（auto/manual）统控，状态靠 checkpoint。
6. **cancel 带回滚**：取消掐断在途执行 + 回滚到 pre-run 状态。
7. **崩溃有界重试**（非无缝恢复）：重试次数耗尽标 failed，前端重试。
8. **HITL 过期清理**：interrupted 态 run 超期标 abandoned。
9. **串行写入**：连续性敏感产出（资产/剧本/台词/视频提示词）按序串行写；并行只用于收集/起草。
10. worker 经 service 操作 DB，禁止直接操作。

## B.4 LLM 网关
1. 文本/视觉提示词 LLM 调用走 `app/llm/` litellm 网关；图像/视频生成走 provider 直连厂商 SDK（Part A.5），不经 litellm。禁止 service/agent 直接调模型 SDK。
2. 模型以**注册表**静态声明在 `config.yaml`，按职责（duty）分类，每分类 ≥1 默认；前端按分类选择。**禁止 DB 动态路由、禁止前端运行时编辑模型定义**。
3. 敏感凭证 `$VAR` 占位，`.env` 注入。
4. LLM 调用记录请求/响应摘要日志，禁止打印完整响应体。
5. LLM 消息在消息边界落库。
6. **duty 路由**：worker 暴露 `model_overrides` 到 configurable，支持子图 factory-time 据用户选 duty 解析模型。

## B.5 横切关注点（原则）
1. **工具装配**：多来源（builtin / config / MCP / subagent / ACP）+ 能力策略 + 技能 allowed-tools 白名单收缩。
2. **沙箱**：抽象层 + 虚拟路径契约 + 隔离 + 双向路径翻译。
3. **子代理委派**：`task()` 协议 + 并发硬上限 + guardrail（turn / token / loop）。
4. **技能系统**：`SKILL.md` 协议 + per-user 存储 + 显式激活。
5. **HITL 确认中间件**：manual 走白名单，未命中 interrupt 给三决定。
6. **引用注入中间件**：workspace + `@ref` + 截断，控上下文不超 token。
7. **上下文压缩中间件**：摘要触发 + 历史替换 + 循环检测 + token 预算硬上限。

---

# Part C — 前端规范（Next.js 双端，2026-08-16 生效）

## C.1 工程形态与技术栈
两个独立 Next.js app（2026-08-16 P0 拍板，用户选定：双端独立部署演化，接受共享层重复建设，不引入 monorepo）：
- `web/` —— 用户端（暖白「Warm Ivory」令牌，定稿见 `design-system/mindsoar-web/MASTER.md`）
- `admin/` —— 平台管理端（亮色「Indigo Admin」令牌）

栈：Next.js（App Router）+ TypeScript + Tailwind CSS + shadcn/ui + Zustand + TanStack Query + Axios + React Flow（画布）+ AntV G6（图谱）+ lucide-react（双端统一图标）+ react-markdown/remark-gfm（消息流 markdown 渲染，web 端）。API 真后端优先直连 `/api/v1`（A.3 前缀不变）。两套视觉令牌独立维护、**禁止混用**（核心值见 `design/README.md`）。

## C.2 ESLint + Prettier + 测试（强制，已随启动落地）
每个 app 必须：ESLint（flat config）+ Prettier + Vitest + Testing Library，与后端 ruff + pytest 对齐；合并主干前 lint + format + 测试全过。Stylelint / Playwright / Husky + lint-staged / CI 前端 job 仍 defer（`TASK.md` §F）。

---

# Part D — MindSoar 项目实际

## D.1 系统定位
基于时间线感知的漏斗式映射系统：**小说 → 剧本 → 提示词 → 视频**，层间多对多映射。

## D.2 技术栈选型（强制）
| 职责 | 技术 |
|------|------|
| Web 框架 | FastAPI ≥0.110 |
| ORM | SQLAlchemy 2.0+ async |
| 迁移 | Alembic |
| 数据库 | PostgreSQL ≥15（JSONB） |
| 图数据库 | Neo4j Community ≥5 |
| 缓存 / Stream | Redis ≥7 |
| 对象存储 | MinIO（S3 兼容） |
| 异步任务 | Redis Stream + 消费组（无 arq） |
| 校验 | Pydantic v2 |
| 配置 | config.yaml + pydantic-settings |
| Python | ≥3.12 |
| 依赖管理 | uv |
| LLM 网关 | litellm（文本/视觉） |
| 图像/视频生成 | dashscope + volcengine-python-sdk[ark]（provider 直连） |

## D.3 目录结构
```
MindSoar/
├── docker-compose.yml          # 中间件编排（PG / Neo4j / Redis / MinIO）
├── config.yaml / config.example.yaml   # 非敏感配置 / 模板
├── .env.example                # 敏感变量模板
├── pyproject.toml              # 依赖（uv）
├── alembic.ini + alembic/      # 数据库迁移
├── TASK.md                     # 后续实现待办索引
├── web/                        # 用户端 Next.js app（Indigo Dream 暗令牌）
├── admin/                      # 管理端 Next.js app（Indigo Admin 亮令牌）
├── app/
│   ├── main.py                 # FastAPI 入口 + lifespan
│   ├── core/                   # 基础设施（DB / Redis / Neo4j / MinIO / locks / config）
│   ├── models/                 # SQLAlchemy ORM（仅定义结构）
│   ├── schemas/                # Pydantic 请求/响应契约
│   ├── api/v1/                 # 路由层（解析 + 校验 + 调 service）
│   ├── services/               # 业务逻辑层（事务边界）
│   ├── repositories/           # 数据访问层（单表/关联查询）
│   ├── llm/                    # LLM 网关
│   ├── agents/                 # Agent 工作流
│   ├── middleware/             # LangGraph 横切中间件
│   └── workers/                # Redis Stream Worker
└── tests/                      # 单元 + 集成测试
```

## D.4 三级漏斗数据模型
```
小说层 (Novel) -> 章 (Chapter)
                | 多对多
            剧本层 (Script) -> 节 (Section)
                            | 一对多
                        提示词层 (Prompt) -> 镜头 (Shot)
```
1. 剧本节与小说章节**多对多**，每节记录来源章节区间（不可空，可追溯原文）。
2. 镜头与剧本节**一对多**，每镜头关联剧本节。
3. 映射关系创建/变更记审计日志。

## D.5 Neo4j 图模型（数据契约）
节点：`Character` / `Alias` / `CharacterState` / `TrueIdentity` / `VisualIdentity` / `PlotLine` / `KeyEvent` / `Mission` / `Scene`。
关系：`HAS_ALIAS` / `HAS_STATE` / `TRUE_IDENTITY` / `VISUAL_IDENTITY` / `CONTAINS_EVENT` / `OCCURS_IN` / `INVOLVES_CHARACTER` / `DRIVEN_BY` / `ASSIGNED_TO`。

查询规则：
1. 查人物状态须指定章节区间，返回区间内最近状态快照。
2. 查事件沿 PlotLine → KeyEvent → Scene + CharacterState + Mission 路径。
3. **身份解耦**：剧本生成用 TrueIdentity，提示词生成强制用 VisualIdentity。

> 本节为 v1 基线；时序资产图谱 v2（双轨语义 + image_url + novel_id 隔离 + 软删除）见 `md/arch/v2/关系架构设计.md`，落地后回写。

## D.6 变更控制
1. 本文件为项目最高规范，修改需记录原因和日期。
2. 新增模块 / 调整层级依赖 / 换技术栈须先更新本文件。
3. 代码审查以本文件为第一准则。
4. **变更记录**（简述，详见 git 历史）：
   - 2026-06-28 · v2 架构：异步传输 Redis Stream；配置双层 yaml + env；LLM 网关 litellm 按职责分类。
   - 2026-07-01 · 移除 arq，纯 Redis Stream + 消费组；单会话单 run 改 PG 唯一索引保证。
   - 2026-07-20 · 宪法四段重构 + IDOR 全量 retrofit + CI/CD pre-commit + 全局 CI/CD 强制段。
   - 2026-08-12 · CI 增强：激活本地 hook；basedpyright 类型检查（include 限定 app，排除 tests/迁移/提示词）；coverage 防回退门禁（fail_under=75）；GitHub Actions 云端门禁（push main/dev + PR，ruff+basedpyright+pytest+coverage 全链）。
   - 2026-08-13 · Spec 1b 无限画布：26 种 GenerationMode（spec 标题「27」系 off-by-one 笔误）+ provider 统一 MediaGenRequest/submit+poll + mode dispatch（image 9/video 7/audio 3）+ audio provider（ali/bytedance/minimax）+ local_processor（PIL/ffmpeg 6 本地 mode）+ media_worker canvas_node/mode/local dispatch + canvas 三表 repo + DAG 环检测（DFS+Kahn）+ canvas_service（CRUD/node/edge/generate/upload/import-export）+ canvas API（17 端点）+ models available?mode；真厂商 API 细节 defer TASK.md #13（audio poll + image/video 编辑 submit + 3 local mode NotImplementedError+TODO）。
   - 2026-08-13 · Spec 1d 流水线进度：novel_ingest target 驱动（§5.1，spec4 T9 已落地：target 默认 1 + 移除 skip）+ GET /worlds/{world_id}/pipeline_status 聚合端点（§5.2：PipelineService 只读 COUNT 四级漏斗 + 活跃 run + Neo4j 活跃 CharacterState 区间并集覆盖章数）；explore 驱动逻辑（§5.3）属 Spec 4/5 范围已完成。9 份架构 spec 全部实施完毕（1a/1b/1c/1d/2/3/4/5/6）。
   - 2026-08-16 · Part C 前端栈改版：React 18 + Vite → **两个独立 Next.js app（web/ 用户端 + admin/ 管理端）** + 锁定组件栈（Tailwind/shadcn/Zustand/TanStack Query/Axios/React Flow/G6）+ lucide-react + 真后端直连 `/api/v1`；管理端 8 缺口端点拍板补后端真实现。原因：设计稿阶段收官转入正式实现，P0 五项立项决议用户逐项拍板（进度文档 §2.1）；ESLint + Prettier + Vitest 随脚手架落地，TASK.md §F 剩余项继续 defer。
   - 2026-08-23 · Part C.1 栈清单补记 react-markdown + remark-gfm（消息流 markdown 渲染，随 `a810c11` web 工作台引入、产品契约见 `design/README.md` §六；按 D.6 补回写，非新增引入）+ 双端新增 `typecheck` 脚本（`next typegen` 前置，干净 checkout 裸 tsc 不再依赖 .next 生成物报 TS2304）。

## D.7 永久环境约束 + 后续实现索引

**永久环境约束**（Windows GBK 例外，永久构建/编码规则，非待办）：
- `alembic.ini` 必须 ASCII / 英文注释（GBK locale 读 ini 报错）。
- 离线 SQL 需 `PYTHONIOENCODING=utf-8`（迁移 comment 含非 GBK 字符时）。
- xxhash `_xxhash.pyd` 被 Windows WDAC 阻塞 → venv 内纯 Python stub 覆写（策略解除后 `pip install --force-reinstall xxhash` 恢复）。

**后续实现功能索引**（详情见 `TASK.md`；格式 `{实现时间}:{TASK.md:行号}`；已实现不列，git 可查）：

| 功能 | 索引 |
|------|------|
| Docker sandbox provider | {待定}:{TASK.md #7, L10-15} |
| config-defined 工具反射 + deferred | {待定}:{TASK.md #8, L16-21} |
| react factory-time model 真验证 | {待定}:{TASK.md #11, L22-27} |
| 图片/视频真厂商端到端 | {待定}:{TASK.md #13, L28-38} |
| 图片后台任务重启安全 | {待定}:{TASK.md #14, L39-44} |
| media-gen 专用池 | {待定}:{TASK.md #15, L45-52} |
| `shutdown_media_pool` 优雅关停（核实无需改） | {待定}:{TASK.md §B, L59} |
| `_to_dict` 收敛（评估不做） | {待定}:{TASK.md §B, L60} |
| 真厂商测试覆盖缺口 | {待定}:{TASK.md §B, L61} |
| 前端 CI 链剩余项（Stylelint/Playwright/Husky；TS/ESLint/Prettier/Vitest 已随双 app 脚手架落地 2026-08-16） | {待定}:{TASK.md §F, L92} |
| 媒体链路流式传输 + IO 优化 deferred（OPT-9/11/13；OPT-7/8/10/12 已落地 2026-08-16） | {待定}:{TASK.md §J, L162-168} |

**本地 hook**：首次 clone 后 `pre-commit install` + `pre-commit install --hook-type pre-push`（pre-commit 跑 ruff check + format + 通用校验；pre-push 跑 pytest 全测）。合并主干前 lint + format + 全测必须全过，见 `.pre-commit-config.yaml`。

---

## 风格要点小结

1. 头注三要素：定位声明（只存原则/约束/数据契约）+ 违者不合入主分支 + 引用全局规范（含章节号，不重复正文）
2. 四段结构：Part A 后端通用 / Part B 架构层（本示例为 agent 项目）/ Part C 前端 / Part D 项目实际——每段以 `#` 级标题分隔，段内 `# Part X` 下按 A.1/A.2 编号
3. 段落按技术栈裁剪：无对应组件则省略整段/整节，编号不重排
4. 强制条款用编号列表 + 禁令表述（"禁止…""必须…""违反任一均为缺陷"）
5. 表格化：目录职责边界、技术栈选型、后续实现索引
6. Part D 承载项目专属事实：系统定位、技术栈选型表、目录树、数据模型/图模型契约（含查询规则）、变更控制（记录每次修订）、永久环境约束 + 后续实现索引
7. 依赖方向用 text 图（`api/v1 -> services -> …`），数据契约含关系名清单与查询规则