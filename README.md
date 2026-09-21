# 布局布线（P&R）学习笔记

一套从「看得懂」到「学得深」的 P&R（Placement & Routing，布局布线）学习资料。先建立芯片设计的全局地图，再梳理六阶段经典算法，最后看清学术前沿与工业实践的分歧与融合。

## 📖 在线文档

| 文档 | 内容简介 | 规模 |
| :--- | :--- | :--- |
| **[经典方法综述：算法谱系与技术梳理](./classic-methods.html)** | 六阶段 56 篇经典文献，含通俗理解 + 电路场景 + 算法详解。从分区、布局规划一路讲到详细布线，是整套笔记的「算法家底」 | 6 阶段 / 56 篇 / 152 段代码 |
| **[文献综述：分开做还是一体化？](./pnr-survey.html)** | 22 篇 2024–2026 年文献，回答「布局与布线到底该分开做还是协同做」。含三条技术路线、四条关键判断，以及一份英文缩写速查表 | 22 篇 / 3 条路线 / 9 类术语 |
| **[芯片设计全流程笔记：从 RTL 到 GDSII](./chip-design-flow.html)** | 把芯片从 RTL 到 GDSII 的全过程串成一张地图，讲清数字 / 模拟 / 存储宏的分工定位，是深入 P&R 之前的预备知识 | 前端 + 后端 / 9 个环节 |

> 三个页面顶部都有一条导航条，可以随时互相跳转；点「首页」回到 [index.html](./index.html)。

## 📂 目录结构

```
.
├─ index.html              # 导航首页（卡片式入口）
├─ classic-methods.html    # 经典方法综述
├─ pnr-survey.html         # 文献综述
├─ chip-design-flow.html   # 全流程笔记
├─ assets/
│   ├─ katex.min.css       # KaTeX 样式（三个页面共用）
│   └─ fonts/              # 60 个 KaTeX 字体文件
└─ src/                    # 原始 Markdown 源码
    ├─ classic-methods.md
    ├─ pnr-survey.md
    └─ chip-design-flow.md
```

## ✨ 页面功能

- **左侧目录树**：按章节自动生成，可折叠 / 展开，支持关键词搜索，滚动时自动高亮当前小节
- **KaTeX 公式**：行内与行间公式全部由 KaTeX 渲染，可离线打开，无需联网
- **深色 / 浅色主题**：跟随系统偏好，手动切换后记忆选择
- **响应式排版**：窄屏自动把侧栏收成抽屉，手机也能看
- **代码块复制**：悬停出现「复制」按钮
- **阅读辅助**：顶部进度条、回到顶部、标题锚点可直达链接
- **打印友好**：每章自动另起一页

## 🛠 重新编译

源码在 `src/` 下，改完 Markdown 后可重新生成 HTML。

> 编译工具链不在本仓库内（含 `build.js` / `style.css` / `app.js` 与 `node_modules`）。如需重新编译，请先安装依赖：

```bash
npm install markdown-it markdown-it-texmath katex
node build.js src/classic-methods.md .
node build.js src/pnr-survey.md .
node build.js src/chip-design-flow.md .
```

## 🌐 部署

本站通过 **GitHub Pages** 发布，直接使用仓库根目录。推送到 `main` 分支后，在
`Settings → Pages → Source` 中选择 `Deploy from a branch`，分支选 `main`、目录选 `/ (root)` 即可。

---

*本仓库仅用于个人学习笔记整理。*
