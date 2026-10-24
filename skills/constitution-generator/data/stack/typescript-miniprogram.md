# typescript-miniprogram · 技术栈宪法条目

> **适用范围**：微信原生小程序（非 Taro / uni-app 抽象层）· TypeScript（strict）· TDesign 小程序版（≥2.12.0 线）· echarts-for-weixin（canvas 2d）· npm 包管理（pnpm 布局约束见 C.4）· 基础库下限按依赖综合取值（隐私接口 ≥2.32.3、canvas 2d ≥2.9.0，取最高者）
> **入库记录**：2026-10-24 用户达标确认回写，来源 TallyHo 项目 miniprogram 宪法（含审核修订）与调研报告（微信官方文档 20+ 篇 / GitHub 官方仓库 4 个核验）
> **槽位覆盖**：A.1 / A.2 / A.3 / A.5 / A.7 / C.3 / C.4 + 附加分节（隐私授权 / 广告组件 / 敏感词扫描）；A.4 无 DB 省略、A.6 引用全局注释 / 日志 / 测试规范不入库
> **CI 构建配置段**：见 [ci/typescript.md](../ci/typescript.md)「微信原生小程序（npm）」小节

## A.1 编码约束

- 工程以官方 TypeScript 模板为基线（`setting.useCompilerPlugins: ["typescript"]`，可与 less/sass 组合）；**CI 必须独立执行 `tsc --noEmit` 做类型检查**——开发者工具的 TS 编译基于 Babel 仅移除类型、不报类型错误，不得依赖工具把关
- 微信 API 类型统一 `miniprogram-api-typings`（tsconfig `compilerOptions.types` 指定），禁散装 typings；tsconfig 开启 strict 系列开关
- 禁止同名 .ts/.js 并存（工具优先取 ts，残留 js 误导）；.js 编译产物不入库
- `wx:for` 必须带 `wx:key`；WXML 内仅写简单表达式，复杂逻辑放 TS 或 wxs；wxs 遵循 ES5 语法（var、匿名 function）且为纯函数
- 尺寸统一 rpx；app.wxss 全局 / 页面 wxss 局部 / 组件样式默认隔离；公共样式抽公共 wxss @import（相对路径）；图标统一 iconfont 字体方案
- 页面 / 组件四件套同目录同名（ts/json/wxml/wxss）；组件 json 必含 `"component": true`；usingComponents 按页面 / 全局分级注册
- 组件 properties 原生层用 Constructor 类型（String/Number/Boolean/Object/Array/null），TS 侧补类型注解，对外属性提供默认值

## A.2 配置管理

- project.config.json 放共享配置入库；project.private.config.json 放个人配置入 .gitignore；私有文件同名项优先，但与编译产物相关的设置及 libVersion 的 latest/trial 取值在私有文件无效
- appid：正式 appid 入库、本地测试 appid 私有文件覆盖、CI 上传经 miniprogram-ci 参数显式传入
- 环境区分（dev/prod 后端域名）三件套：env 常量文件 + packOptions.ignore 上传剔除 dev 专用配置 + CLI 编译条件切换；禁业务代码散写域名
- setting 关键项：es6/enhance 同开、minified/minifyWXSS/minifyWXML 上传压缩开启、urlCheck 生产必开（开发期仅私有文件关闭）
- 基础库 libVersion 显式锁定，下限按依赖综合取值；生产 request 合法域名（HTTPS）mp 后台白名单，「不校验合法域名」仅开发期生效

## A.3 API 设计（对后端契约消费）

- 全局唯一 request 封装：基于 wx.request 的泛型方法 `request<T>(opts): Promise<T>`；未设 `dataType:'json'` 时对返回 data 手动 JSON.parse 容错
- 统一响应体建模 `ApiResult<T>`，code 类型与后端契约严格一致（禁前端擅自按 number/string 另定）
- 认证 token 存 storage、请求头注入；401 走静默重登（wx.login 换发 token 后重放原请求），禁弹窗打断
- openid / session_key 等敏感信息仅后端经 code2Session 获取，前端不接触；code 5 分钟有效
- 错误三层处理：网络层 fail / HTTP 层非 2xx / 业务层 code 非成功值，分层提示禁吞错
- 并发纪律：wx.request 最大并发 10，密集请求队列化 / 合并

