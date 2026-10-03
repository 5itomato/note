# 🍅 Tomato 的考研数学笔记

个人考研数学复习笔记，基于 [Quartz 5](https://quartz.jzhao.xyz/) 构建并发布为静态站点。

- 🌐 **在线阅读**：https://note.itmt.io/
- 📦 **仓库地址**：https://github.com/5itomato/note
- 📝 **问题反馈**：[Issues](https://github.com/5itomato/note/issues)

## 内容结构

笔记位于 `content/Tomato的考研数学笔记/` 下，按科目分目录组织：

| 目录 | 篇数 | 说明 |
| --- | --- | --- |
| `0. 杂项` | — | 零散知识与公式总结、学习心得（待补充） |
| `1. 高等数学` | 2 | 极限概念与性质、等价无穷小 |
| `2. 线性代数` | 17 | 矩阵运算、行列式、秩、特征值与二次型 |
| `3. 概率论与数理统计` | 3 | 大数定律、中心极限定理、切比雪夫不等式 |

首页 `content/index.md` 提供复习进度与目录导航。数学公式使用 KaTeX 渲染。

## ⚠️ AI 辅助编写声明

**本仓库的笔记内容由 AI 辅助编写**，包括正文、公式推导与总结，并经人工整理、校订与审阅后发布。

- AI 生成的内容**可能存在事实性错误、推导疏漏或表述不严谨**，请务必以教材与课堂讲义为准。
- 建议**不要将本笔记作为唯一复习依据**，使用前请自行核对关键结论与公式。
- 发现错误或有改进建议，欢迎提交 [Issue](https://github.com/5itomato/note/issues)。

## 技术栈

- **Quartz 5** —— 静态站点生成器（Node.js + TypeScript + Preact）
- **GitHub Actions** —— 构建并部署到 GitHub Pages
- **自定义域名** —— `note.itmt.io`（通过 `CNAME` 文件与指向 `5itomato.github.io` 的 DNS 记录）

## 本地开发

需要 Node.js 22+。

```bash
npm install               # 安装依赖
npx quartz build          # 构建到 public/
npx quartz build --serve  # 本地预览（http://localhost:8080）
```

## 部署

推送到 `v5` 分支即自动触发部署，工作流见
[`.github/workflows/deploy-github-pages.yaml`](.github/workflows/deploy-github-pages.yaml)：

1. `npm ci` 安装依赖
2. `npx quartz build` 把 `content/` 构建到 `public/`
3. 上传构建产物并发布到 GitHub Pages

页面路径挂在**域名根**下，例如
`https://note.itmt.io/tomato的考研数学笔记/2.-线性代数/矩阵的秩`，
**没有** `/note` 这一层子路径。

## 配置说明

站点配置在 [`quartz.config.yaml`](quartz.config.yaml)：

- `baseUrl: note.itmt.io` —— 决定 canonical / sitemap / RSS 中的绝对地址，换域名时必须同步修改
- `@quartz-community/cname` 插件**已启用** —— 用于生成指向自定义域名的 `CNAME` 文件

> ⚠️ 若将来不再使用自定义域名、回到 `5itomato.github.io/note` 这类项目站点，
> 需要同时改回 `baseUrl` 并**禁用** CNAME 插件，否则 `CNAME` 文件会把站点指向域名根，
> 导致 `/note/` 下的所有路径失效。

## 许可

本项目统一采用 [MIT License](LICENSE.txt)：既包括 Quartz 框架代码（Copyright © 2021
jackyzha0），也包括本仓库的笔记内容。
