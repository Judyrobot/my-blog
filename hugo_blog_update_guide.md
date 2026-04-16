# Hugo 主题更新与博客内容更新说明

本文档说明两个常见操作：

1. 如何更新 Hugo 博客主题  
2. 如何新增或修改博客文章并发布到线上

适用环境：

- 本地使用 Hugo 搭建博客
- 代码托管在 GitHub
- 站点部署在 Cloudflare Pages

---

## 一、项目基本结构

以当前博客项目为例，常见目录如下：

```text
my-blog/
├─ archetypes/
├─ content/
│  ├─ posts/
│  └─ about.md
├─ static/
│  └─ img/
├─ themes/
├─ hugo.toml
├─ .gitmodules
└─ .gitignore
```

重点说明：

- `content/posts/`：存放博客文章
- `content/about.md` 或 `content/about/index.md`：关于页
- `static/`：存放图片等静态资源
- `themes/`：存放 Hugo 主题
- `hugo.toml`：站点配置文件
- `.gitmodules`：如果主题通过 git submodule 引入，会有这个文件

---

## 二、Hugo 主题更新

### 2.1 当前主题在哪里配置

主题名称一般写在 `hugo.toml` 中，例如：

```toml
theme = "PaperMod"
```

如果要切换主题，本质上要做两件事：

1. 把目标主题下载安装到 `themes/` 目录
2. 把 `hugo.toml` 中的 `theme` 改成对应名称

---

### 2.2 安装一个新主题

以 PaperMod 为例，在项目根目录执行：

```bash
git submodule add https://github.com/adityatelange/hugo-PaperMod themes/PaperMod
```

如果是其他主题，只需要替换仓库地址和目录名。

例如：

```bash
git submodule add <主题仓库地址> themes/<主题目录名>
```

说明：

- `themes/PaperMod` 是本地主题目录
- 使用 `git submodule` 的好处是后续可以单独更新主题版本
- 执行后，`.gitmodules` 会自动更新

---

### 2.3 切换主题

打开 `hugo.toml`，把：

```toml
theme = "旧主题名"
```

改成：

```toml
theme = "PaperMod"
```

注意：

- 主题名称要和 `themes/` 目录名一致
- 大小写也要一致

---

### 2.4 本地预览主题效果

切换主题后，在项目根目录执行：

```bash
hugo server
```

然后浏览器打开：

```text
http://localhost:1313/
```

检查以下内容：

- 首页是否正常显示
- 菜单是否正常
- 文章列表是否正常
- 头像、图片、关于页是否正常
- 文章详情页目录是否正常

---

### 2.5 更新已有主题版本

如果主题是通过 `git submodule` 安装的，可以进入主题目录更新：

```bash
cd themes/PaperMod
git pull
cd ../..
```

然后提交主题更新：

```bash
git add .
git commit -m "update PaperMod theme"
git push
```

更稳的方式是：

```bash
git submodule update --remote --merge
```

然后再提交：

```bash
git add .
git commit -m "update theme submodule"
git push
```

说明：

- 更新主题前，建议先备份或先本地预览
- 某些主题新版本可能会改配置字段，更新后要重新检查 `hugo.toml`

---

### 2.6 删除旧主题（可选）

如果已经确认不再使用旧主题，可以后续清理旧主题目录。

例如删除旧主题目录后，再检查：

- `hugo.toml` 中是否还有旧主题配置
- `.gitmodules` 中是否还保留旧主题信息

如果你对 git submodule 不熟，建议先不要删旧主题，先确保新主题完全稳定。

---

## 三、博客内容更新

博客内容更新分为两种：

1. 修改已有文章
2. 新增一篇文章

---

### 3.1 修改已有文章

文章通常放在：

```text
content/posts/
```

例如：

```text
content/posts/my-first-post.md
content/posts/second-blog.md
```

如果要修改已有文章，直接编辑对应 `.md` 文件即可。

---

### 3.2 新增一篇文章

推荐使用 Hugo 命令创建：

```bash
hugo new content posts/my-new-post.md
```

创建后会生成一个新的 Markdown 文件，例如：

```text
content/posts/my-new-post.md
```

---

### 3.3 文章头信息写法

每篇文章开头通常都有 Front Matter，例如：

```md
---
title: "我的第二篇博客"
date: 2026-03-25T14:00:00+08:00
draft: false
tags:
  - Hugo
  - 博客搭建
categories:
  - 技术记录
---

这是正文内容。
```

字段说明：

- `title`：文章标题
- `date`：发布时间
- `draft`：是否草稿
- `tags`：标签
- `categories`：分类

注意：

