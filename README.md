# DocMind Frontend - AI 知识库问答前端

DocMind 的前端应用，基于 UniApp（Vue3）开发，支持 H5 和微信小程序双端发布。

## 在线体验

**H5 版本：** http://47.82.90.240/doc/

## 技术栈

- UniApp — 跨平台前端框架（H5 + 微信小程序）
- Vue 3 — Composition API + `<script setup>`
- SCSS — 样式预处理

## 核心功能

- 📎 PDF 文档上传，一键建立知识库
- 📂 多文档管理，顶部文档栏切换，支持删除
- 💬 自然语言提问，AI 基于所选文档精准回答
- ⚡ SSE 流式输出，逐字显示，类 ChatGPT 体验
- 🔄 多轮对话上下文，AI 记得上一句说了什么
- 🗨️ 聊天气泡界面，用户/AI 消息左右区分
- 📜 对话历史自动滚动到最新消息
- 🔀 切换文档自动清空对话，避免上下文混淆

## 项目结构

```
doc-mind-frontend/
├── pages/
│   └── index/
│       └── index.vue    # 主页面（文档管理 + 对话）
├── static/              # 静态资源
├── App.vue              # 根组件
├── main.js              # 入口文件
├── manifest.json        # 应用配置（含 H5 base 路径）
└── pages.json           # 页面路由配置
```

## 本地运行

**1. 克隆项目**

```bash
git clone https://github.com/qingfengsuiyu/doc-mind-frontend.git
```

**2. 用 HBuilderX 打开项目**

下载 [HBuilderX](https://www.dcloud.io/hbuilderx.html)，打开项目文件夹。

**3. 配置后端地址**

在 `pages/index/index.vue` 中，将接口地址改为本地后端地址：

```javascript
// 本地开发
const BASE_URL = 'http://127.0.0.1:8000'

// 生产环境
// const BASE_URL = 'http://47.82.90.240/doc-api'
```

**4. 启动**

顶部菜单 → 运行 → 运行到浏览器 → Chrome

## 生产部署

打包：顶部菜单 → 发行 → 网站-PC Web或手机H5 → 发行

打包产物位于 `unpackage/dist/build/web/`，上传到服务器后通过 Nginx 提供静态文件服务。

Nginx 关键配置（SSE 流式输出需关闭缓冲）：

```nginx
location /doc/ {
    alias /path/to/dist/web/;
    index index.html;
    try_files $uri $uri/ @doc_fallback;
}

location /doc-api/ {
    proxy_pass http://localhost:8000/;
    proxy_buffering off;     # SSE 必须关闭缓冲
    proxy_cache off;
    proxy_set_header Connection '';
}
```

## 后端项目

本项目依赖 [doc-mind-backend](https://github.com/qingfengsuiyu/doc-mind-backend) 提供接口服务，请先启动后端。

## 开发计划

- [x] 聊天气泡界面
- [x] PDF 文档上传
- [x] 多文档管理（上传/切换/删除）
- [x] 问答接口联调
- [x] SSE 流式输出（逐字显示）
- [x] 多轮对话上下文
- [x] 自动滚动到底部
- [x] 切换文档清空对话
- [x] 部署上线
- [ ] 上传进度条
- [ ] 回答来源引用显示
- [ ] JWT 用户鉴权
- [ ] 微信小程序适配发布

## License

MIT