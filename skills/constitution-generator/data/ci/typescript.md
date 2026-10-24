# typescript · CI 构建配置段条目

> **适用语言与版本范围**：Node ≥24（engines + engine-strict 双件套为安装期硬门禁）· pnpm（corepack + packageManager 字段精确钉版；「微信原生小程序」小节例外用 npm）；工具链前提：eslint（扁平配置）+ prettier + vitest + Vite / tsc 构建 + 可选 openapi-typescript（gen:api 类型副本）
> **入库记录**：2026-10-24 维护者收录（内置路径），来源 img-create 项目 user-web / admin-web 双宪法 C.5 与 `scripts/ci.sh` 实跑沉淀；同日 TallyHo 项目用户达标确认回写补入「微信原生小程序（npm）」小节（剥离项目专属细节）
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

## 微信原生小程序（npm）

> 适用于「原生小程序 + TypeScript」栈（配套 stack 条目 typescript-miniprogram.md）；包管理器钉死 npm（微信「构建 npm」按标准 npm 布局做依赖分析，pnpm 符号链接布局会导致 miniprogram_npm 空产物，官方未承诺兼容）。

- 快层：pre-commit 通用文本卫生钩子（tsconfig*.json 为 JSONC 豁免严格解析）+ prettier --check + eslint（`--no-warn-ignored --max-warnings 0`，仅暂存 .ts 文件）
- 慢层阶段链（任一失败非零退出即阻断合入）：
  1. `npm ci` —— 依赖一致性门禁（package-lock.yaml 锁定，禁 npm/yarn/pnpm 平行锁文件）
  2. `npx tsc --noEmit` —— **独立类型检查门禁（栈级关键）**：微信开发者工具 TS 编译基于 Babel 仅移除类型不报错，不能依赖工具把关
  3. `npm run lint --max-warnings 0` —— eslint 全量
  4. 敏感词扫描（如项目需要）—— 自写 Node 脚本或 ESLint 自定义规则，禁词表配置驱动，命中输出 文件:行号 exit 1；图片资源内文字走流程管控不入静态扫描
  5. `npm run build:npm`（miniprogram-ci packNpm）并校验 miniprogram_npm 非空 —— 防依赖布局异常导致空产物
- 合入主干层：慢层全量 + `npm run preview`（miniprogram-ci 预览出码，校验可构建包）+ 图片资源文字人工复核登记
- 上传发布（发版流程，非日常门禁）：`npm run upload` 仅显式触发，robot / version / desc 显式传参；miniprogram-ci 私钥禁入库（环境变量注入）+ mp 后台 IP 白名单；提审前跑微信官方「小程序安全检测」兜底

## 来源

- img-create 项目 `user-web/AGENTS.md` / `admin-web/AGENTS.md` C.5、`.pre-commit-config.yaml`（web prettier / eslint 钩子）、`scripts/ci.sh`（ci_web 实跑沉淀，含 --frozen-lockfile 依赖一致性与 gen:api 防漂移）
