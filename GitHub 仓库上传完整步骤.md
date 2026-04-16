# GitHub 仓库上传完整步骤（MD版）

本文档包含 **命令行（通用）** 和 **图形化工具（简化）** 两种上传方法，适配 Windows/Mac/Linux 系统，新手可直接复制命令执行，避开常见坑。

# 一、前期准备

## 1. 安装 Git

- Windows：下载 [Git for Windows](https://git-scm.com/)，安装时勾选「Git Bash Here」（方便右键打开命令行）。

- Mac：终端执行 `brew install git`，或从官网下载安装。

- Linux：终端执行 `sudo apt install git`（Ubuntu/Debian）或 `sudo yum install git`（CentOS）。

验证安装成功（终端输入）：

```bash
git --version
```

## 2. 配置 Git 账户（与 GitHub 一致）

终端输入以下命令，替换为自己的 GitHub 用户名和邮箱（用于提交记录关联）：

```bash
git config --global user.name "你的GitHub用户名"
git config --global user.email "你的GitHub邮箱"
```

# 二、在 GitHub 官网创建远程仓库

1. 登录 GitHub（[https://github.com/](https://github.com/)），点击右上角「+」→「New repository」。

2. 填写仓库基本信息：


    - Repository name：仓库名（建议和本地项目文件夹同名，避免混乱）。

    - Description：可选，填写项目描述（如「个人学习项目，用于测试GitHub上传」）。

    - Public / Private：公开（所有人可见）/ 私有（仅自己或指定人可见），根据需求选择。

    - **关键提醒**：不要勾选「Add a README file」「Add .gitignore」「Choose a license」，否则会导致本地仓库与远程仓库冲突，后续推送失败。

3. 点击「Create repository」，创建成功后，复制页面中的「HTTPS 仓库地址」（格式：`https://github.com/用户名/仓库名.git`），后续会用到。

# 三、命令行上传（通用方法，推荐）

打开 Git Bash（Windows 右键项目文件夹→「Git Bash Here」）或终端（Mac/Linux），按以下步骤执行命令（复制即可，替换对应路径/地址）。

## 步骤1：进入本地项目根目录

替换命令中的「你的项目路径」，进入项目文件夹（示例）：

```bash
# Windows 示例（比如项目在D盘的test文件夹）
cd /d/TestProject

# Mac/Linux 示例（比如项目在文档文件夹）
cd ~/Documents/TestProject
```

## 步骤2：初始化本地 Git 仓库

仅第一次上传时执行，初始化后会在项目文件夹生成隐藏的 `.git` 文件夹（管理版本记录）：

```bash
git init
```

## 步骤3：将本地文件添加到暂存区

「.」表示添加项目中所有文件，若只想添加单个文件，替换为文件名（如 `git add README.md`）：

```bash
git add .
```

## 步骤4：提交文件到本地仓库

替换引号内的「提交说明」，描述本次上传的内容（如「首次提交：项目初始化，包含基础代码」），方便后续查看版本：

```bash
git commit -m "首次提交：项目初始化"
```

## 步骤5：关联远程仓库

替换为第二步复制的「GitHub 仓库地址」，将本地仓库与远程仓库关联：

```bash
git remote add origin https://github.com/用户名/仓库名.git
```

## 步骤6：重命名主分支（适配 GitHub 默认）

GitHub 目前默认主分支为 `main`，执行以下命令将本地分支重命名为`main`（避免分支名称不匹配）：

```bash
git branch -M main
```

## 步骤7：推送本地文件到 GitHub

第一次推送需加 `-u`，绑定本地 `main` 分支与远程 `main` 分支，后续推送可简化命令：

```bash
git push -u origin main
```

# 四、身份验证（关键步骤，必看）

GitHub 已禁用密码登录，推送时提示「输入用户名/密码」，注意：

- 用户名：输入你的 GitHub 用户名（或邮箱）。

- 密码：**不是 GitHub 登录密码**，需输入「个人访问令牌（Personal Access Token）」。

## 个人访问令牌生成方法

1. 登录 GitHub → 点击右上角头像 →「Settings」→「Developer settings」→「Personal access tokens」→「Tokens (classic)」。

2. 点击「Generate new token (classic)」，填写信息：
        

    - Note：令牌备注（如「本地推送令牌」），方便后续识别。

    - Expiration：令牌有效期（建议选「No expiration」，避免频繁重新生成）。

    - Scopes：勾选「repo」（全选 repo 下的子选项），其他无需勾选。

3. 点击「Generate token」，生成后**立即复制令牌**（仅显示一次，刷新页面后会消失，建议保存到记事本）。

推送时粘贴令牌，即可完成验证，成功上传。

# 五、后续修改上传（常用命令）

后续修改本地项目后，无需重新初始化和关联，只需执行以下3步即可推送更新：

```bash
# 1. 添加修改后的所有文件到暂存区
git add .

# 2. 提交修改（说明修改内容，如「修复登录bug，优化页面样式」）
git commit -m "修改说明"

# 3. 推送至远程仓库
git push
```

# 六、常见问题及解决方案

## 问题1：fatal: remote origin already exists（远程仓库已存在）

原因：本地已关联过其他远程仓库，解决方案：先删除原有关联，再重新关联：

```bash
git remote remove origin
# 再执行关联命令
git remote add origin https://github.com/用户名/仓库名.git
```

## 问题2：error: src refspec main does not match any（分支不匹配）

原因：未执行「git add .」和「git commit」，本地仓库无提交记录，解决方案：先提交本地文件，再推送：

```bash
git add .
git commit -m "首次提交"
git branch -M main
git push -u origin main
```

## 问题3：每次推送都要输入令牌

解决方案：设置 Git 凭证缓存，输入一次令牌后自动保存：

```bash
git config --global credential.helper store
```

# 七、图形化工具上传（不想用命令行）

## 方法1：GitHub Desktop（最简单）

1. 下载 [GitHub Desktop](https://desktop.github.com/)，安装后登录 GitHub 账户。

2. 点击「File」→「Add existing repository」，选择本地项目文件夹。

3. 填写「Commit message」（提交说明），点击「Commit to main」。

4. 点击「Publish repository」，选择对应的远程仓库，点击「Publish」即可完成上传。

## 方法2：VS Code（适合开发者）

1. 打开 VS Code，加载本地项目文件夹。

2. 点击左侧「源代码管理」图标（Ctrl+Shift+G），点击「初始化仓库」。

3. 输入提交说明，点击「√」提交到本地仓库。

4. 点击「推送」按钮，输入 GitHub 用户名和个人访问令牌，即可推送至远程仓库。

# 八、补充说明

- 若项目有不需要上传的文件（如依赖包、日志文件），可创建 `.gitignore` 文件，写入不需要上传的文件/文件夹名称，Git 会自动忽略这些文件。

- 若推送失败，可先执行`git pull origin main`（拉取远程仓库最新内容），解决冲突后再推送。
> （注：文档部分内容可能由 AI 生成）