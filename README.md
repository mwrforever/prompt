# prompt-skills · 提示词工程技能包

三个生产规范化文档生成技能，均**仅限用户显式斜杠命令触发**（自然语言需求描述不激活）；文档规定调研、实现、审核均由 subagent 派遣完成，主控代理只做编排与门禁判定。

## 技能一览

| 技能 | 功能 | 子命令 | 输出 |
| --- | --- | --- | --- |
| `loop-graph-designer`（交付 Loop/Graph 设计器） | 基于需求（input）或已定 spec 产出生产规范化执行文档：任务拆分、阶段门禁与中止条件、subagent 执行制、Definition of Done；按任务并行性选 loop（序贯阶段）/ graph（并行 DAG）形态。skill 子命令把协同技能编排进执行文档对应阶段，example 子命令按内置场景生成专项文档（当前内置 risk：项目风险扫描） | `/loop-graph-designer [spec\|input\|skill\|example]`，默认 input | `/docs/prompt/{yyyy-MM-dd}-{loop\|graph}-{任务标题}.md` |
| `constitution-generator`（项目宪法生成器） | 按项目技术栈生成分层宪法体系：根定位层（一层目录地图 + 子项目宪法索引）+ 各技术子项目宪法（只存工程原则与约束、零功能实现描述）+ 根索引，AGENTS.md / CLAUDE.md 双命名同时产出。内置本地宪法库（`data/stack/`、`data/ci/`）查库优先，未命中的技术栈才派 subagent 调研官方文档与厂商最佳实践，新片段经用户确认达标后回写补库 | `/constitution-generator [claude\|agent]`，默认 agent | `<项目根>/AGENTS.md` + `<子项目>/AGENTS.md` ×N + 根索引 `CLAUDE.md`（claude 模式命名对调）；另含 `docs/agmds-research/` 调研报告 |
| `session-handoff`（会话交接器） | 把当前会话任务固化成进度交接文档，显式声明「未完成部分须在新会话执行」与续接加载纪律；生成后当前会话立即封存、停止一切任务实现。update 原地更新本会话引入的进度文件 | `/session-handoff [update\|rewrite]`，默认 rewrite | `<项目根>/docs/progress/{yyyy-MM-dd}-{精简标题}.md` |

各技能的行为红线、完整流程与章节模板见其 `SKILL.md` 与 `references/`，README 不复述。

## 兼容性与安装

技能本体采用 [agentskills.io](https://agentskills.io) 开放标准（目录 + `SKILL.md`），zcode、claude code、codex、kimi 原生支持，同一份文件四平台通用。

| 平台 | 安装方式 |
| --- | --- |
| zcode | 设置 → 插件 → 创建 → 添加插件市场（仓库根含 `marketplace.json`），添加成功后在「个人」分段安装 `prompt-skills` 插件 |
| claude code | `/plugin marketplace add <本仓库路径>` 或直接安装 `.claude-plugin` 目录 |
| codex | 按 Codex 插件安装指引添加本仓库（v0.146+），或将技能复制进 `~/.codex/skills/` |
| kimi | 将 `skills/<技能名>/` 复制到 `~/.kimi/skills/` 等任一发现路径，或用 `--skills-dir` 指向本仓库 `skills/` |

> 本仓库是「一个插件（prompt-skills）+ 一个市场（prompt）」形态：根 `marketplace.json` 为 zcode 市场清单，`.claude-plugin/marketplace.json` 供 claude code 使用，市场条目以 `source: "./"` 指向仓库根。kimi 无需工具插件清单（本技能包为纯知识型）。

不用插件清单时，直接复制技能目录到各平台用户级路径即可：

```bash
cp -r skills/loop-graph-designer ~/.zcode/skills/    # zcode；claude code / codex / kimi 换对应目标路径
```

## 使用示例

```text
「/loop-graph-designer input 设计一个{任务}的交付 loop，要求任务精细拆分、明确中止条件」
「/loop-graph-designer spec docs/specs/a.md 技术栈 Spring Boot 3，强制 TDD」
「/loop-graph-designer example risk 对当前仓库做一次项目风险扫描，只读出报告不修复」
「/constitution-generator agent 按这个技术栈生成项目宪法：{技术栈清单}」
「/constitution-generator claude 生成 CLAUDE.md 入口的宪法体系：{技术栈清单}」
「/session-handoff 保存进度，交接给新会话」
```

技能产出文档后，交给支持 subagent 派遣的编码工具或 agent 平台加载执行即可。

## 技能结构

```text
skills/loop-graph-designer/
├── SKILL.md                  # 子命令路由、行为红线与内容来源约束、形态判定、文档设计流程
├── examples/                 # example 子命令场景专项示例（一场景一文件，当前内置 risk）
└── references/
    ├── template.md           # 生成文档的必含章节骨架与逐章填写规则
    └── example.md            # 书面风格示例

skills/constitution-generator/
├── SKILL.md                  # 子命令路由、宪法基础框架槽位、本地宪法库查库优先与达标回写、工作流
├── data/                     # 本地宪法库（查库优先，未命中才派遣；条目经用户达标确认回写增长）
│   ├── README.md             # 库布局（stack/ + ci/ 双目录）、条目格式与维护纪律、条目索引
│   ├── stack/                # 技术栈宪法条目 {技术栈}.md（目录随首个条目回写创建）
│   └── ci/                   # CI 构建配置段条目 {语言}.md（目录随首个条目回写创建）
└── references/
    ├── template.md           # 定位层与子宪法必含章节骨架与逐章填写规则（含模式命名映射）
    └── example.md            # 书面风格示例（根定位层 + 子项目宪法 + 根索引）

skills/session-handoff/
├── SKILL.md                  # 触发即封存：新建交接文档 + 新会话执行与续接加载纪律
└── references/
    └── handoff-template.md   # 交接文档必含章节骨架，顶部「交接声明」不可省略
```

## 新增技能（后续扩展）

1. 在 `skills/` 下新建目录 `skills/<新技能名>/SKILL.md`（frontmatter: `name` + `description`，name 与目录名一致）
2. 三个插件清单均已声明 `"skills": "./skills"`，新技能自动包含
3. 在 README「技能一览」表格登记

## 开发与校验

- 文件编码：UTF-8 无 BOM，LF 行尾
- 修改 SKILL.md 或 references 后，用 2–3 个真实测试 prompt 实测一轮，确认输出结构完整、门禁可验证后再定稿