- `draft: false` 才会正常发布
- `date` 不要写成未来时间，否则文章可能不显示
- 标签和分类有助于后续做归档和检索

---

### 3.4 本地预览文章

写完文章后，执行：

```bash
hugo server
```

如果文章还在草稿状态，也可以执行：

```bash
hugo server -D
```

说明：

- `-D` 表示把 draft 草稿文章也显示出来
- 如果本地能看到，线上通常就能看到
- 如果本地都看不到，先检查文章头信息

---

## 四、发布博客更新到线上

博客线上部署流程通常是：

本地修改 → 提交到 GitHub → Cloudflare Pages 自动部署

---

### 4.1 提交代码到 GitHub

在项目根目录执行：

```bash
git add .
git commit -m "update blog"
git push
```

如果是新增文章，也可以写得更明确，例如：

```bash
git add .
git commit -m "add new post"
git push
```

如果是切换主题：

```bash
git add .
git commit -m "switch theme to PaperMod"
git push
```

---

### 4.2 Cloudflare Pages 自动部署

如果 GitHub 仓库已经和 Cloudflare Pages 正确连接，那么每次 `git push` 后会自动触发部署。

可在 Cloudflare Pages 后台查看：

- 最新 deployment 是否成功
- 最新 commit 是否已同步
- 页面访问地址是否更新

默认访问地址通常类似：

```text
https://your-project-name.pages.dev
```

---

### 4.3 如何判断更新是否生效

有三种方式：

#### 方式 1：看 GitHub
确认最新修改已经 push 到 GitHub 仓库。

#### 方式 2：看 Cloudflare Pages
确认最新 deployment 状态为成功。

#### 方式 3：直接访问博客
刷新博客页面，看新文章、新主题或新内容是否已经显示。

如果没显示，可以尝试：

- 强制刷新浏览器
- 再看 Cloudflare 是否完成最新部署
- 检查文章是否为草稿或未来时间

---

## 五、常见问题排查

### 5.1 新文章没有显示

优先检查：

- `draft` 是否为 `false`
- `date` 是否写成未来时间
- Front Matter 格式是否正确
- 文章是否放在 `content/posts/` 下

---

### 5.2 主题切换后页面报错

优先检查：

- `theme` 名称是否正确
- 新主题目录是否存在
- `hugo.toml` 配置字段是否适配当前主题版本
- 头像、菜单、首页参数是否符合主题要求

---

### 5.3 图片显示异常

优先检查：

- 图片路径是否正确
- 文件名后缀是否一致（`.png` / `.jpg`）
- 图片是否已经被 `git add` 并 push
- 本地访问路径是否能直接打开

---

### 5.4 GitHub push 失败

常见原因：

- 网络不稳定
- HTTPS 连接 GitHub 失败
- 代理设置异常
- Git 凭证失效

可以先检查：

```bash
git remote -v
git config --global --get http.proxy
git config --global --get https.proxy
```

---

## 六、推荐的日常工作流

### 写新文章
```bash
hugo new content posts/new-post.md
```

### 本地预览
```bash
hugo server
```

### 提交并发布
```bash
git add .
git commit -m "add new post"
git push
```

### 查看线上效果
- GitHub 仓库
- Cloudflare Pages Deployments
- 博客正式地址

---

## 七、建议

1. 主题切换先本地预览，确认没问题再 push  
2. 每篇新文章都检查 `draft` 和 `date`  
3. 图片、头像等静态资源统一放在固定目录  
4. 每次改动都写清楚 commit message  
5. 先保证博客可用，再逐步美化样式和功能  

---

## 八、示例：一次完整的博客更新流程

### 新增文章
```bash
hugo new content posts/pdf-parser-notes.md
```

### 编辑文章
填写标题、时间、正文、标签、分类。

### 本地预览
```bash
hugo server
```

### 提交上线
```bash
git add .
git commit -m "add pdf parser notes"
git push
```

### 检查部署
- GitHub 是否有新 commit
- Cloudflare Pages 是否部署成功
- 博客页面是否出现新文章

---

## 九、示例：一次完整的主题切换流程

### 安装新主题
```bash
git submodule add https://github.com/adityatelange/hugo-PaperMod themes/PaperMod
```

### 修改配置
在 `hugo.toml` 中改：

```toml
theme = "PaperMod"
```

### 本地预览
```bash
hugo server
```

### 提交上线
```bash
git add .
git commit -m "switch theme to PaperMod"
git push
```

### 检查效果
- 首页风格是否变化
- 菜单是否正常
- 文章页目录是否正常
- Cloudflare 是否部署成功

---

以上就是 Hugo 主题更新和博客更新的基本操作说明。
