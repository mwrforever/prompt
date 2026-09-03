# prompt-skills · 提示词工程技能包

生产规范化文档的生成技能集，三类产出：

- **交付执行文档**：基于用户需求或已定项 spec，产出带阶段门禁、循环/中止机制、DoD 目标设定的执行文档
- **项目宪法**：基于项目技术栈，派发 subagent 调研各厂商最佳实践，产出分层宪法体系——根文件只做定位层（一层目录地图 + 子项目宪法索引），各技术子项目目录各生成一份宪法承载宪法深度，只存工程原则与约束；整个体系只配一份根级索引文件（与宪法体系命名相反），经链接间接可达全部宪法
- **任务交接文档**：手动触发，把当前会话任务固化成交接文档并显式声明「未完成部分须在新会话执行」，生成后当前会话封存、不再继续实现

## 当前技能

| 技能 | 用途 | 输出 |
| --- | --- | --- |
| `loop-graph-designer`（交付 Loop/Graph 设计器） | 手动触发（`/loop-graph-designer [spec\|input\|skill]` 或「设计交付 loop / 按 spec 产出执行文档」等，无子命令默认 input 模式），触发即加载本技能。只设计交付执行提示词文档、绝不亲自实现功能：spec 子命令基于功能文档确认实际内容后按「一份 spec = 一个任务」排期生成，不自行探索实现方案；input 子命令基于现有代码与用户逐问澄清、提出 2-3 种实现方案供确认后再排期，不额外产出 spec；skill 子命令把指定协同技能显式编排进执行文档对应阶段。文档内容只来自用户本次输入与目标项目实际探查，禁止从历史记忆或旧文档搬运；初稿交付前必经成品审核优化循环（信息来源、相关性、核心流程链路、三原则四项审计，未通过不得交付）。文档原则：精确、完整、无冗余；每份 spec = 一个任务、需求按复杂性模块化拆分（不过度拆分）、loop/graph 编排、阶段门禁与中止条件、subagent 执行制、Definition of Done | `/docs/prompt/{yyyy-MM-dd}-{loop|graph}-{任务标题}.md` |
| `constitution-generator`（项目宪法生成器） | 仅限手动调用（`/constitution-generator [claude\|agent] {用户指令}`，无子命令默认 agent 模式），触发即加载本技能。根据项目技术栈生成分层项目宪法体系：按技术栈组件族划分调研主题，派发 subagent 调研各技术栈官方文档与厂商最佳实践，提炼为「只存工程原则与约束」的条款（业务数据契约归 `specs/` 职责行、变更记录外置 `CHANGELOG.md`）。归属判定从严：仅**所有**子项目共有的硬约束才上收根定位层总则，子项目约束（含双端前端相同的门禁细则）全文写入各自子宪法、宁可重复不写「共享」；说明性内容（定位 / 目录地图 / 目录结构）不重复；各子宪法必含常用命令节（打包 / 测试 / 格式化 / 修复 / 启动）；引用只认当前工作区实存（`specs/` 等设计文档禁止凭项目历史 / 全局记忆关联旧文档）。子命令决定宪法体系命名：agent 模式（默认）= AGENTS.md 宪法体系（根定位层 + 各子项目宪法）+ 根 `CLAUDE.md` 唯一索引；claude 模式 = CLAUDE.md 宪法体系 + 根 `AGENTS.md` 唯一索引；全项目仅根目录一份索引文件，经 md 链接间接可达全部子宪法，子项目目录内不生成索引文件 | agent 模式：`<目标项目根>/AGENTS.md`（定位层）+ `<子项目>/AGENTS.md` ×N + 根索引 `CLAUDE.md`；claude 模式命名整体对调；另含 `/docs/agmds-research/{yyyy-MM-dd}-{主题}.md` 调研报告；整仓单应用时仅根宪法 + 根索引两份 |
| `session-handoff`（会话交接器） | 仅限手动触发（`/session-handoff [update\|rewrite]` 或「保存进度 / 交接 / 生成交接文档 / 续接准备」等），触发即加载本技能。只生成/更新进度交接文档、绝不实现任务：默认（rewrite）在项目 docs/progress/ 新建交接文档（同名追加 -2、-3 序号，不覆盖历史），update 原地更新当前会话内引入的进度文件（本会话生成的或续接时读取的那篇，绝不取自历史记忆），项目内无符合规范的进度文件时退化为 rewrite；文档内容仅来自当前会话的设计实现，禁止叠加引用上一份进度文档或历史记忆中的旧文档。顶部显式声明未完成任务须在新会话执行；生成并回报后当前会话立即封存，停止一切任务实现，后续续做请求一律引导至新会话 | `<目标项目根>/docs/progress/{yyyy-MM-dd}-{精简标题}.md` |

文档形态：**loop**（序贯阶段 + 门禁 + 修复循环）或 **graph**（并行分支 DAG + 汇合门禁 + 回退边），按任务是否存在独立可并行分支判定。

## 兼容性

