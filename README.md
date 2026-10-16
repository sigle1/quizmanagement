# Quiz 管理后台

基于 Vue 2、Vite 和 Element UI 实现的 Quiz 管理后台前端项目。

## 功能

- 用户管理：按用户名查询、新增、编辑、删除与分页
- 题目管理：按题目关键字查询、新增、编辑、删除与分页
- 本地数据保存：用户和题目数据会保存在浏览器 `localStorage`
- 响应式布局：适配常见桌面端与移动端宽度

## 技术栈

- Vue 2
- Element UI
- Vite

## 安装与启动

```bash
npm install
npm run dev
```

启动后，访问终端输出的本地地址（默认端口为 `7000`）。

## 构建

```bash
npm run build
```

构建产物会生成在 `dist/` 目录，可将整个目录部署到静态 Web 服务器。

## 项目结构

```text
.
├── src/
│   ├── App.vue       # 用户、题目管理页面与交互逻辑
│   ├── main.js       # Vue 与 Element UI 入口
│   └── style.css     # 页面样式
├── index.html
├── vite.config.js
└── package.json
```
