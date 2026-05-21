# git 相关工具使用

Git 本质上是一种**代码版本管理工具**。在 IC 工作环境中常见的是 SVN，但 Git 已是目前最主流的版本管理工具。

我日常主要使用 GitHub CLI 创建仓库，并通过 Git 进行版本维护。当前的使用思路是把它当作一个**可追溯的网盘**，用于存放整理后的笔记。

## gh (github cli)

> GitHub 本质是一个代码托管与协作平台。

### 登录授权

```bash
# 登录
gh auth login

# 查看状态
gh auth status

# 增加 delete_repo 权限
gh auth refresh -h github.com -s delete_repo
```

登录按提示选择即可。这种方式可以避免手动输入 Token，配置一次即可长期生效：

* GitHub.com（默认）
* 协议选 HTTPS
* 认证方式选 Login with a web browser
* 复制验证码 → 浏览器打开链接 → 授权登录

### 核心命令

#### 1.仓库管理

repo（repository）即仓库。在 Git 语境中，repo 是一个独立完整的项目目录，并保留完整的版本历史。

```bash
# 克隆仓库（自动配置 origin）
gh repo clone 用户名/仓库名

# 创建新仓库（交互式，选公开/私有、初始化README等）
gh repo create

# 在浏览器打开当前仓库页
gh repo view --web

# 列出当前用户的所有仓库
gh repo list

# Fork 别人的项目
gh repo fork <owner>/<repo> [--clone]

# 删除仓库，不加 --yes 会要求确认
gh repo delete <owner>/<repo> [--yes]
```

创建仓库时，我更偏好交互式方式，步骤清晰、上手更快。

#### 2.PR

PR（Pull Request）是 Git 协作中最核心的机制之一，用于将开发者的修改提交给项目维护者进行审核。目前个人场景暂时用不到。

### 实践思路记录

1. Fork 别人的仓库并设为 private，用于学习和测试，完成后删除。例如：`octocat/Hello-World`（官方测试项目）
2. Create 自己的仓库进行练习，完成后删除

## git

Git 相较于 SVN，属于**分布式**版本管理方式。

### 基础操作

```bash
# 1) 一次性配置身份与查看，永久生效
git config --global user.name "你的名字"
git config --global user.email "你的邮箱"
git config --global --list # 查看当前设置
git config --global core.quotepath false # 设置路径显示不转义中文，建议设置

# 2) 初始化仓库（在当前目录）
git init

# 3) 查看当前状态
git status

# 4) 添加文件到暂存区
git add 文件名
git add .

# 5) 提交（形成一个版本快照）
git commit -m "docs: 新增git工具使用笔记"

# 6) 查看提交历史（终端打印方式更优美）
git log --oneline --graph --decorate [-n 20]

# 7) 查看文件差异
git diff
git diff --staged

# 8) 撤销修改
# 丢弃工作区改动（未 add）
git restore 文件名
# 取消暂存（已 add 未 commit）
git restore --staged 文件名

# 9) 关联远端并推送
git remote add origin https://github.com/<user>/<repo>.git
git push -u origin main

# 10) 拉取更新
git pull --rebase

# 11) 临时保存现场
git stash
git stash pop

# 12) 删除文件
# 同时删除工作区文件 + 从版本库中移除
git rm 文件名

# 只从版本库中移除，保留工作区文件（常用于补加 .gitignore 后清除已跟踪的文件）
git rm --cached 文件名
git rm --cached -r 目录名

# 13) 重命名文件或目录
# git mv 相当于 mv + git rm + git add 三步合一
git mv 旧文件名 新文件名
git mv 旧目录名/ 新目录名/

# 提交重命名
git commit -m "chore: 重命名目录 旧目录名 → 新目录名"

# 14) list文件或目录
git ls-files
git ls-tree -r HEAD

# 15) 查看当前仓库体积
git count --objects -vH 
```

#### 常用工作流（知识库场景）

```bash
# 日常更新三步
git status
git add .
git commit -m "docs: 更新协议与标准笔记"

# 推送到远端
git push
```

#### 命名建议

提交信息建议用前缀，后续检索更方便：

* docs: 文档和笔记更新
* chore: 目录整理、配置调整
* fix: 修正错别字或错误命令

### 功能概览

Git 功能很多，个人笔记场景通常只会用到其中一部分，下面做简要说明。

| 功能 | 说明 | 个人是否常用 |
| --- | --- | --- |
| **branch** | 分支管理，隔离不同任务的开发线，互不干扰 | 偶尔用，主要在 main 上工作 |
| **merge / rebase** | 将分支合并回主线；rebase 可使提交历史更线性整洁 | 偶尔用 |
| **stash** | 临时保存未提交的改动，切换任务后还原 | 偶尔用 |
| **tag** | 给某个提交打标记（如 v1.0），常用于版本发布 | 偶尔用 |
| **submodule** | 在一个仓库中嵌入另一个仓库，管理依赖 | 暂不需要 |
| **PR（Pull Request）** | 向项目提交代码审查请求，团队协作核心流程（GitHub 概念） | 暂不需要 |
| **Issue** | GitHub 上的任务/Bug 跟踪系统，不属于 git 本身 | 暂不需要 |
| **Actions** | GitHub CI/CD 自动化流水线，如自动测试、自动部署 | 暂不需要 |

---

## 参考资料

* 《GitHub入门与实践》chapter: 1-5
