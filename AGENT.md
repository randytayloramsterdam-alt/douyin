# AGENT.md

本文件给后续在本仓库工作的 AI Agent / 维护者使用。请优先用中文沟通，改动前先理解现有结构和移动端交互逻辑，避免做无关重构。

## 项目概览

这是一个名为 `douyin-vue` 的 Vue3 移动端短视频仿抖音项目。核心体验是移动端竖滑视频流、横向频道切换、用户页、消息页、商城页、登录页等抖音式页面。项目使用本地 JSON 数据和 `axios-mock-adapter` 模拟后端接口，不依赖真实服务端。

技术栈：

- Vue 3 + `<script setup>`
- Vite
- TypeScript
- Vue Router 4
- Pinia
- Less
- axios + axios-mock-adapter
- `@jambonn/vue-lazyload` 图片懒加载
- `@iconify/vue` 图标
- `libarchive-wasm` 用于特定 Pages 场景下读取伪装成 `.md` 的压缩数据

## 快速命令

优先使用项目已有脚本，不要临时发明新命令。

```bash
pnpm install
pnpm dev
pnpm build
pnpm type-check
pnpm lint
pnpm format
pnpm preview
```

常用脚本说明：

- `pnpm dev` / `pnpm start` / `pnpm serve`：启动 Vite，本地端口默认 `3000`，并监听 `0.0.0.0`。
- `pnpm build`：生产构建。
- `pnpm build-check`：先类型检查再构建。
- `pnpm type-check`：运行 `vue-tsc --build --force`。
- `pnpm lint`：运行 ESLint 并自动修复。
- `pnpm format`：用 Prettier 格式化 `src/`。
- `pnpm report`：构建并启用 Rollup visualizer。
- `pnpm build-gp-pages` / `pnpm build-gitee-pages` / `pnpm build-uni-app`：按不同部署模式构建。

本项目主要面向移动端预览。PC 浏览器调试时，需要打开开发者工具并切换到移动设备模拟模式。

## 顶层目录结构

```text
.
├── .github/              # GitHub issue 模板和 CI/CD 工作流
├── .husky/               # Git hooks：pre-commit、commit-msg
├── docs/                 # 多语言 README 和项目说明资源
├── env/                  # Vite 环境文件，控制 VITE_ENV
├── node/                 # 数据清洗、转换、采集后处理脚本
├── public/               # 静态资源和前端运行时读取的数据
├── src/                  # Vue 应用源码
├── Dockerfile            # Docker 多阶段构建
├── Makefile              # Docker 构建/运行辅助命令
├── package.json          # 前端项目依赖和脚本
├── pnpm-lock.yaml        # pnpm 锁文件
├── vite.config.ts        # Vite 配置
└── README.md             # 项目说明
```

## `src/` 源码结构

```text
src/
├── main.ts               # 应用入口，注册 Pinia、Router、mixin、懒加载、mock
├── App.vue               # 根组件，控制路由转场、keep-alive、全局 Call 组件
├── api/                  # API 函数封装
├── assets/               # 图片、Less、内置数据
├── components/           # 全局复用组件
├── config/               # 环境和静态资源 URL 配置
├── mock/                 # axios-mock-adapter 接口模拟
├── pages/                # 页面级 Vue 组件
├── router/               # 路由表和路由实例
├── store/                # Pinia store
└── utils/                # 工具函数、事件总线、滑动逻辑、hooks
```

### 入口与全局初始化

- `src/main.ts`
  - 创建 Vue 应用。
  - 引入全局样式 `src/assets/less/index.less`。
  - 注册 `Pinia`、`Vue Router`、全局 mixin、图片懒加载。
  - 注册自定义 `v-click` 指令。
  - 调用 `startMock()` 启动本地接口模拟。
  - 给 `HTMLElement.prototype.addEventListener` 包了一层代理，用 `window.isMoved` 避免滑动后误触点击事件。
  - 初始化全局静音相关状态：`window.isMuted`、`window.showMutedNotice`。

- `src/App.vue`
  - 根路由出口使用 `<router-view>` + `<transition>` + `<keep-alive>`。
  - 根据路由前后顺序选择 `go` / `back` 转场。
  - 监听窗口 resize，重新计算 `--vh`，并回到 `BASE_URL + '/'`。
  - 通过 `store.excludeNames` 控制哪些页面不缓存。

### 路由

- `src/router/index.ts`
  - 根据 `IS_SUB_DOMAIN` 选择 `createWebHashHistory()` 或 `createWebHistory()`。
  - `beforeEach` 中根据路由顺序维护 `excludeNames`，用于前进/后退时的页面缓存策略。

