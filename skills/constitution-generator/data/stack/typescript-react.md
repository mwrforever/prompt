# typescript-react · 技术栈宪法条目

> **适用范围**：Node ≥24 · pnpm ≥12（corepack + packageManager 字段精确钉版）· React ≥19 · TypeScript strict（≥5，新工程从零开启）· Vite ≥8 · TanStack Router / TanStack Query 5+ · zustand · vitest · eslint（扁平配置）+ prettier · openapi-typescript + openapi-fetch
> **入库记录**：2026-10-24 维护者收录（内置路径），来源 img-create 项目 user-web（Tailwind 4 + shadcn 系）与 admin-web（antd 6 系）双宪法（含 2026-10-18～20 用户钉版条款）；共享核心 + 双 UI 栈分节
> **槽位覆盖**：A.1（含 UI 栈分节）/ A.2 / A.3（API 层）/ A.5（渲染性能）/ A.7 / C.4；A.4 无 DB 省略（请求 / 响应建模由 A.7 条款 2 承载）；A.6 无本项目额外约束（省略）。CI 构建配置段见 [ci/typescript.md](../ci/typescript.md)。

## A.1 编码约束（通用）

- 函数组件唯一，禁类组件；仅两处例外：`getSnapshotBeforeUpdate` 无函数等价物、Error Boundary 用 `react-error-boundary` 而非自写类
- Hooks 仅在函数组件 / 自定义 Hook 函数体顶层调用，禁在条件、循环、事件处理器、try/catch、`useMemo`/`useEffect` 回调内、条件 return 之后调用
- eslint-plugin-react-hooks 保持官方 recommended preset 整体启用（rules-of-hooks=error、exhaustive-deps=warn），禁手工删减规则、只增不删
- TypeScript 严格基线：`"strict": true` 必须开启（新工程从零开启，无存量例外）；禁显式 `any`——未知形状用 `unknown` + 类型收窄，已知形状用具体 interface / type；确需 any 处用局部行级 eslint-disable 并注明原因，禁整文件 / 整规则关闭
- 状态分层铁律：服务端状态一律 TanStack Query（缓存 / 去重 / 失效），客户端状态 zustand；禁手写请求 hooks 承担服务端态、禁把服务端数据快照复制进 zustand 长期存储
- zustand 订阅必须用 selector 且原子化：禁订阅整 store、禁 selector 内解构 / 组装新对象；组合多字段用 `useShallow`；store 定义统一 curried 写法 `create<T>()((set) => ({...}))`
- 路由用 TanStack Router（类型安全路由，文件式：`src/routes/` 目录 + @tanstack/router-plugin 生成 `src/routeTree.gen.ts`，生成物入库）；重依赖页路由级代码分割
- 动效克制原则：动画辅助理解状态变化优先于表演——「删掉后只是少了花哨」的动画不加
- 【全局铁律】一切组件按需加载、禁全量引入

### A.1.1 UI 栈分节 · Tailwind 4 + shadcn/ui 系

- 纯自设计路线：基础交互组件以 shadcn/ui 为起点（CLI 拉源码入仓，天然按需、可完全定制，底层 Radix + Tailwind）；shadcn 未覆盖的交互原语可按需引入 Radix 单包自建；shadcn 默认令牌必须改造对齐本端 `@theme` 令牌（单一事实源，禁双令牌体系）
- Tailwind CSS 4：设计令牌统一 `@theme` 定义（同时输出 CSS 变量）；禁散写硬编码色值 / 间距魔法数；与第三方样式共存冲突须显式处理，禁 `!` 前缀滥用构成事实穿透
- 动效组件 React Bits 优先（源码入仓、天然按需）、无内置实现时用 motion 重写（LazyMotion 按需）；motion 适用限于组件库覆盖不了的动效——布局动画（拖拽排序 / 列表增删平滑移位）、自定义组件进出场、数据变化过渡、侧边栏折叠等内容同步缩放、滚动与手势交互
- 图标一律优先图标库（如 lucide-react），禁手搓内联 SVG；品牌 logo 等确需定制者除外，须注释声明理由

### A.1.2 UI 栈分节 · antd 6 系

- antd 组件按需命名导入（默认 tree-shaking），禁引入全量样式文件
- 主题定制只走 ConfigProvider token 体系（seed → algorithm → map/alias → component token），禁直接对组件内部类名 / DOM 选择器写死 CSS 覆盖（确需覆盖第三方样式须白名单 + 注明原因）
- antd v6 废弃 API 禁新增使用（`bordered`→`variant`、子组件方式→`items`、`destroyOnClose`→`destroyOnHidden` 等，见官方 v5→v6 迁移清单）
- 静态方法（`message.xxx` / `Modal.xxx`）不读 ConfigProvider context：消息 / 弹窗统一经 `App` 组件包裹 + `useMessage` / `useModal` + contextHolder，封装收敛到组件层
- `theme` prop 禁在 `undefined` 与对象间切换（致组件重新 mount），用空对象 `{}` 代替
- 布局自建：antd Layout / Menu + ConfigProvider token，零额外布局依赖；表格表单用 antd Table + antd Form 原生方案，禁引入 ProComponents 预发布线（稳定发布后再评估）
- motion 禁与 antd 重复造轮子：Modal / Drawer / Menu 等过渡一律用其内建 motion / transitionName 配置，禁用 motion 重新包裹；两者禁对同一元素叠加双动画
- 样式双轨分工：antd 组件视觉一律走 ConfigProvider token 唯一出口；Tailwind 只承担页面布局、间距微调与非组件区域自定义样式，禁作用于 antd 组件内部类名、禁 `!` 前缀穿透；preflight 必须关闭（防重置样式污染）；视觉令牌单一事实源 = antd token（可配 cssVar 输出 CSS 变量）供 `@theme` 引用消费，禁另立第二套色板 / 间距体系；运行时 CSS-in-JS 库禁入
- 表单双轨：常规筛选表单 antd Form；复杂长表单 react-hook-form + zod，antd 输入组件经 Controller 适配层收敛于通用组件层

