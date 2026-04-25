# git 相关工具使用

本质是一种**管理代码版本的工具**，在工作中IC环境主要用到的是svn，但git是目前最流行的版本管理工具。

个人的日常使用主要涉及到 github cli 创建 git 仓库 和 使用 git 工具进行版本维护。因为目前的想法是把它当作一个**网盘**，存放本人的整理后的笔记。

## gh (github cli)

> github 是提供代码管理的一个平台。

### 登录授权

```bash
# 登录
gh auth login

# 查看状态
gh auth status

#增加delete_repo权限
gh auth refresh -h github.com -s delete_repo
```

登录按提示选，这种方式可以避免手动写token，一次配置永久生效：

* GitHub.com（默认）
* 协议选 HTTPS（新手最省事）
* 认证方式选 Login with a web browser
* 复制验证码 → 浏览器打开链接 → 授权登录

### 核心命令

#### 1.仓库管理

repo(repositoiry), 本意是仓库，储藏室。在工具来看，repo 是一个独立完整的项目文件夹，并有完整的版本历史记录。

```bash
# 克隆仓库（自动配置 origin）
gh repo clone 用户名/仓库名

# 创建新仓库（交互式，选公开/私有、初始化README等）
gh repo create

# 在浏览器打开当前仓库页
gh repo view --web

# 列出当前用户的所有仓库
gh repo list

# fork别人的项目
gh repo fork <owner>/<repo> [--clone]

# 删除仓库，不加yes会要求确认
gh repo delete <owner>/<repo> [--yes]
```

在创建仓库时，个人比较喜欢交互式方式，比较简单粗暴。

#### 2.PR

PR(pull requeset)，git工具实现代码协作的最核心功能。将代发开发者的修改，提交给项目维护者审核。暂时用不到这个功能。

### 实践思路记录

1. fork 别人的仓库，设为private，学习和测试，最终删掉。 example: `octocat/Hello-World` , 官方的测试项目
2. crete 自己的仓库，最终删掉

## git

git相较于svn，是一种**分散型**的管理方式。

### 基础操作

```bash
# 1) 一次性配置身份与查看，永久生效
git config --global user.name "你的名字"
git config --global user.email "你的邮箱"
git conifg --global --list

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
# 丢弃工作区改动（未add）
git restore 文件名
# 取消暂存（已add未commit）
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

git 功能众多，个人笔记场景只用到其中一小部分，以下做简要说明。

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