## A.5 渲染性能

- setData 最小化：路径表达式局部更新，禁整对象 / 整列表回写；后台态页面暂停 setData；连续多次合并
- data 仅存渲染相关数据（渲染无关挂 this 实例）；**图片地址禁入 data**（本地图片放页面目录、网络图片直写 wxml）
- 长列表阈值 100 条：以内整体 setData + wx:key；更长用分页 + recycle-view 虚拟列表（createRecycleContext 页面临界生命周期清理）
- 分包纪律：主包仅 tabBar / 首页 / 公共代码；重资源页独立分包；分包禁互引 js/template/资源；跨分包复用组件上移主包
- 体积红线（硬约束）：单个主包 / 分包 ≤2MB、总包 ≤20M；`preloadRule` 预下载重分包
- 图片 image `lazy-load`（仅 page 与 scroll-view 下生效）并处理 bindload/binderror；原生组件层级最高，覆盖场景注意同层渲染
- 动效性能铁律（**属性级**，与动效工具无关）：逐帧动效仅走 transform / opacity 合成属性；布局类动效（尺寸 / 位置过渡）为唯一例外通道，须显式注释声明重排代价并优先以 FLIP 思路承载（先测布局位、再以 transform 过渡）
- ECharts 专项：echarts.js 官网按需构建替换且保持文件名 echarts.js；基础库 ≥2.9.0 默认 canvas 2d（禁退回旧 canvas）；初始化 lazy-load 模式（数据到达后再 init）；ec-canvas 运行时放图表分包
- 启动性能：`lazyCodeLoading: "requiredComponents"` 按需注入；非首屏请求延后；结合分包预下载

## A.7 跨层数据对象与传参约束（本栈习语）

1. **interface / type 分工**：对象形状一律 interface（可 extends / 声明合并）；联合类型、字面量类型、工具类型用 type；禁同一对象两套混用
2. **请求 / 响应分别建模**：每端点 ReqXxx / RespXxx 一一对应，统一经 `ApiResult<T>` 包装；禁 any、禁共享万能 DTO；类型集中管理禁跨模块重复定义
3. **参数对象化**：组件对外数据一律 properties 承载（Constructor 类型 + TS 注解 + 默认值），复杂结构收拢为 object 类型 property；导出函数（utils / api 模块等）形参 > 3 必须收拢为单一参数 interface / type 整体传参，禁逐参罗列
4. **职责隔离**：接口 DTO 与页面渲染 model 分开定义、转换函数集中一处；领域状态机用字符串字面量联合类型建模，禁裸 number 魔法值
5. **例外边界**：仅业务确需动态 / 灵活结构可偏离且须能陈述业务理由；无理由违反视为缺陷不得合入

## C.3 目录职责骨架

```text
<子项目根>/
├── package.json            # 依赖定稿（参与构建 npm 的依赖必须在 dependencies）
├── package-lock.json       # 锁文件入库禁手改
├── project.config.json     # 公共配置（miniprogramRoot、libVersion、setting、packNpm 关系）
├── project.private.config.json  # 个人配置，入 .gitignore
├── tsconfig.json           # strict 基线 + api-typings + 组件库路径映射
├── scripts/                # 子项目脚本（敏感词扫描等），禁业务逻辑
└── <miniprogramRoot>/      # 源码根（与 node_modules 隔离，packNpmManually + packNpmRelationList 指定关系）
    ├── app.ts/.json/.wxss  # 全局入口与唯一全局注册（pages/subpackages/tabBar/preloadRule/lazyCodeLoading）
    ├── pages/              # 主包页面（tabBar / 首页 / 高频页）
    ├── <分包>/             # 重资源分包（如图表：报表页 + ec-canvas 运行时就近放置）
    ├── components/         # 跨分包复用公共组件唯一落点（哑组件为主）
    ├── api/                # 后端交互唯一出口（request 泛型封装 + 端点模块）
    ├── types/              # 领域类型集中（Req/Resp 契约、状态机字面量联合）
    ├── utils/              # 通用工具；禁业务依赖
    └── assets/             # 静态资源
```