- `src/router/routes.ts`
  - 所有页面路由集中定义。
  - `/` 重定向到 `/home`。
  - 主要页面包括：
    - `/home` 首页短视频主界面
    - `/shop` 商城
    - `/me` 我的
    - `/message` 消息
    - `/publish` 发布
    - `/login` 登录
    - `/video-detail` 视频详情
  - 多数二级页面使用动态 import 懒加载。

修改路由时要同步注意：

- 路由顺序会影响转场方向判断。
- 底部主 Tab 页面通常不需要动画，需要检查 `App.vue` 和 `router/index.ts` 里的 `noAnimation` 列表。
- 如果页面需要被 keep-alive 正确缓存/排除，组件名和路由守卫逻辑要一起考虑。

### 状态管理

- `src/store/pinia.ts`
  - 定义 `useBaseStore`。
  - 存储全局尺寸、遮罩弹窗、版本号、keep-alive 排除列表、用户信息、好友数据、加载状态等。
  - `init()` 会调用 `panel()` 和 `friends()` 初始化当前用户和用户列表。
  - `updateExcludeNames()` 用于路由前进/后退时动态调整缓存排除列表。

### 接口与 mock 数据流

接口封装位于：

- `src/api/videos.ts`
- `src/api/user.ts`

请求实例位于：

- `src/utils/request.ts`

mock 实现位于：

- `src/mock/index.ts`

数据流大致如下：

```text
页面组件
  -> src/api/*.ts
  -> src/utils/request.ts 的 axiosInstance
  -> src/mock/index.ts 拦截请求
  -> src/assets/data/*.json 或 public/data/*.json/.md
  -> 返回统一的 { success, code, data } 结构
```

注意点：

- `axiosInstance` 的 `baseURL` 来自 `src/config/index.ts` 的默认导出 `baseUrl`。
- `startMock()` 在 `main.ts` 最后调用，因为部分 mock 逻辑依赖 Pinia。
- `_fetch()` 在开发环境或非 Gitee Pages 环境会把 `.md` 替换为 `.json` 再请求。
- Gitee Pages 场景下，部分 `.md` 实际上用于规避 JSON 文件限制，会通过 `libarchive-wasm` 解压读取内部 JSON。
- mock 中的推荐视频初始来自 `src/assets/data/posts6.json`，随后异步合并 `public/data/videos.*`。
- 评论、用户视频列表、商品、帖子等数据主要来自 `public/data/`。

### 页面结构

`src/pages/` 按业务区域划分：

```text
pages/
├── home/                 # 首页、推荐流、直播、音乐、搜索、发布、举报
├── login/                # 登录、密码登录、验证码、找回密码、国家选择
├── me/                   # 我的主页、资料编辑、收藏、设置、青少年模式等
├── message/              # 消息列表、聊天、通知、粉丝、访客等
├── other/                # 视频详情、图集详情、服务协议
├── people/               # 通讯录、扫一扫、面对面、关注粉丝等
├── shop/                 # 商城和商品详情
└── test/                 # 测试页面
```

首页关键文件：

- `src/pages/home/index.vue`
  - 首页总容器。
  - 外层 `SlideHorizontal` 管理左侧功能栏、主视频区、用户面板。
  - 内层 `SlideHorizontal` 管理频道切换。
  - 通过事件总线控制评论、分享、全屏、用户面板等行为。

- `src/pages/home/slide/Slide0.vue`
  - 推荐视频流入口之一。

- `src/pages/home/slide/LongVideo.vue`
  - 长视频频道。

- `src/pages/home/slide/Community.vue`
  - 图文/社区类内容。

### 组件结构

`src/components/` 中有两类组件：

1. 通用 UI / 业务弹层
   - `BaseHeader.vue`
   - `BaseFooter.vue`
   - `BaseMask.vue`
   - `Comment.vue`
   - `Share.vue`
   - `DouyinCode.vue`
   - `UserPanel.vue`
   - `dialog/*`

2. 滑动和视频播放核心
   - `slide/SlideHorizontal.vue`
   - `slide/SlideVertical.vue`
   - `slide/SlideVerticalInfinite.vue`
   - `slide/SlideItem.vue`
   - `slide/BaseVideo.vue`
   - `slide/ItemToolbar.vue`
   - `slide/ItemDesc.vue`
   - `slide/SlideUser.vue`

滑动组件是项目体验核心，改动时要谨慎。

### 滑动系统

核心文件：

- `src/utils/slide.ts`
- `src/components/slide/SlideVerticalInfinite.vue`
- `src/components/slide/SlideHorizontal.vue`
- `src/utils/const_var.ts`

主要机制：

- 使用 pointer 事件处理移动端和 PC 调试。
- `slideTouchStart` 记录起点。
- `slideTouchMove` 判断方向、计算位移、阻止误触。
- `slideTouchEnd` 根据距离和时间判断是否切换到上一项/下一项。
- `slideReset` 复位 transform、状态和 `window.isMoved`。
- `SlideVerticalInfinite.vue` 做虚拟列表，只保留有限数量 DOM 节点，滚动时动态插入/卸载。

