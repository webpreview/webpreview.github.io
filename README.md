# 页面预览中心（preview-page）

基于 **Vue 2 + Element-UI** 的页面集中管理与预览平台。将多个页面（含静态模板页与外部链接）统一聚合到一个看板中，支持搜索、排序、详情查看与实时预览，构建产物为纯静态文件，可直接托管到任意静态服务器或 GitHub Pages。

## 预览
> 在线预览：[https://webpreview.github.io/](https://webpreview.github.io/)
<p align="center">
  <img width="900" src="https://cdn.jsdelivr.net/gh/webpreview/img-cdn@main/preview-page.png">
</p>

---

## 技术栈

| 分类 | 技术 |
| --- | --- |
| 框架 | Vue `^2.6.14` |
| UI 组件库 | Element-UI `^2.15.14` |
| 构建工具 | `@vue/cli-service` `^4.5.19`（Vue CLI） |
| 模板编译 | `vue-template-compiler` `^2.6.14` |
| 部署工具 | `gh-pages` `^5.0.0` |

---

## 环境要求

- **Node.js**：建议 `12.x` ~ `16.x`（Vue CLI 4.x 兼容范围；不推荐使用 Node 18+ 以避免潜在依赖问题）
- **包管理器**：`npm`（随 Node 安装）或 `yarn`
- 具备可访问 npm 源的网络环境（首次安装依赖需要联网）

---

## 项目脚本

`package.json` 中已内置以下脚本：

| 命令 | 说明 |
| --- | --- |
| `npm run serve` | 启动本地开发服务器（默认端口 `8080`，自动打开浏览器） |
| `npm run build` | 生产构建，输出到 `dist/` 目录 |
| `npm run lint` | 代码风格检查与自动修复 |
| `npm run deploy` | 构建并发布到 GitHub Pages（`gh-pages` 分支） |

---

## 1. 安装依赖

首次运行或克隆仓库后，需先安装依赖：

```bash
# 进入项目根目录
cd preview-page

# 使用 npm
npm install

# （可选）如使用 yarn
# yarn install
```

依赖安装完成后，项目目录会生成 `node_modules/`（已被 `.gitignore` 忽略，无需提交）。

---

## 2. 运行开发环境

```bash
npm run serve
```

- 默认监听 `http://localhost:8080`，并自动打开浏览器。
- 数据源采用 Mock 模式（详见 [配置说明](#5-配置说明)），数据来自 `public/mockData.json`，修改该文件后刷新页面即可生效，无需重启服务。
- 开发环境使用**相对路径** `publicPath: './'`，资源可正常加载。

如端口被占用，可在 `vue.config.js` 的 `devServer.port` 中修改端口，或临时指定：

```bash
npx vue-cli-service serve --port 3000
```

---

## 3. 构建打包

执行生产构建：

```bash
npm run build
```

构建过程会：

1. 读取 `.env` 与 `.env.production`（Vue CLI 在 `build` 时自动加载 `*.production` 文件）。
2. 依据 `VUE_APP_PUBLIC_PATH` 设置 `publicPath`（生产环境默认为 `/preview-page/`，见 `.env.production`）。
3. 将编译产物输出到 `dist/` 目录（含 `static/` 资源子目录），并将 `public/` 下的静态文件（含 `a/`、`b/`、`c/` 模板页与 `mockData.json`）**原样复制**到 `dist/` 根目录。

> 构建产物 `dist/` 已被 `.gitignore` 忽略，无需提交。

### 产物结构示意

```text
dist/
├── index.html                 # 应用入口
├── mockData.json              # 运行时数据源（可静态修改）
├── a/index.html               # 静态模板页 A
├── b/index.html               # 静态模板页 B
├── c/index.html               # 静态模板页 C
└── static/                    # JS / CSS / 图片等编译资源
    ├── css/
    └── js/
```

---

## 4. 各平台产物使用说明

构建产物为**纯静态文件**，可根据托管场景选择不同的 `publicPath` 配置。关键在于：不同托管位置决定了访问根路径，需保证 `publicPath` 与部署路径一致。

### 场景 A：本地 / 任意「相对路径」静态托管

默认构建（或以相对路径构建）后，可直接用任意静态服务器托管：

```bash
# 1) 构建（使用相对路径 ./）
#    临时指定：
VUE_APP_PUBLIC_PATH=./ npm run build
# 或直接用默认构建（默认生产已设为 /preview-page/，本地托管建议改为 ./）

# 2) 进入产物目录并启动静态服务器
cd dist
npx serve .            # 或：python -m http.server 8080
```

- 访问 `http://localhost:8080`（或 `serve` 输出的地址）即可。
- 此方式适用于本地预览、内网服务器、对象存储（OSS/COS）等**以根目录或任意路径托管**的场景。

### 场景 B：GitHub Pages 项目站点（当前默认）

本仓库已针对 GitHub Pages 项目站点（仓库名 `preview-page`，站点根路径为 `/preview-page/`）配置：

- `.env.production` 中已设置 `VUE_APP_PUBLIC_PATH=/preview-page/`。
- 直接执行 `npm run build` 即生成符合该路径的产物。

部署到 GitHub Pages：

```bash
npm run deploy
```

该命令等价于 `npm run build && gh-pages -d dist`，会：

1. 重新构建；
2. 将 `dist/` 推送到仓库的 `gh-pages` 分支；
3. 在仓库 **Settings → Pages** 中，将发布源设为 `gh-pages` 分支（根目录）后即可访问。

线上地址：[https://webpreview.github.io/](https://webpreview.github.io/)

> 注意：`VUE_APP_PUBLIC_PATH` 结尾必须带 `/`，否则子资源 404。

### 场景 C：自定义子路径 / 自建 Web 服务器（如 Nginx）

若部署到自有域名下的某个子路径（例如 `https://example.com/preview/`），修改 `.env.production` 或构建时传入对应值：

```bash
VUE_APP_PUBLIC_PATH=/preview/ npm run build
```

以 Nginx 托管为例，将 `dist/` 内容放到站点子目录后，访问 `https://example.com/preview/` 即可。Nginx 最小配置参考：

```nginx
server {
  listen 80;
  server_name example.com;

  location /preview/ {
    alias /var/www/preview-page/dist/;
    try_files $uri $uri/ /preview/index.html;
  }
}
```

### 各场景对照表

| 托管场景 | publicPath 取值 | 构建方式 | 访问地址示例 |
| --- | --- | --- | --- |
| 本地 / 相对路径托管 | `./` | `VUE_APP_PUBLIC_PATH=./ npm run build` | `http://localhost:8080/` |
| GitHub Pages 项目站点 | `/preview-page/` | `npm run build`（默认） | `https://<user>.github.io/preview-page/` |
| 自定义子路径 | `/preview/` | `VUE_APP_PUBLIC_PATH=/preview/ npm run build` | `https://example.com/preview/` |
| 自有域名根目录 | `/` | `VUE_APP_PUBLIC_PATH=/ npm run build` | `https://example.com/` |

---

## 5. 配置说明

### 环境变量文件

| 文件 | 生效时机 | 关键变量 |
| --- | --- | --- |
| `.env` | 始终加载 | `VUE_APP_USE_MOCK`、`VUE_APP_API_BASE` |
| `.env.production` | 仅 `npm run build` 时 | `VUE_APP_PUBLIC_PATH=/` |

`.env` 内容要点：

```bash
VUE_APP_USE_MOCK=true        # 是否使用本地模拟数据
VUE_APP_API_BASE=/api        # 对接真实后端时的接口基础路径
```

### 切换为真实后端

1. 将 `.env` 中 `VUE_APP_USE_MOCK` 改为 `false`。
2. 在 `src/api/pageApi.js` 中启用已注释的 axios 实现（视图组件无需改动）。
3. 通过 `VUE_APP_API_BASE` 配置接口前缀（如 `/api`）。

---

## 6. 数据源与更新

页面列表数据来自 **`public/mockData.json`**，运行时由 `src/api/pageApi.js` 通过 `fetch` 读取，**不会被打包进 bundle**。

- 字段：`id`、`name`、`description`、`status`（published/draft/offline）、`icon`、`color`、`previewUrl`、`previewType`、`updatedAt` 等。
- 部署后如需更新页面列表，**直接修改 `public/mockData.json`（或 `gh-pages` 分支上的同名文件）即可，无需重新构建**。
- 若修改了 `public/` 下的 `a/`、`b/`、`c/` 静态模板页，需要重新构建并部署（这些页随构建复制到 `dist/`）。

---

## 7. 目录结构

```text
preview-page/
├── public/                  # 静态资源（构建时原样复制到 dist/）
│   ├── index.html
│   ├── mockData.json        # 运行时数据源
│   ├── a/ b/ c/             # 静态模板页
├── src/
│   ├── api/pageApi.js       # 数据访问层（取数唯一入口）
│   ├── components/          # PageList / PagePreviewDrawer / PageDetailDialog
│   ├── App.vue
│   └── main.js
├── .env                     # 通用环境变量
├── .env.production          # 生产构建环境变量（GitHub Pages 路径）
├── vue.config.js            # Vue CLI 配置（publicPath / outputDir 等）
├── package.json
└── README.md
```

---

## 8. 常见问题

- **子资源 404 / 白屏**：检查 `VUE_APP_PUBLIC_PATH` 是否与部署路径一致，且结尾带 `/`。
- **预览页打不开**：确认 `mockData.json` 中 `previewUrl` 配置正确；外链预览页若禁止被 iframe 嵌入，可改用卡片上的「跳转预览页」按钮在新标签打开。
- **GitHub Pages 不更新**：`npm run deploy` 已包含构建步骤；若手动推送，请确认 `gh-pages` 分支内容为最新 `dist/`。
- **依赖安装慢**：可切换国内 npm 镜像，如 `npm config set registry https://registry.npmmirror.com`。

---

## 许可证

仅供学习与交流使用。
