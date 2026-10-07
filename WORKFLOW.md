# 构建 · 提交 · 部署 工作流教程

本仓库（Tomato 的考研数学笔记）从写作到上线的完整手动操作流程。
线上地址：<https://note.itmt.io/>

---

## 0. 核心概念：为什么不需要手动部署

**部署是自动的。** 你只要 `git push` 到 `v5` 分支，GitHub Actions 就会自动构建并发布：

```
你 push v5  →  GitHub Actions  →  npm ci  →  npx quartz build  →  发布到 GitHub Pages
```

工作流文件：[`.github/workflows/deploy-github-pages.yaml`](.github/workflows/deploy-github-pages.yaml)

所以**日常流程只有三步**：

```bash
npx quartz build                      # ① 本地验证能构建
git add -A && git commit -m "..."     # ② 提交
git push                              # ③ 推送（自动部署）
```

> ⚠️ **`public/` 是构建产物，已在 `.gitignore` 中，不要提交**。
> CI 会自己重新构建，提交它只会污染仓库。

---

## 1. 环境准备（只需一次）

```bash
node -v     # 需要 v22 或更高
npm -v
```

仓库有 `.node-version`（当前 `v22.16.0`）。如果 Node 版本不对，用 nvm 切换：

```bash
nvm use 22
```

首次克隆仓库后安装依赖：

```bash
npm install
```

---

## 2. 本地写作与预览

笔记文件放在：

```
content/
├── index.md                          ← 首页（复习进度 + 目录导航）
└── Tomato的考研数学笔记/
    ├── index.md                      ← 笔记集入口
    ├── 0. 杂项/
    ├── 1. 高等数学/
    ├── 2. 线性代数/
    └── 3. 概率论与数理统计/
```

**实时预览**（推荐写作时开着，保存即刷新）：

```bash
npx quartz build --serve
```

然后浏览器打开 <http://localhost:8080>。

> 想顺便预览 Quartz 官方文档：`npm run docs`

---

## 3. 构建验证

推之前**一定要先本地构建一次**，避免推上去才发现报错：

```bash
npx quartz build
```

**成功的标志**：

```
Found 56 input files from `content`
Parsed 56 Markdown files
Emitted 173 files to `public`
Done processing 56 files
```

**需要留意的输出**：

| 输出                                                 | 含义                                   | 处理                 |
| ---------------------------------------------------- | -------------------------------------- | -------------------- |
| `Emitted N files`                                    | 构建成功，产物在 `public/`             | 正常                 |
| `Warning: ... isn't yet tracked by git`              | 新文件还没 `git add`，日期取自文件系统 | 提交后即消失，可忽略 |
| `LaTeX-incompatible input ... unicodeTextInMathMode` | 公式里混了中文标点（如 `，`）          | 建议改掉，非阻断     |
| `Error: ...`                                         | 构建失败                               | **必须修复再推送**   |

快速确认产物正常：

```bash
ls public/index.html public/sitemap.xml public/CNAME
```

---

## 4. 提交

```bash
git status                 # 先看清楚改了什么
git add -A                 # 暂存全部改动
git commit -m "docs: 补充多重积分笔记"
```

**提交信息建议**（仓库现有的风格）：

| 前缀     | 用途                     |
| -------- | ------------------------ |
| `docs:`  | 笔记内容增删改           |
| `style:` | 排版、缩进、标题层级     |
| `fix:`   | 修复链接、域名、构建问题 |
| `ci:`    | 工作流调整               |

---

## 5. 推送并自动部署

```bash
git push
```

推送成功后，构建+部署大约需要 **1–3 分钟**。

### 查看部署状态

浏览器打开：

```
https://github.com/5itomato/note/actions
```

看到 **Deploy Quartz notes to GitHub Pages** 这条，两个任务都要是绿勾：

```
build   ✓     ← 构建
deploy  ✓     ← 发布
```

### 手动重新部署

如果某次推送失败、或想在不改代码的情况下重新发布：

**Actions → Deploy Quartz notes to GitHub Pages → Run workflow → 选 `v5` → Run**

（对应 `workflow_dispatch` 触发器。）

---

## 6. 部署后验收

推完等 1–2 分钟，检查这几项：

```bash
# 首页能打开
curl -I https://note.itmt.io/

# CNAME 没有被改动（自定义域名依赖它）
curl https://note.itmt.io/CNAME        # 应输出 note.itmt.io

# 抽查刚改的页面
curl -I "https://note.itmt.io/tomato的考研数学笔记/2.-线性代数/矩阵的秩"
```

