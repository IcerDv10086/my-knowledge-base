# mkdocs配置和使用

mkdocs 的功能是把 Markdown 文档快速转成干净、可部署的**静态网站**，主打「项目文档 / 知识库」场景。

## 网页前端基础

标准网页的结构基本是固定的，通常由以下部分构成：

* 头部 Header：网站标题、Logo、搜索框
* 导航栏 Nav：菜单、点了跳转到对应页面
* 侧边栏 Sidebar：MkDocs 左边那一列目录或者右边的那一栏
* 主内容区 Main：你写的 Markdown 正文、代码、文字
* 页脚 Footer：底部版权、备案、版本信息

静态网站通常指的是不包含数据库和后端服务器的网站，具有打开快、部署简单的特点。

## Mkdocs

mkdocs.yml文件中，最主要的设置分为：site、theme、nav。主体部分是我们编写的markdown文件内容。对于脚注可以使用theme.footer配置自定义。

### site

* `site_name`：网站标题，显示在页面头部和浏览器标签。
* `site_description`：网站描述，主要用于 SEO 和搜索引擎摘要。
* `site_url`：网站部署后的完整 URL（如用于生成 sitemap、社交分享等）。
* `site_author`：作者信息（可选）。
* `repo_url`：项目源码仓库地址（如 GitHub 链接，Material 主题下会自动显示在页面）。
* `repo_name`：仓库显示名称。

!!! note
    1. repo_url 和 repo_name 主要用于给用户提供“源码入口”按钮。
    2. site_author 一般不会在网页模板中直接显示，它主要用于网页元数据。

### theme

* `name`：主题名称（如 `material`、`readthedocs` 等），目前知识库类型最常用的是material。
* `language`：界面语言（如 `zh`、`en`）。
* `palette`：配色方案。
    * Material 主题支持多套配色（如亮色/暗色模式切换），每个 `scheme` 可以单独设置主色（primary）、强调色（accent）、切换按钮（toggle）等。常见用法是配置 `default`（亮色）和 `slate`（暗色）两套方案，网页右上角会出现切换按钮，用户可自由切换主题风格。
    * `scheme`：配色方案名（如 `default`、`slate`）。
    * `primary`：主色调（如 `indigo`、`blue`、`red` 等）。
    * `accent`：强调色。
    * `toggle`：切换按钮的图标和提示语。
* `font`：字体类型设置，分为text、code类型，但默认是Roboto/Roboto Mono，已经比较合适。
* `logo`：自定义 Logo 图片。
* `favicon`：浏览器标签的小图标。
* `features`：Material 主题下的功能开关列表。通过启用不同功能项，可以增强导航体验、搜索体验和代码阅读体验。
    * 常见导航类：
        * `navigation.tabs`：顶栏显示一级导航标签（横向 tab 样式）。
        * `navigation.sections`：侧边栏按章节分组显示，层级更清晰。
        * `navigation.top`：页面右下角出现“回到顶部”按钮。
        * `navigation.instant`：页面切换更快（类似单页应用的无刷新体验）。
        * `navigation.tracking`：滚动页面时，URL 会跟随当前标题更新锚点。
    * 常见搜索类：
        * `search.suggest`：搜索框输入时显示联想建议。
        * `search.highlight`：在结果页高亮关键词。
        * `search.share`：搜索后 URL 会带查询参数，方便分享结果链接。
    * 常见内容类：
        * `content.code.copy`：代码块显示复制按钮。
        * `content.code.annotate`：支持代码注释标记的高亮交互（配合注释语法使用）。
        * `toc.follow`：目录（TOC）跟随滚动高亮当前阅读位置。
    * 使用方式：`features` 是列表，按需启用即可，不需要一次开全。知识库场景一般先开 `navigation.tabs`、`navigation.sections`、`navigation.top`、`search.suggest`、`search.highlight`、`content.code.copy`。
    * 示例：

        ```yaml
        theme:
          name: material
          features:
            - navigation.tabs
            - navigation.sections
            - navigation.top
            - search.suggest
            - search.highlight
            - content.code.copy
        ```

* `custom_dir`：自定义主题目录（用于高级定制）。
* `footer`：自定义页脚内容（Material 主题可用）。

### nav

* 结构为列表，决定导航栏/侧边栏的显示顺序和分组。
* 每一项格式为 `显示名: 路径`，路径为 docs 目录下的 Markdown 文件相对路径。
* 可以嵌套分组（如 `- 组名: [子项1, 子项2]`）。
* 只会显示 nav 中列出的文件，未列出的不会出现在导航栏。
* 支持多级目录和多层嵌套，适合大型知识库。

### 其他

* `plugins`：站点功能插件列表。
    * 最常用是 `search`（站内搜索）。
    * 其他常见插件：
        * `git-revision-date-localized`：显示文档最后更新时间。
        * `minify`：压缩页面资源，减少体积（第三方插件）。
* `markdown_extensions`：增强 Markdown 语法能力。
    * 常用扩展：
        * `admonition`：提示框（note、warning 等）。
        * `tables`：表格语法。
        * `toc`：目录锚点和标题链接。
        * `footnotes`：脚注。
        * `pymdownx.superfences`：更强的代码块嵌套能力。
        * `pymdownx.highlight`：代码高亮增强。
        * `pymdownx.details`：可折叠功能块
* `exclude_docs`：排除不参与构建的文档路径。
    * 常见用途：
        * 排除草稿目录（如 `drafts/**`）。
        * 排除模板目录（如 `templates/**`）。
        * 排除构建产物目录（如 `site/**`，防止被再次处理）。

### 图表渲染

图表是理解复杂技术关系和系统结构的有效工具。MkDocs Material 支持通过 Mermaid 渲染流程图、时序图、状态图等常见图表。

Mermaid 图表的渲染通常依赖 `pymdownx.superfences` 扩展，通过自定义代码块 fence，将 `mermaid` 代码块识别并渲染为图表。如下所示：

```yaml
- pymdownx.superfences:
        custom_fences:
            - name: mermaid
                class: mermaid
                format: !!python/name:pymdownx.superfences.fence_code_format
```

完成配置后，即可在 Markdown 中直接编写 Mermaid 图表。

如需进一步自定义 Mermaid 的主题、配色或初始化参数，可新增 `docs/javascripts/mermaid.mjs`，并在 `mkdocs.yml` 中通过 `extra_javascript` 引入该脚本。

---

## 参考资料

* [mkdocs官方文档](https://squidfunk.github.io/mkdocs-material)
