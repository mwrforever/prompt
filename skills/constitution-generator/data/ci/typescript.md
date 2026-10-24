# typescript · CI 构建配置段条目

> **适用语言与版本范围**：Node ≥24（engines + engine-strict 双件套为安装期硬门禁）· pnpm（corepack + packageManager 字段精确钉版）；工具链前提：eslint（扁平配置）+ prettier + vitest + Vite / tsc 构建 + 可选 openapi-typescript（gen:api 类型副本）
> **入库记录**：2026-10-24 维护者收录（内置路径），来源 img-create 项目 user-web / admin-web 双宪法 C.5 与 `scripts/ci.sh` 实跑沉淀
> **CI 形态**：按「本地双层 CI」沉淀——快层随 git commit 由 pre-commit 触发（仅暂存文件），慢层随 git push 由 pre-push 委托脚本门禁；云端流水线未收录，阶段与门禁链可直接映射为云端 job，需平台细则时按未命中调研。

## 快层（pre-commit · 仅暂存文件）

- 通用文本卫生钩子（pre-commit-hooks 官方集）：trailing-whitespace / end-of-file-fixer / check-yaml（pnpm-lock.yaml 多文档生成物豁免）/ check-json（tsconfig*.json 为 JSONC 豁免严格解析）/ mixed-line-ending `--fix=lf` / detect-private-key / no-commit-to-branch
- prettier --check + eslint --no-warn-ignored --max-warnings 0，仅对暂存文件（`pnpm -C <子项目>` 免 cd 路径换算）

```yaml
- id: web-eslint
  name: eslint（暂存文件）
  entry: bash -c 'f=(); for p in "$@"; do f+=("${p#<子项目>/}"); done; pnpm -C <子项目> exec eslint --no-warn-ignored --max-warnings 0 "${f[@]}"'
  language: system
  files: ^<子项目>/.*\.(ts|tsx)$
  pass_filenames: true
```

## 慢层（push 门禁 · 全量）

- 阶段链（任一失败非零退出即阻断合入）：
  1. `pnpm install --frozen-lockfile` —— 依赖一致性门禁：依赖与锁文件不一致即失败（禁 npm / yarn 平行锁文件）
  2. `pnpm lint --max-warnings 0` —— eslint，0 error 0 warning
  3. `pnpm exec prettier --check .` —— 格式化校验
  4. `pnpm test` —— vitest run 全量
  5. `pnpm build` —— 类型检查先行 + vite build
  6. OpenAPI 防漂移（gen:api 项目适用）：`pnpm gen:api` 再生成 + `git diff --exit-code -- src/api/schema`，spec 更新后未重跑 gen:api 即失败（见 ci/java.md 防漂移节）
- `--no-verify` 绕过视为违规；main 分支受 no-commit-to-branch 保护

## 生成物与锁文件纪律

- 路由生成物（routeTree.gen.ts）与 API 类型副本等生成物**入库**并纳入 diff 校验；pnpm-lock.yaml 入库、禁手改（经 `pnpm install` / `pnpm add` 变更）；registry 镜像为机器级 `~/.npmrc` 配置不入库

## 缓存与加速

- pnpm store 缓存 + pnpm-lock.yaml 锁定 + vitest 缓存；快层仅暂存文件、慢层全量的双层分工

## 来源

- img-create 项目 `user-web/AGENTS.md` / `admin-web/AGENTS.md` C.5、`.pre-commit-config.yaml`（web prettier / eslint 钩子）、`scripts/ci.sh`（ci_web 实跑沉淀，含 --frozen-lockfile 依赖一致性与 gen:api 防漂移）