> **路径规则**：页面挂在**域名根**下，正确答案是
> `https://note.itmt.io/tomato的考研数学笔记/...`
> **没有** `/note/` 这一层。写 `/note/` 一定 404。

---

## 7. 常用维护操作

### 7.1 修改复习进度

编辑 [`content/index.md`](content/index.md) 里的进度块（20 格制，百分比 × 0.2 = `#` 个数）：

```text
高等数学总进度：90.0% [##################  ]
线性代数总进度：90.0% [##################  ]
概率论与数理统计总进度：90.0% [##################  ]
```

90% = 18 个 `#`，80% = 16 个，10% = 2 个。**保持每行 20 格宽**，否则等宽字体下会错位。
百分比数值建议统一成 `xx.x%`（5 字符）并左对齐，这样三个 `[` 才能对齐。

### 7.2 排版格式化

仓库使用 Prettier（配置见 [`.prettierrc`](.prettierrc)），`content/` 也在管辖范围内：

```bash
npx prettier content/**/*.md --check     # 检查
npx prettier content/**/*.md --write     # 自动修复
```

全仓库格式化：`npm run format`

> Prettier 会做三件安全的事：规范列表/块引用缩进、给数学块 `$$...$$` 前后补空行、
> 修正块引用内公式的 `>` 前缀。**不会改动公式内容**。

### 7.3 标题层级约定

笔记正文标题**从 `###` 开始**，最深到 `####`：

- `###` —— 一级小节（定义 / 充要条件 / 必要条件 / 其他性质 / 题型与应用）
- `####` —— 二级小节（`1.` `2.` …）

**不要**在正文里写 `# 页面名`，因为 Quartz 已经用文件名作为页面标题，
再写一次会在正文顶部**重复显示一遍页面名**。

### 7.4 检查失效链接

笔记之间用 Obsidian 双链：`[[Tomato的考研数学笔记/2. 线性代数/矩阵的秩|矩阵的秩]]`。
写之前确认目标文件存在，否则点击 404。

---

## 8. ⚠️ 两个高风险配置（改域名时必看）

都在 [`quartz.config.yaml`](quartz.config.yaml) 里，**两者必须一致**：

```yaml
configuration:
  baseUrl: note.itmt.io # 决定 canonical / sitemap / RSS 的绝对地址

plugins:
  - source: "@quartz-community/cname"
    enabled: true # 生成 CNAME 文件，自定义域名依赖它
```

| 场景                 | `baseUrl`                 | CNAME 插件   |
| -------------------- | ------------------------- | ------------ |
| **当前**：自定义域名 | `note.itmt.io`            | **启用**     |
| 回到 GitHub 项目站点 | `5itomato.github.io/note` | **必须禁用** |

> **为什么**：CNAME 插件只取 `baseUrl` 的**主机名**。在项目站点上启用它，
> 会生成内容为 `5itomato.github.io` 的 CNAME，把站点劫持到域名根，
> 导致 `/note/` 下所有路径失效。

**推送前顺手扫一眼配置**，确认没有多出一条自动追加的 `github:quartz-community/cname` 条目：

```bash
git diff quartz.config.yaml
```

---

## 9. 出问题怎么回滚

### 9.1 回滚代码（触发一次重新部署）

```bash
git log --oneline -5                  # 找到要回退到的提交
git revert <commit>                   # 生成一个反向提交（推荐，保留历史）
git push
```

或者硬回退（**会改写历史，慎用**）：

```bash
git reset --hard <commit>
git push --force-with-lease
```

### 9.2 只在 GitHub 上重跑

**Actions → 选择历史成功的那次运行 → Re-run all jobs**

---

## 10. 速查表

```bash
# 写作
npx quartz build --serve              # 本地预览 http://localhost:8080

# 上线三步
npx quartz build                      # ① 验证构建
git add -A
git commit -m "docs: ..."             # ② 提交
git push                              # ③ 推送 → 自动部署

# 检查
git status                            # 改了什么
git diff quartz.config.yaml           # 配置有没有被意外改动
npx prettier content/**/*.md --check  # 排版是否规范

# 验收
curl https://note.itmt.io/CNAME       # 应为 note.itmt.io
```

**部署状态**：<https://github.com/5itomato/note/actions>
**线上站点**：<https://note.itmt.io/>
