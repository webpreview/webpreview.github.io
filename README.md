# Vue Admin + Electron

基于 `vue-admin-template`（Vue 2 + Element UI）集成 Electron，可同时作为 **Web 应用** 与 **桌面应用** 运行。

## 预览
<p align="left">
  <img width="900" src="https://cdn.jsdelivr.net/gh/webpreview/img-cdn@main/vue-admin-electron-pre.png">
</p>

## 功能

- 登录 / 注销（桌面端由主进程本地 Mock 服务提供接口，无需后端）
- 权限校验（路由级 + `v-permission` 指令级）
- 多语言（中/英/西/日）
- 动态侧边栏、面包屑、TagsView
- 全局搜索（支持拼音）、全屏
- 开发环境热重载（HMR）

## 项目结构

```
├── mock                     # Mock 数据（mockjs，供主进程本地 HTTP 服务使用）
├── public                   # 静态资源（favicon 等）
├── src
│   ├── api                  # 接口请求
│   ├── background.js        # Electron 主进程（窗口 / 本地 Mock 服务）
│   ├── preload.js           # 预加载脚本（向渲染进程暴露 API 地址）
│   ├── components           # 通用组件（含 HeaderSearch）
│   ├── router               # 路由（hash 模式，适配 file:// 协议）
│   ├── store / views / ...  # Vuex / 页面
│   ├── utils/request.js     # axios 封装，baseURL 取自 preload 或环境变量
│   ├── main.js              # 入口（Electron 构建时关闭渲染进程内 mock）
│   └── ...
├── build                    # 打包资源目录（图标等）
├── vue.config.js            # publicPath 按环境切换，devServer 在 Electron 模式不自动开浏览器
└── package.json             # main / build / electron 脚本与依赖
```

## 环境差异与登录请求原理

| 场景 | 页面来源 | API 地址 | 接口由谁提供 |
| --- | --- | --- | --- |
| `npm run dev`（Web 开发） | http://localhost:9528（dev-server） | 相对路径（dev-server 中间件） | vue-cli dev-server 的 mock 中间件 |
| `npm run electron:serve`（桌面开发） | http://localhost:9528（dev-server） | 相对路径 | dev-server mock 中间件 + HMR |
| `npm run electron:build`（桌面生产） | `http://127.0.0.1:34567`（主进程同源托管） | `http://127.0.0.1:34567`（同源，空串亦可） | **主进程本地 HTTP 服务（同源托管页面 + Mock API）** |

> 关键点：早期方案以 `file://` 加载页面、再用 `http://127.0.0.1` 调接口，会因 `file://` 源 + 跨域(CORS)在部分环境下报 `Network Error`。现改为**主进程在同一端口同源托管页面与 API**，渲染进程以 `http` 加载，彻底规避 `file://` 协议与跨域问题。

## 开发（Web）

```bash
npm install
npm run dev              # http://localhost:9528
```

## 桌面端开发（带热重载）

```bash
npm run electron:serve   # 并发启动 dev-server 与 Electron，F12 可开 DevTools
```

## 桌面端打包

```bash
npm run electron:build:win     # Windows (nsis)
npm run electron:build:mac     # macOS (dmg)
npm run electron:build:linux   # Linux (AppImage)
npm run electron:build         # 当前平台
```

打包脚本会注入 `VUE_APP_BASE_API=http://127.0.0.1:34567` 与 `VUE_APP_ELECTRON=true`：
- `VUE_APP_ELECTRON=true`：关闭渲染进程内 Mock，改由主进程服务接管。
- `VUE_APP_BASE_API`：作为 API 地址兜底（实际以 `preload` 暴露的 `electronAPI.apiBaseUrl` 为准）。

产物输出到 `dist_electron/`。

## 登录

- 账号：`admin` / `editor` / `zhangsan` 等，密码任意（≥6 位），由主进程 Mock 服务返回对应角色路由。
- 具体账号见 `mock/user.js`。

### 接入真实后端（替代 Mock）

1. 创建 `.env.production`（或对应环境文件），设置：
   ```bash
   VUE_APP_MOCK=false
   VUE_APP_BASE_API=https://your-api.example.com
   ```
2. 重新执行 `npm run electron:build:win`。
3. 后端需允许跨域：由于桌面端源为 `file://`（null origin），响应头需包含
   `Access-Control-Allow-Origin: *`（或 `null`），并允许 `Content-Type`、`X-Token` 头与 `OPTIONS` 预检。

## 权限控制（RBAC）

基于角色的访问控制：用户关联角色，角色决定可访问路由与界面元素。

- **路由级**：动态路由经 `generateRoutes` 过滤后以 `addRoute` 注入。
- **指令级**：`<el-button v-permission="['admin']">删除</el-button>`。

内置角色见 `mock/user.js`：`admin`（全部）、`editor`、`addor`、`lisi` 等。

## 常见问题

- **打包后登录报 Network Error**：确认是桌面生产环境。若误用渲染进程内 Mock，请改用本方案的「主进程本地 Mock 服务」；若用真实后端，检查 `VUE_APP_BASE_API` 与后端 CORS。
- **HeaderSearch 报 weights 超限**：`src/components/HeaderSearch/index.vue` 中 Fuse 各 `key` 的 `weight` 之和必须 ≤ 1（已设为 `0.6/0.2/0.2`）。

## License

MIT