修改滑动逻辑时重点验证：

- PC 移动设备模拟模式下 pointer 事件是否正常。
- 手机浏览器触摸滑动是否正常。
- 竖滑视频、横滑频道、首页左侧栏三者是否互相抢事件。
- 滑动后点击是否被正确抑制，避免误触。
- 无限列表边界：第一条下拉刷新、最后一条加载更多。

### 工具层

常用工具：

- `src/utils/index.tsx`
  - 本地存储、复制、数字格式化、时间格式化、弹窗创建、toast、mock 数据 fetch、视频/图片渲染分发。

- `src/utils/bus.ts`
  - 全局事件总线和事件 key。

- `src/utils/dom.ts`
  - DOM 样式读写工具。

- `src/utils/hooks/useNav.ts`
  - 路由跳转封装。

- `src/utils/hooks/useClick.ts`
  - 自定义点击指令。

- `src/utils/enums.ts`
  - 枚举常量。

- `src/utils/const_var.ts`
  - 默认用户、滑动类型等常量。

## 静态数据与数据加工

### 前端运行时数据

`public/data/` 是运行时可访问数据目录：

```text
public/data/
├── videos.json / videos.md
├── posts.json / posts.md
├── users.json / users.md
├── goods.json / goods.md
├── comments/
└── user_video_list/
```

`src/assets/data/` 是构建时打包数据：

```text
src/assets/data/
├── posts6.json
├── posts6-old.json
├── resource.js
└── lyrics/
```

注意：

- `posts6.json` 被直接 import 到 mock 层，属于首屏/初始推荐数据。
- `public/data/*.md` 在部分部署环境下可能不是普通 Markdown，相关读取逻辑在 `_fetch()`。
- 不要随意删除 `.json` 和 `.md` 的配对文件，部署脚本和 mock 逻辑可能依赖两者。

### `node/` 数据处理脚本

`node/` 是独立 Node 脚本区，用于清洗和生成前端数据，不是 Vite 应用源码。

常见文件：

- `node/process-post-list.js`
  - 读取 `node/post/data/` 下各用户视频数据，交错合并生成 `posts6.json` 和 `posts.json`。

- `node/post/process-post.js`
  - 从输入数据中保留需要的字段，追加到指定用户 JSON。

- `node/post/process-post-img.js`
  - 处理帖子图片相关数据。

- `node/user/process-user.js`
  - 处理用户数据。

- `node/comment/process.js`
  - 处理评论数据。

- `node/xhs/process-xhs-img.js`
  - 处理小红书图片数据。

运行 `node/` 脚本前先确认当前工作目录。很多脚本使用相对路径，直接从仓库根目录或 `node/` 目录运行可能产生不同结果。

## 环境与部署

环境文件位于 `env/`：

```text
env/.env              -> VITE_ENV = "DEV"
env/.env.prod         -> VITE_ENV = "PROD"
env/.env.gitee_pages  -> VITE_ENV = "GITEE_PAGES"
env/.env.gp_pages     -> VITE_ENV = "GP_PAGES"
env/.env.uni          -> VITE_ENV = "UNI"
```

`src/config/index.ts` 根据 `VITE_ENV` 派生：

- `BASE_URL`
- `IMG_URL`
- `FILE_URL`
- `IS_SUB_DOMAIN`
- `IS_GITEE_PAGES`
- `IS_DEV`

部署相关文件：

- `vite.config.ts`
  - `base: './'`
  - `envDir: 'env'`
  - CDN import：Vue、Vue Router、Vue Demi、Mock.js
  - 根据 `report` 脚本启用 visualizer
  - 手动 chunk 拆分部分页面到 `other`
  - 注入 `LATEST_COMMIT_HASH`
  - 开发服务器端口 `3000`
  - preview 端口 `5555`

- `Dockerfile`
  - Node 20 builder 阶段执行 `pnpm install` 和 `npm run build`
  - 最终镜像基于 `ghcr.io/rookie-luochao/nginx-runner:latest`

- `Makefile`
  - 提供 Docker build/run 辅助目标。

- `.github/workflows/`
  - GitHub Pages、Gitee Pages、Docker 镜像、README 相关工作流。

## 代码风格

格式化配置：

- 2 空格缩进
- 不使用分号
- 单引号
- `printWidth: 100`
- `trailingComma: none`

ESLint：

- 使用 Vue3 essential、eslint recommended、TypeScript、Prettier skip-formatting。
- 关闭 `vue/multi-word-component-names`。
- 忽略 `vite.config.ts` 和 `mobile-select.js`。

提交规范：

