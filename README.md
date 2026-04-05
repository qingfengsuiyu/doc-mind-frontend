# DocMind Frontend - AI 知识库问答前端

DocMind 的前端应用，基于 UniApp 开发，支持 H5 和微信小程序双端发布。

## 技术栈

- UniApp — 跨平台前端框架
- Vue 3 — Composition API + `<script setup>`
- SCSS — 样式预处理

## 核心功能

- 📎 PDF 文档上传，一键建立知识库
- 💬 自然语言提问，获取 AI 精准回答
- 🗨️ 聊天气泡界面，用户/AI 消息左右区分
- 📜 对话历史记录，自动滚动到最新消息
- ⚡ 请求状态管理，loading 反馈

## 项目结构

```
doc-mind-frontend/
├── pages/
│   └── index/
│       └── index.vue    # 主页面（上传 + 对话）
├── static/              # 静态资源
├── App.vue              # 根组件
├── main.js              # 入口文件
├── manifest.json        # 应用配置
└── pages.json           # 页面路由配置
```

## 本地运行

**1. 克隆项目**

```bash
git clone https://github.com/你的用户名/doc-mind-frontend.git
```

**2. 用 HBuilderX 打开项目**

下载 [HBuilderX](https://www.dcloud.io/hbuilderx.html)，打开项目文件夹。

**3. 配置后端地址**

在 `pages/index/index.vue` 中，将接口地址改为你的后端地址：

```javascript
const BASE_URL = 'http://127.0.0.1:8000'  // 本地开发
// const BASE_URL = 'https://你的域名'     // 生产环境
```

**4. 启动**

顶部菜单 → 运行 → 运行到浏览器 → Chrome

## 后端项目

本项目依赖 [doc-mind-backend](https://github.com/你的用户名/doc-mind) 提供接口服务，请先启动后端。

## 开发计划

- [x] 聊天气泡界面
- [x] PDF 文档上传
- [x] 问答接口联调
- [x] 自动滚动到底部
- [ ] 流式输出（逐字显示）
- [ ] 上传进度条
- [ ] 文档管理页面
- [ ] 微信小程序适配发布

## License

MIT