## A.2 配置管理

- 自定义环境变量必须 `VITE_` 前缀（envPrefix 默认不改）；无前缀变量不暴露给客户端代码
- 禁在 env 存放敏感信息：`VITE_` 变量构建时静态替换打包进产物，token / 密钥只经后端会话
- 环境判断用 `import.meta.env`（MODE / PROD / DEV），禁用 `process.env`；环境判定用 MODE（与 NODE_ENV 区分）
- 新增 env 变量必须同步在 `src/vite-env.d.ts` 增强 `ImportMetaEnv`（该文件禁 import 语句，否则增强失效）
- `.env.local` / `.env.[mode].local` 默认不入 git；提交 example 模板不提交 local 真实值；`.env` 修改后须重启 dev server
- API 地址分层钉死：默认相对路径走同源反代（dev = dev server proxy、prod = nginx），直连场景用 `VITE_API_BASE` 显式注入——同源反代为默认、env 覆盖为例外

## A.3 API 层设计

- API 类型唯一来源 = OpenAPI 生成副本（openapi-typescript + openapi-fetch，`gen:api` 脚本再生成）：禁手写后端契约镜像类型、禁手改生成产物；spec 更新后必须先 `gen:api` 再开发
- HTTP 调用统一经 `src/api/` 客户端封装，禁页面 / 组件裸发 fetch / 裸建 EventSource
- 多前缀域分治：各后端前缀域（不同错误契约）各自统一客户端封装并塑形统一错误对象，业务层禁各自解析原始 error
- SSE 事件流访问必须经统一 hook 封装（连接生命周期与断线续传收敛于此），禁页面裸建 EventSource

## A.5 性能基础设施（前端渲染性能）

- 渲染期间的重计算优先并发 API：`useDeferredValue`（必须配 `memo` 才生效；禁给受控输入本身用）、`useTransition`（禁对受控输入用）；非渲染工作才用 debounce / throttle，统一自实现 util + 单测
- 长列表达到虚拟化阈值（候选 >200 行 DOM）必须虚拟化；小列表（<100 项）禁引入
- 图片懒加载：离屏图统一 `loading="lazy"`（可加 `decoding="async"`），在网格 / 通用组件统一封装、业务侧禁散写；首屏 / LCP 图禁懒加载（用 `fetchpriority="high"`）
- 动效性能铁律（属性级约束，与动效工具无关——原生 CSS、组件库内建动画、motion 一律适用）：逐帧动画只允许合成属性 transform / opacity；布局类动效（尺寸 / 位置变化）为唯一例外通道，一律经 FLIP 机制（motion layout，内部 transform 校正）承载；禁逐帧动画 width / height / top / left 等重排属性
- 重依赖页必须路由级代码分割；组件库源码随用随引，禁全量拷入

## A.7 跨层数据对象与传参约束（本栈习语）

1. **参数对象化**：对象建模 interface 优先（需联合 / 元组 / 条件类型等 type 特性时才用 type）；组件参数 > 3 个或含可选项必须收拢为单个 typed props 对象（React 组件签名即对象参数），回调 prop 用 `onXxxChange` 命名保持逆向数据流显式；仅业务确需灵活传参可例外。
2. **返回业务对象**：`src/api/` 每个函数必须返回 OpenAPI 生成的具体 typed 类型，禁返回裸响应对象或 any 散给调用方；请求与响应类型分别引用生成物，禁同一类型双向复用。
3. **职责隔离**：API 生成类型（后端契约镜像）≠ store state（客户端态 + actions）≠ 组件 props（视图定制参数），字段全同也按职责各自建模；依赖方向单向（契约可被里层引用，里层类型禁外溢成 API 参数）；派生数据计算而非存储；跨层数据对象为纯数据（可序列化），行为放 actions / 回调 props，禁携带函数 / 闭包。
4. **例外边界**：外部包暂无类型 / 迁移过渡期允许局部 any + 行级 eslint-disable（注明原因与移除计划）；类型收窄代替 `as` 断言（断言仅限类型守卫函数 / 确知的边界转换处）；无理由的违反视为缺陷，不得合入。

## C.4 常用命令骨架

```bash
pnpm install                     # 安装 / 同步依赖（按 pnpm-lock.yaml，锁文件入库）
pnpm dev                         # 启动 dev server（proxy 指向各后端）
pnpm build                       # 构建（类型检查先行 + vite build）
pnpm lint                        # eslint 全量检查（CI 快层加 --max-warnings 0）
pnpm exec eslint . --fix         # lint 自动修复（仅安全修复项）
pnpm test                        # vitest 全量测试
pnpm exec vitest run tests/x.test.ts  # 单文件测试
pnpm gen:api                     # 按 OpenAPI spec 再生成类型副本（spec 更新后必跑）
pnpm exec prettier --check .     # 格式化校验（--write 修复）
```

## 来源

- img-create 项目 `user-web/AGENTS.md`、`admin-web/AGENTS.md`（Part A / C.4 / C.6，含用户钉版条款）及其 C.2 版本表实测记录
- 关键官方文档：React 19（并发 API）、TanStack Router / Query、Tailwind CSS 4（@theme）、Ant Design v5→v6 迁移清单与 ConfigProvider token、zustand（curried 写法）、openapi-typescript