- commitlint 使用 `@commitlint/config-conventional`。
- Husky：
  - `pre-commit` 执行 `npx lint-staged`
  - `commit-msg` 执行 `npx --no-install commitlint --edit $1`

## 开发注意事项

1. 优先保持现有目录和代码风格。
2. 页面级组件放到 `src/pages/` 对应业务目录。
3. 可复用组件放到 `src/components/`。
4. 接口函数放到 `src/api/`，mock 响应同步放到 `src/mock/index.ts`。
5. 全局状态优先放到 `src/store/pinia.ts`，局部状态留在组件内。
6. 样式以 Less 为主，全局变量放 `src/assets/less/index.less` 或拆分的 Less 文件。
7. 路由新增后检查转场、keep-alive、底部 Tab 动画豁免。
8. 涉及视频流、滑动、事件总线的改动要做移动端交互验证。
9. 涉及数据文件时，确认是构建时数据 `src/assets/data`，还是运行时静态数据 `public/data`。
10. 不要随意改动 `window.isMoved`、全局点击代理和 slide touch 逻辑，它们用于避免滑动误触。
11. 不要随意把 `.md` 数据文件当普通文档删除或改名，它们可能参与部署兼容逻辑。
12. 依赖版本以 `pnpm-lock.yaml` 为准。

## 推荐改动流程

处理功能或修复时建议按以下顺序：

1. 读相关页面组件，确认状态和事件来源。
2. 读对应复用组件，尤其是弹层、滑动、视频播放组件。
3. 读 `src/api/` 和 `src/mock/index.ts`，确认数据结构。
4. 如涉及路由，检查 `src/router/routes.ts` 和 `src/router/index.ts`。
5. 小范围修改，避免跨模块重构。
6. 运行至少一个验证命令：
   - 纯类型/结构改动：`pnpm type-check`
   - 构建相关改动：`pnpm build`
   - 样式和组件改动：`pnpm dev` 后手动移动端预览
   - 代码风格改动：`pnpm lint`

## 常见任务指引

### 新增页面

1. 在 `src/pages/<业务目录>/` 新增 Vue 文件。
2. 在 `src/router/routes.ts` 增加路由。
3. 如页面由底部 Tab 进入，检查 `noAnimation`。
4. 如果需要全局导航，用 `useNav()` 或事件总线保持现有方式。

### 新增 mock 接口

1. 在 `src/api/` 增加 API 函数。
2. 在 `src/mock/index.ts` 增加 `mock.onGet` / `mock.onPost`。
3. 返回结构保持 `{ data, code: 200, msg }`，使 `request.ts` 能统一转成成功结构。
4. 如使用静态数据，放到 `public/data/` 或 `src/assets/data/` 时要说明原因。

### 修改视频流

优先检查这些文件：

- `src/pages/home/index.vue`
- `src/pages/home/slide/*.vue`
- `src/components/slide/SlideVerticalInfinite.vue`
- `src/components/slide/BaseVideo.vue`
- `src/utils/slide.ts`
- `src/utils/bus.ts`
- `src/mock/index.ts`

视频流问题通常不是单文件问题，可能同时涉及数据、播放状态、事件总线和滑动状态。

### 修改样式

优先使用已有变量：

- `--main-bg`
- `--active-main-bg`
- `--primary-btn-color`
- `--footer-height`
- `--common-header-height`
- `--indicator-height`
- `--page-padding`

移动端尺寸大量使用 `rem`，根字号在全局样式中设置为 `1px`，实际 `14rem` 可近似理解为 14px。

## 已知特殊点

- `src/components/mobile-select/mobile-select.js` 是第三方/遗留脚本，ESLint 已忽略。
- `src/components/slide/SlideAlbum.txt`、`SlideVerticalInfinite2.txt` 是文本形态保留文件，不是当前主要入口。
- `node/ouput.json` 文件名疑似拼写为 `ouput`，不要未经确认直接改名，因为脚本可能依赖。
- `src/mock/index.ts` 中有 TODO：推荐视频分页初始返回 6 条时可能导致第二次请求从第 10 条开始，漏掉中间 4 条。
- `vite.config.ts` 中 CDN import 的 Vue 版本路径和 `package.json` 中 Vue 版本不完全一致，改 CDN 配置时要额外验证。
- `App.vue` resize 时会跳回首页，这会影响桌面调试体验，修改前确认是否有业务意图。

## 给 Agent 的工作约束

- 用中文回复用户。
- 改动前先看相关文件，不要只凭文件名猜。
- 保持改动聚焦，避免顺手格式化全仓库。
- 不要覆盖用户未提交改动。
- 不要运行破坏性 git 命令。
- 如需安装依赖或访问网络，先说明原因并等待授权。
- 文档、注释、命名优先沿用项目已有中文风格。
- 交付前说明改了什么、验证了什么、还有什么没验证。
