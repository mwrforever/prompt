# 本地宪法库（data/ 目录）

> constitution-generator 的内置技术宪法库，双目录布局：**技术栈宪法**存 `data/stack/{技术栈}.md`、**CI 构建配置段**存 `data/ci/{语言}.md`。生成宪法时**查库优先**——命中直接复用不派遣、部分命中只对缺失槽位派遣补齐、全未命中才整主题派遣调研；新调研片段经用户**明确表示达标**后回写补库（唯一入库路径）。查库与回写规范正文见 SKILL.md §五。

## 目录与命名

- `data/stack/{技术栈}.md` —— 技术栈宪法条目：小写 kebab-case，主语言 / 运行时在前、框架在后（如 `python-fastapi.md`、`typescript-nextjs.md`、`go.md`）；独立基础设施主题（缓存 / 消息队列等跨栈组件）单独成文件并映射其宪法槽位（如 `redis-stream.md` → A.5）。
- `data/ci/{语言}.md` —— CI 构建配置段条目：按语言唯一命名（如 `python.md`、`typescript.md`、`go.md`、`java.md`）；同语言多框架差异以条目内小节区分（如「构建（Next.js）」「构建（Vue）」），多 CI 平台分段（GitHub Actions / GitLab CI / Jenkins…）。
- 库文件以本技能基准目录为根解析相对路径；查库时目录或文件不存在即按未命中处理。

## 条目格式

**stack 条目**（技术栈宪法）：

1. 文件头：适用范围（组件 + 版本下限，项目实测版本低于下限即视为未命中）+ 入库记录（日期与来源：用户达标确认回写，附调研报告路径）+ 槽位覆盖清单。
2. 槽位条款：按宪法框架槽位分节（A.1–A.6 / A.7 本栈习语 / C.3 目录职责骨架 / C.4 常用命令），原则级条款（禁令 / 必须 / 边界），零功能实现描述；不适用槽位写省略说明（与 template 省略规则一致），A.8 四段律为体系内置条款不在库条目范围。
3. 文末来源：关键官方文档 URL，真实可查。

**ci 条目**（CI 构建配置段）：

1. 文件头：适用语言与版本范围 + 工具链前提（包管理器 / lint / 测试框架）+ 入库记录。
2. 平台分段：每 CI 平台一节，含触发方式、硬门禁链（lint → format 校验 → 测试 → 构建）、缓存策略与关键 yaml 段（合成时按项目实际裁剪）。

## 条目索引

条目随技能使用增长：用户达标确认后回写新增（SKILL.md §六.9）、维护者可内置收录（本仓库行为），本表由入库动作同步维护。首批条目 2026-10-24 自 img-create 项目宪法收录（含 2026-10-18～20 多轮用户钉版条款，项目专属内容已剥离）。

| 类型 | 文件 | 适用范围 | 覆盖槽位 |
|---|---|---|---|
| stack | [java-springboot.md](stack/java-springboot.md) | Java ≥21 · Spring Boot ≥4.1 · MyBatis-Plus ≥3.5.17 · PostgreSQL ≥15 · Maven ≥3.9（Boot 3.x 未覆盖） | A.1–A.5 / A.7 / C.3 / C.4（A.6 引全局不入库） |
| stack | [python-fastapi.md](stack/python-fastapi.md) | CPython ≥3.13 · FastAPI ≥0.140 · SQLAlchemy 2.x async + asyncpg · redis-py ≥8 · uv · ruff · pytest | A.1–A.5 / A.7 / C.3 / C.4 |
| stack | [typescript-react.md](stack/typescript-react.md) | Node ≥24 · pnpm ≥12 · React ≥19 · TS strict · Vite ≥8 · TanStack Router/Query · vitest（UI 分节：Tailwind 4 + shadcn / antd 6） | A.1–A.3 / A.5 / A.7 / C.4（A.4 无 DB 省略） |
| ci | [java.md](ci/java.md) | Maven/mvnw · Spotless · Surefire · springdoc spec 防漂移 · Flyway 号段校验 | 快层 / 慢层 / 合干层（本地双层 CI） |
| ci | [python.md](ci/python.md) | uv · ruff · pytest marker 分层 · FastAPI spec 防漂移 | 同上 |
| ci | [typescript.md](ci/typescript.md) | pnpm · eslint · prettier · vitest · gen:api 防漂移 | 同上 |

## 维护纪律

- 唯一入库路径 = 用户明确表示达标后的回写（SKILL.md §五 / §六.9）；回写内容为**通用条款**（栈级 / 语言级），须剥离项目专属细节（目录名、业务模型、端口、项目名），保持条目可跨项目复用。
- 技能包发版时可由维护者直接内置新条目或修订错漏（本仓库行为），修订须保持条款与来源一一对应。
- 插件缓存目录可写但会被插件更新覆盖：运行期回写落进缓存后，须把条目文件同步回技能源仓库再发版，否则更新即丢失。
