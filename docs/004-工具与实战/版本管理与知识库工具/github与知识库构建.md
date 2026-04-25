# 知识库构建

核心思路： `本地写markdown + GitHub 自动构建发布页面`

## Github 自动构建

`GitHub Pages` 是 GitHub 自带的**免费静态网站**托管服务。它把 Git 仓库里的静态文件（HTML/CSS/JS/Markdown）直接变成可访问的网站，不用买服务器、不用配置 Nginx，提交代码就自动上线。

1. 修改仓库的pages设置为 `GitHub Actions`
2. 创建并修改文件 `.github/workflows/pages.yml`，设置构建的工作流，文件核心结构如下：
    * on : 触发条件，设置为 `push`
    * permissions : 设置工作流的最小权限
    * jobs.build : 构建阶段，步骤为 `checkout repo -> setup_python -> setup mkdocs mkdocs-material -> build & update`
    * jobs.deploy : 发布阶段
3. 创建 `.mkdocs.yml`，所有知识库文件都放置docs目录下，文件的核心结构如下：
   * 网站定义（网址、主题、语言等）
   * docs_dir : 根目录设置，
   * nav : 定义左侧导航栏，并对应上文件

Note：

1. 文件需要放置在main分支下的/docs目录下
2. 构建与发布操作，本质上是在github服务器上执行的，这也意味着后续如果文件过多，可能需要的时间会加长；push前可以本地安装python和对应库文件，提前构建下，本地查看内容和效果
3. mkdocs.yml 中使用的文件，其路径必须存在且正确，且文档移动后需要同步修改目录

### 快速自检命令

```bash
# 查看最近工作流状态
gh run list --workflow "Deploy MkDocs to GitHub Pages" --limit 5

# 查看最新一次详情
gh run view <run-id>
```