## C.4 常用命令骨架

```bash
npm install            # 依赖安装（单一小程序仓库推荐 npm；官方未承诺 pnpm）
npm run lint           # eslint + prettier 检查
npm run lint:fix       # 自动修复
npx tsc --noEmit       # 独立类型检查（工具编译不查类型，CI 必备）
npm run scan           # 敏感词扫描（禁词表驱动，命中输出 文件:行号 并失败）
npm run build:npm      # 构建 npm（miniprogram-ci packNpm，生成 miniprogram_npm）
npm run upload         # CI 上传（miniprogram-ci upload，密钥环境注入禁入库）
npm run preview        # 预览出码（miniprogram-ci preview）
```

- 构建 npm 三途径：工具菜单「工具 → 构建 npm」（本地图形等价物）/ 命令行 cli build-npm --project（需工具安全设置开启服务端口）/ miniprogram-ci packNpm（CI）
- 源码与 npm 目录分离时 `setting.packNpmManually=true` + `packNpmRelationList`（packageJsonPath + miniprogramNpmDistDir），CI 中保持与 project.config.json 一致
- **pnpm 布局约束**：微信「构建 npm」按标准 npm node_modules 布局做依赖分析，pnpm 默认符号链接布局会导致 miniprogram_npm 空产物（官方未承诺兼容）；确需 pnpm 必须项目根 `.npmrc` 加 `node-linker=hoisted` 且切换布局时删 node_modules 重装，禁混用；单一小程序仓库直接用 npm
- miniprogram-ci 上传：appid/type/projectPath/privateKeyPath/version/robot 1-30 显式传参；密钥 mp 后台下载、禁入库、配 IP 白名单；TS 项目 miniprogram-ci ≥1.9.0 线（以实测为准）

## 附加分节

### 隐私授权（wx.requirePrivacyAuthorize 体系）

- mp 后台先配置《小程序用户隐私保护指引》声明采集用途，再接客户端弹窗；基础库 ≥2.32.3
- 自定义弹窗经 `wx.onNeedPrivacyAuthorization` 监听，确认必须来自用户有效点击（bindagreeprivacyauthorization），禁 showModal 模拟
- 授权结果官方缓存（同意后不再弹），前端不做本地二次弹窗；未处理回调上限 3 次，拒绝后引导 wx.openPrivacyContract

### 广告 banner 组件

- 底部横幅用 banner 广告组件绑定 ad-unit-id（需开通流量主）；bindload 设可视高度、binderror 无填充高度归 0 不占位、bindresize 动态调布局
- 备选 ad 组件须处理错误码（1004 无适合广告 / 1005 封禁）；严禁诱导点击 / 自行点击（封禁风险）；新建广告位注意官方接口版本变更公告

### 敏感词扫描门禁

- 微信无官方源码级敏感词扫描 CLI：自写 Node 脚本（遍历源码文本文件、禁词表配置驱动、命中输出 文件:行号 exit 1）或 ESLint 自定义规则，入 CI 门禁；二选一
- 图片资源内文字无法静态扫描 → 素材流程管控（设计自查 + 词表评审）；提审前跑微信官方「小程序安全检测」兜底

## 来源

- 微信官方文档：compilets（TS 编译特性）、projectconfig、npm、cli、ci、wx.request、登录、setData 优化、长列表、分包 ×3、image、原生组件、WXML/WXSS、自定义组件、隐私授权、banner/ad 组件（developers.weixin.qq.com）
- GitHub 官方仓库：wechat-miniprogram/api-typings、miniprogram-ts-quickstart、recycle-view；ecomfe/echarts-for-weixin
- TDesign 官方文档（tdesign.tencent.com/miniprogram）；pnpm 官方 settings/node-modules；TypeScript / ESLint 官方文档
