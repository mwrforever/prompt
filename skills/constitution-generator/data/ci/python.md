# python · CI 构建配置段条目

> **适用语言与版本范围**：CPython ≥3.13；工具链前提：uv（依赖管理，uv.lock 入库）+ ruff（lint + format）+ pytest（测试）+ 可选 FastAPI（OpenAPI spec 提交物）+ pytest marker 分层（单测 / integration）
> **入库记录**：2026-10-24 维护者收录（内置路径），来源 img-create 项目 backend-ai 宪法 C.5 与 `scripts/ci.sh` 实跑沉淀
> **CI 形态**：按「本地双层 CI」沉淀——快层随 git commit 由 pre-commit 触发（仅暂存文件），慢层随 git push 由 pre-push 委托脚本门禁，合入主干前另设全量层；云端流水线未收录，阶段与门禁链可直接映射为云端 job，需平台细则时按未命中调研。

## 快层（pre-commit · 仅暂存文件）

- 通用文本卫生钩子（pre-commit-hooks 官方集）：trailing-whitespace / end-of-file-fixer / check-yaml / check-json / mixed-line-ending `--fix=lf` / detect-private-key / no-commit-to-branch（豁免纪律见 ci/java.md 快层节）
- ruff check + ruff format --check，仅对暂存 `.py` 文件；uv 经 winget 等安装时 PATH 可能不在旧 shell 内，hook 入口显式注入 PATH 兜底

```yaml
- id: ai-ruff-check
  name: python ruff check（暂存文件）
  entry: bash -c 'export PATH="<uv 链接目录>:$PATH"; cd <子项目> && f=(); for p in "$@"; do f+=("${p#<子项目>/}"); done; uv run ruff check --force-exclude "${f[@]}"'
  language: system
  files: ^<子项目>/.*\.py$
  pass_filenames: true
```

## 慢层（push 门禁 · 全量）

- 阶段链：`uv sync`（按 uv.lock 同步，兼验锁文件）→ `uv run ruff check .` → `uv run ruff format --check .` → `uv run pytest -m "not integration"`（单测层）→ OpenAPI spec 防漂移（spec 再导出 + `git diff --exit-code`，见 ci/java.md 防漂移节）
- **pytest 退出码 5（无收集用例）必须显式视为失败**：测试目录为空或 marker 过滤后无用例时 pytest 以 5 退出，门禁不得放行空跑
- 任一阶段非零退出即阻断合入；`--no-verify` 绕过视为违规

## 合入主干层（全量 + 集成）

- gate 全量 + 集成层：`uv run pytest tests/integration/`（连真实本地依赖：数据库 / 中间件；前置基础设施就绪脚本，冷机场景先起基础设施再跑）
- 集成层与单测层用 pytest marker 分层（`-m "not integration"` 进 push 层、integration 目录进合干层），控制 push 时长

## 缓存与加速

- uv 全局缓存 + ruff 内建缓存 + `.pytest_cache`；测试分层（单测进 push、集成进合干）控制时长；全量 pytest 时长实测后校正分层

## 来源

- img-create 项目 `backend-ai/AGENTS.md` C.5、`.pre-commit-config.yaml`（ai-ruff-check / ai-ruff-format）、`scripts/ci.sh`（ci_ai / ci_all 实跑沉淀，含退出码 5 失败化与集成分层）
