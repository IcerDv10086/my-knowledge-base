# 知识库构建

核心思路：`本地写 Markdown + GitHub 自动构建发布网站`

## GitHub 自动构建

`GitHub Pages` 是 GitHub 提供的**免费静态网站托管服务**。它将 Git 仓库中的静态文件（HTML/CSS/JS/Markdown）自动转换成可访问的网站，无需购买服务器或配置 Nginx，提交代码即可上线。

1. 修改仓库设置，启用 `GitHub Actions` 作为 Pages 构建源
2. 配置工作流文件 `.github/workflows/pages.yml`，核心配置如下：
    * `on`：触发条件，设为 `push` 到主分支时执行
    * `permissions`：分配工作流最小必要权限
    * `jobs.build`：构建阶段，执行代码检出 → Python 环境配置 → MkDocs 依赖安装 → 站点构建
    * `jobs.deploy`：部署阶段，将构建产物发布到 GitHub Pages
3. 配置 `mkdocs.yml`，将所有知识库文件放在 `docs` 目录下，关键配置包括：
   * 站点信息（标题、URL、主题、语言等）
   * `docs_dir`：指定文档根目录为 `docs`
   * `nav`：定义左侧导航菜单及对应的文档文件路径

### 注意事项

1. 所有知识库文件必须放在 `main` 分支的 `/docs` 目录下
2. 构建和发布在 GitHub 服务器上执行。文件数量增加后，构建时间可能会变长。建议在本地预先安装 Python 和 MkDocs 环境，提前构建验证，再推送到仓库
3. `mkdocs.yml` 中配置的文件路径必须真实存在且准确。文档移动后，务必同步更新导航配置
4. 每次构建通常需要 30-60 秒。更新后在浏览器中使用 `Ctrl+F5` 强制刷新以查看最新内容（清除缓存）

### 快速自检命令

```bash
# 查看最近 5 次工作流运行
gh run list --workflow "Deploy MkDocs to GitHub Pages" --limit 5

# 查看指定运行的详细信息
gh run view <run-id>

# 查看最新一次运行的状态
gh run view --exit-status
```
