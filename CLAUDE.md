# CLAUDE.md

本文件为 Claude Code (claude.ai/code) 在处理此存储库中的代码时提供指导。

## 常用命令

```bash
# 本地预览（草稿默认不显示，加 -D 可预览 draft: true 的文章）
hugo server
hugo server -e production      # 按生产环境预览
hugo server -D                 # 同时预览草稿

# 构建静态站点（输出到 docs/ 目录，而非默认的 public/）
hugo
hugo -e production             # 生产环境构建（README 中的正式发布流程）

# 新建文章（Page Bundle 形式：目录名即文章目录，内含 index.zh-cn.md）
hugo new posts/my-post/index.zh-cn.md
hugo new posts/2026-01-23/index.zh-cn.md   # 实际也常用日期作为目录名

# 部署 = 把构建产物提交推送到 GitHub（见下文「部署流程」）
git add docs/
git commit -m "publish"
git push
```

没有 lint / test / format 命令——这是纯内容型 Hugo 站点，质量校验依赖本地 `hugo server` 肉眼预览。

## 高等级架构

### 核心定位与技术栈
- 个人中文博客，静态站点生成器为 **Hugo v0.104.3 extended**（extended 版本才能编译 SCSS）。
- 主题为 **LoveIt**，以 git submodule 形式挂在 `themes/LoveIt`（见 `.gitmodules`）。
- 站点语言 `zh-CN`，`hasCJKLanguage = true`，所有内容文件后缀为 `.zh-cn.md`。

### 关键约定（与其他 Hugo 项目不同之处）
1. **发布目录是 `docs/` 而非 `public/`**：`config.toml` 中 `publishDir = "docs"`。GitHub Pages 被配置为从 `master` 分支的 `/docs` 目录提供服务，因此**构建产物 `docs/` 是直接提交进仓库的**，不要把它加进 `.gitignore`。
2. **不要修改 `themes/LoveIt` 内的任何文件**：主题是 submodule，改动会导致 submodule 脏状态并可能让构建/部署失败。所有定制都通过项目根目录的同名覆盖目录/文件实现（Hugo 的优先级：根 `layouts/`、`assets/` 高于主题）。
3. **Permalink 规则**：`[Permalinks] posts = ":filename"`，文章最终 URL 取决于 `content/posts/<目录名>/` 的目录名。仓库中存在两套命名并存：纯日期目录（如 `2026-11-02`）和语义化目录（如 `flutter-1`、`python-method`）。**目录名一旦发布就等于 URL，重命名等于改链接**。

### 定制层（根目录覆盖主题的部分）
- `layouts/_default/baseof.html`：覆写主题基础模板，注入 Mermaid 支持（`baseof.html:48-53`，通过 `{{ if .Store.Get "hasMermaid" }}` 条件加载 CDN 脚本）。
- `layouts/_default/_markup/render-codeblock-mermaid.html`：把 ```mermaid 代码块渲染为 `<pre class="mermaid">` 并设置 `hasMermaid` 标记，实现流程图按需加载。
- `assets/css/_custom.scss`：**绝大多数视觉定制的真实所在**（约 600 行）。已通过 `config.toml` 的 `customCSS` 启用。包含：自定义字体、正文排版（字距/行高/颜色）、TOC 目录样式、文章宽度响应式断点、引用块「」装饰引号（依赖 `assets/fonts/sspai_community_icon.ttf`），以及一整套 **GitHub 风格代码高亮配色**（浅色/深色两套 Chroma token 覆盖）。改样式优先改这里，而不是主题。
- `assets/css/_override.scss`：预留的主题变量覆盖入口（当前几乎为空）。

### 内容组织
- `content/posts/`：博客正文，每篇是一个 Page Bundle（`index.zh-cn.md` + 可选 `images/` 子目录存放配图，相对路径引用）。
- `content/about/`：独立页面（`me`、`soul`、`proust`、`40questions-2025` 等），挂在导航「about」下。
- `content/tags/`、`content/categories/`：预置的 `_index.zh-cn.md` 定义标签/分类的展示标题（如 `python`、`Linux`、`documentation`）。

### 文章前置参数（front matter）约定
参考 `archetypes/default.md` 与现有文章，标准字段为：
```yaml
---
title: "标题"
date: 2026-11-02T09:05:14+08:00
lastmod:
draft: false
tags: ["标签1", "标签2"]
categories: ["分类"]
description: ""
---
```
`draft: true` 的文章不会出现在正式构建中，需预览请用 `hugo server -D`。

### 主题功能开关（均在 config.toml）
- 评论系统：当前启用 **utterances**（`params.page.comment.utterances`，绑定 `leeyorke/leeyorke.github.io` 仓库 issue），其余（disqus/gitalk/valine/giscus 等）均关闭。
- 站内搜索：Lunr 本地搜索（`params.search.type = "lunr"`），依赖首页输出 JSON 索引（`outputs` 中 home 含 `JSON`）。
- 数学公式：KaTeX 已启用；`markup.goldmark.renderer.unsafe = true` 允许 Markdown 中直接写 HTML。
- 文章页额外输出 Markdown 原文（`outputs.page` 含 `MarkDown`，配合 `linkToMarkdown` 显示「阅读原文」）。

### 部署流程（人工，无 CI）
仓库无 `.github/workflows`，部署是**手动**的：本地 `hugo -e production` 构建到 `docs/` → `git add docs/` 提交 → 推送到 `origin master` → GitHub Pages 从 `master/docs` 发布到 `https://leeyorke.github.io`。

## 参考文档
- 主题 LoveIt 中文文档：<https://hugoloveit.com/zh-cn/theme-documentation-basics/>
- 项目自带的入门说明见 `README.md`（含建站、更新文章、 utterances 评论配置 FAQ）。
