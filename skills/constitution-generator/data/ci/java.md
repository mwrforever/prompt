# java · CI 构建配置段条目

> **适用语言与版本范围**：Java ≥21 · Maven ≥3.9（经 mvnw wrapper）；工具链前提：Spotless（格式化，palantir-java-format 等）+ Surefire / JUnit5（测试）+ 可选 springdoc（OpenAPI spec 提交物）+ 可选 Flyway（迁移）+ 可选 Testcontainers（PG 集成测试）
> **入库记录**：2026-10-24 维护者收录（内置路径），来源 img-create 项目 backend-java 宪法 C.5 与 `scripts/ci.sh` 实跑沉淀；同日 TallyHo 项目用户达标确认回写补入合干层 Testcontainers 探测条目
> **CI 形态**：按「本地双层 CI」沉淀——快层随 git commit 由 pre-commit 触发（仅暂存文件、秒级），慢层随 git push 由 pre-push 委托脚本门禁，合入主干前另设全量层；云端流水线（GitHub Actions / GitLab CI 等）未收录，本条目阶段与门禁链可直接映射为云端 job，需平台细则时按未命中调研。

## 快层（pre-commit · 仅暂存文件）

- 通用文本卫生钩子（pre-commit-hooks 官方集）：trailing-whitespace（尾随空白）、end-of-file-fixer（末尾换行）、check-yaml、check-json、mixed-line-ending `--fix=lf`（行尾统一）、detect-private-key（私钥泄露检测）、no-commit-to-branch（主干分支禁直推）
- 豁免纪律：包管理器生成物（如 pnpm-lock.yaml 多文档）豁免语法校验；JSONC 文件（tsconfig*.json）豁免严格 JSON 解析；必须保留 CRLF 的 Windows 批处理文件豁免行尾归一钩子（入库经 autocrlf 归一）
- Spotless 校验：暂存 `.java` 时触发 `spotless:check`（增量缓存约 1s）。**Windows 陷阱**：`spotlessFiles` 传文件名按正则匹配反斜杠绝对路径、posix 相对路径永不命中，会静默失效——快层应全量 check + `pass_filenames: false`（2026-10-20 实测修复）
- mvnw 命令前必须 `export JAVA_HOME=<钉版 JDK 路径>`（系统默认版本不符为常见陷阱）

```yaml
- id: java-spotless
  name: java spotless（全量校验）
  entry: bash -c 'export JAVA_HOME=/path/to/jdk && cd <子项目> && ./mvnw -q spotless:check'
  language: system
  files: ^<子项目>/.*\.java$
  pass_filenames: false
```

## 慢层（push 门禁 · 全量）

- `./mvnw clean verify` = spotless:check（绑定 verify 阶段）→ test（Surefire 全量）→ package，任一失败非零退出即阻断合入；`--no-verify` 绕过视为违规
- OpenAPI 防漂移门禁（springdoc 项目适用）：spec 再导出脚本 + `git diff --exit-code` 校验提交物；漂移时再生成产物留在工作区作为证据（diff 即漂移内容），修复后重新提交或还原。注意：spec 再导出须起真实应用——依赖本地数据库 / 中间件在位，缺位时前置探测报错并给出明确指引（而非让导出阶段报晦涩错误）

```bash
bash scripts/export-openapi.sh                  # 再导出 spec 提交物
git diff --exit-code -- docs/openapi/api.yaml   # 漂移即失败，工作区产物即证据
```

- 迁移校验（Flyway 项目适用）：脚本校验迁移文件命名（`V{n}__{snake_case}.sql`）、版本号唯一性（重复即失败）、业务域号段归属；零迁移期空过 PASS

```bash
ls db/migration | sed -n 's/^V\([0-9]\+\)__.*/\1/p' | sort | uniq -d   # 版本重复检测
```

## 合入主干层（全量 + 集成）

- 前置：基础设施就绪（本地数据库 / 中间件，幂等起停脚本）；Testcontainers 集成测试纳入本层时，起容器前先探测 Docker 可用性，缺失明确报错指引（禁晦涩失败）
- gate 全量 + 启动冒烟：起应用 → `/health` 探活 → 停。注意硬停后租约类资源（如机器号注册 TTL 键）靠过期归还，紧随其后的重启会撞号 fail-fast——冒烟前轮询等待租约过期再起（设上限超时放行走原失败路径留痕）
- 退出码即门禁：任一阶段非零退出即阻断；分阶段横幅计数便于定位失败阶段

```bash
set -euo pipefail
STAGE_NO=0
stage() { STAGE_NO=$((STAGE_NO+1)); echo "======== [${STAGE_NO}] $1 ========"; }
fail() { echo "[CI 失败] $1（阶段 ${STAGE_NO}）" >&2; exit 1; }
```

## 缓存与加速

- `~/.m2` 本地仓库复用；Spotless up-to-date 增量 + ratchetFrom 限定新增代码；快层仅暂存文件、慢层全量的双层分工控制 commit 耗时

## 来源

- img-create 项目 `backend-java/AGENTS.md` C.5、`.pre-commit-config.yaml`、`scripts/ci.sh`（ci_java / ci_openapi_drift / ci_flyway_check / ci_all 实跑沉淀，含 Windows spotlessFiles 陷阱与 machineId 租约竞态修复记录）
