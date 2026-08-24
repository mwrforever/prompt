# prompt-skills · 提示词工程技能包

生产规范化文档的生成技能集，三类产出：

- **交付执行文档**：基于用户需求或已定项 spec，产出带阶段门禁、循环/中止机制、DoD 目标设定的执行文档
- **项目宪法**：基于项目技术栈，派发 subagent 调研各厂商最佳实践，产出只存原则、约束与数据契约的 AGENTS.md
- **任务交接文档**：手动触发，把当前会话任务固化成交接文档并显式声明「未完成部分须在新会话执行」，生成后当前会话封存、不再继续实现

## 当前技能

| 技能 | 用途 | 输出 |
| --- | --- | --- |
| `loop-graph-designer`（交付 Loop/Graph 设计器） | 把需求或多份 spec 转化为生产级交付执行文档：每份 spec = 一个任务、需求按复杂性模块化拆分（不过度拆分）、loop/graph 编排、阶段门禁与中止条件、subagent 执行制、Definition of Done | `/docs/prompt/{yyyy-MM-dd}-{loop|graph}-{任务标题}.md` |
| `constitution-generator`（项目宪法生成器） | 根据项目技术栈生成项目宪法 AGENTS.md：按技术栈组件族划分调研主题，派发 subagent 调研各技术栈官方文档与厂商最佳实践，提炼为「只存原则、约束与数据契约」的四段结构宪法（A 后端通用 / B 架构层 / C 前端 / D 项目实际） | `<目标项目根>/AGENTS.md` + `/docs/agmds-research/{yyyy-MM-dd}-{主题}.md` 调研报告 |
| `session-handoff`（会话交接器） | 仅限手动触发（「保存进度 / 交接 / 生成交接文档 / 续接准备」等）。在项目 docs/progress/ 新建任务交接文档，顶部显式声明未完成任务须在新会话执行；生成并回报后当前会话立即封存，停止一切任务实现，后续续做请求一律引导至新会话。每次触发必新建文档（同名追加 -2、-3 序号），不覆盖历史 | `<目标项目根>/docs/progress/{yyyy-MM-dd}-{精简标题}.md` |

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
「根据这个技术栈生成项目宪法 AGENTS.md：FastAPI + SQLAlchemy 2.0 + PostgreSQL + Redis Stream + LangGraph + Next.js」/
「生成一份类似 MindSoar 那种的宪法，项目：{名称}，技术栈：{…}，参考设计文档 {路径}」/
「保存进度，交接给新会话」/「生成交接文档，我要在新会话继续这个任务」
```

技能产出文档后，将该文档交给支持 subagent 派遣的编码工具或 agent 平台加载执行即可；文档规定调研、实现、审核均由 subagent 派遣完成，主控代理只做编排与门禁判定。

## 技能结构

```text
skills/loop-graph-designer/
├── SKILL.md                  # 工作流：输入收集、形态判定、8 步设计流程、产出要求
└── references/
    ├── template.md           # 生成文档的必含章节骨架与逐章填写规则
    └── example.md            # 书面风格示例（loop 形态 + graph 形态要点）

skills/constitution-generator/
├── SKILL.md                  # 工作流：输入收集、技术栈分域、subagent 调研派遣、章节合成、落盘
└── references/
    ├── template.md           # AGENTS.md 必含章节骨架与逐章填写规则
    └── example.md            # 书面风格示例（源自 MindSoar 项目宪法）

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