技能本体采用 [agentskills.io](https://agentskills.io) 开放标准（目录 + `SKILL.md`），该标准被 zcode、claude code、codex、kimi 原生支持，同一份文件四平台通用；插件清单按各平台私有约定分别提供。

| 平台 | 插件清单 | 技能发现路径 | 安装方式 |
| --- | --- | --- | --- |
| zcode | `.zcode-plugin/plugin.json` | 设置 → 插件 → 创建 → 添加插件市场，填仓库地址（仓库根含 `marketplace.json`） | 市场添加成功后在「个人」分段安装 `prompt-skills` 插件，即获得本仓库全部技能 |
| claude code | `.claude-plugin/plugin.json` | 插件安装后自动注册 | `/plugin marketplace add <本仓库路径>` 或直接安装 `.claude-plugin` 目录 |
| codex | `.codex-plugin/plugin.json` | 插件机制（v0.146+ 支持插件；技能亦可直接放入 `~/.codex/skills/`） | 按 Codex 插件安装指引添加本仓库 |
| kimi（Kimi Code CLI） | 无需清单（Skills 按标准路径发现） | `~/.kimi/skills/`、`~/.claude/skills/`、`~/.codex/skills/`、`~/.agents/skills/`、`~/.config/agents/skills/` 之一 | 将 `skills/loop-graph-designer/` 复制到任一发现路径；或用 `--skills-dir` 指向本仓库 `skills/` |

> 本仓库是「一个插件（prompt-skills）+ 一个市场（prompt）」形态：仓库根 `marketplace.json` 为 zcode 市场清单（`.claude-plugin/marketplace.json` 供 claude code 市场使用），市场条目以 `source: "./"` 指向仓库根（插件本体，`skills/` 内含全部技能）。以 git 仓库或本地目录方式添加市场时，客户端均先校验该市场清单。

> kimi 的 `plugin.json` 仅用于声明可执行工具；本技能为纯知识型（无需工具），因此不提供 kimi 工具插件清单，按技能方式安装即可。

### 未使用插件清单时的兜底安装（四平台通用）

```bash
# 任选其一（按平台惯例）
cp -r skills/loop-graph-designer ~/.zcode/skills/          # zcode 用户级
cp -r skills/loop-graph-designer ~/.claude/skills/         # claude code 用户级
cp -r skills/loop-graph-designer ~/.codex/skills/          # codex 用户级
cp -r skills/loop-graph-designer ~/.kimi/skills/           # kimi 用户级
```

## 使用方式

```text
在任意支持本项目技能的平台输入，例如：
「设计一个{任务}的交付 loop」/「按这份 spec 产出执行文档 /docs/prompt/...」/
「{需求描述}，要求任务精细拆分、明确中止条件，附参考文档 {路径} 与实现要求 {约束}」/
「/loop-graph-designer spec docs/specs/a.md docs/specs/b.md 技术栈 Spring Boot 3，强制 TDD」
「/loop-graph-designer input 给现有订单模块增加部分退款，要求不引入新表」
「/loop-graph-designer skill {需触发的技能列表} 按这份 spec 落地：{路径}」
「根据这个技术栈生成项目宪法 AGENTS.md：FastAPI + SQLAlchemy 2.0 + PostgreSQL + Redis Stream + LangGraph + Next.js」/
「生成项目宪法：根目录只做一层目录地图与索引，前后端规范分开——项目：{名称}，技术栈：{…}，参考设计文档 {路径}」/
「/constitution-generator agent 按这个技术栈生成项目宪法：{技术栈清单}」（默认，宪法体系 = AGENTS.md + 根 CLAUDE.md 唯一索引）/
「/constitution-generator claude 生成 CLAUDE.md 入口的宪法体系：{技术栈清单}」（宪法体系 = CLAUDE.md + 根 AGENTS.md 唯一索引）/
「保存进度，交接给新会话」/「生成交接文档，我要在新会话继续这个任务」
```

技能产出文档后，将该文档交给支持 subagent 派遣的编码工具或 agent 平台加载执行即可；文档规定调研、实现、审核均由 subagent 派遣完成，主控代理只做编排与门禁判定。

## 技能结构

```text
skills/loop-graph-designer/
├── SKILL.md                  # 子命令路由（spec/input/skill）、行为红线与内容来源约束、形态判定、文档设计流程
└── references/
    ├── template.md           # 生成文档的必含章节骨架与逐章填写规则
    └── example.md            # 书面风格示例（loop 形态 + graph 形态要点）

skills/constitution-generator/
├── SKILL.md                  # 子命令路由（claude/agent，默认 agent）、唯一根索引策略、工作流：输入收集、技术栈分域、subagent 调研派遣、章节合成、落盘
└── references/
    ├── template.md           # 定位层与子宪法必含章节骨架与逐章填写规则（含模式命名映射）
    └── example.md            # 书面风格示例（根定位层 + 子项目宪法 + 唯一根索引）

skills/session-handoff/
├── SKILL.md                  # 手动触发、触发即封存：新建交接文档 + 显式声明新会话执行 + 当前会话停止实现
└── references/
    └── handoff-template.md   # 交接文档必含章节骨架，顶部「交接声明」不可省略
```

## 新增技能（后续扩展）

1. 在 `skills/` 下新建目录 `skills/<新技能名>/SKILL.md`（frontmatter: `name` + `description`，name 与目录名一致）
2. 保持 `skills` 目录引用不变（三个插件清单均已声明 `"skills": "skills"`，自动包含新技能）
3. 在 README「当前技能」表格中登记

## 开发与校验

- 文件编码：UTF-8 无 BOM，LF 行尾
- 修改 SKILL.md 或 references 后，用 2–3 个真实测试 prompt 实测一轮（直接向支持该技能的平台提出），确认输出结构完整、门禁可验证后再定稿
