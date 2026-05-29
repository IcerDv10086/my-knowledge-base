# agentskills总结

## 概述

- 一种轻量级、开放的格式，用于为 AI 智能体添加新的能力和专业知识
- 核心结构是一个包含 `SKILL.md` 文件的文件夹，该文件包含**元数据（名称、描述）**和**执行特定任务的指令**
- 可捆绑脚本、参考资料、模板等资源

### 1. 背景

智能体的能力愈发强大，但是他们通常不具备开展可靠工作能力的专业知识。Skills 通过打包流程化的知识与公司、团队、个人特定的上下文成可移植、版本控制的文件夹，智能体可以动态加载。因此，这赋予了智能体:

- **领域专业知识**：捕获法律审查、数据分析、演示格式等专业知识
- **可重复工作流**：将多步骤任务转化为一致、可审计的流程
- **跨产品复用**：一次构建，可在任何兼容的智能体中使用

### 2. 工作原理

本质：渐进式披露。

1. **发现阶段**：智能体仅加载技能的名称和描述
2. **激活阶段**：当任务匹配时，读取完整的 `SKILL.md` 指令
3. **执行阶段**：遵循指令执行，按需运行代码或加载文件

## 文档结构

### 目录结构

```txt
    my-skill/
    ├── SKILL.md          # Required: metadata + instructions
    ├── scripts/          # Optional: executable code
    ├── references/       # Optional: documentation
    ├── assets/           # Optional: templates, resources
    └── ...               # Any additional files or directories
```

### skill.md 格式

前言部分：

| 字段 | 必填 | 约束条件 |
| --- | --- | --- |
| **name** | 是 | 最多 64 个字符。仅允许**小写字母**、**数字**和**连字符**。不能以连字符开头或结尾。 |
| **description** | 是 | 最多 1024 个字符。非空。描述技能的功能和使用场景。 |
| **license** | 否 | 许可证名称或对捆绑许可证文件的引用。越短越好。 |
| **compatibility** | 否 | 最多 500 个字符。指示环境要求（目标产品、系统包、网络访问等）。 |
| **metadata** | 否 | 用于附加元数据的任意键值映射。 |
| **allowed-tools** | 否 | 空格分隔的字符串，列出技能可以使用的预先批准的工具。（实验性） |

示例:

```markdown
---
name: pdf-processing
description: Extract PDF text, fill forms, merge files. Use when handling PDFs.
license: Apache-2.0
metadata:
  author: example-org
  version: "1.0"
---

```

主体部分：

没有格式限制，官方推荐的内容尽量包括以下部分：

- Step-by-step instructions：分步骤操作说明
- Examples of inputs and outputs：示例输入与输出
- Common edge cases：常见边界情况

### 其他目录

**scripts 目录:**

智能体可以主动运行的脚本文件。脚本应该：

- 独立或清晰地记录依赖关系
- 包括有用的错误消息
- 优雅地处理边界情况

支持地语言取决于代理的环境。常见选项通常包括Python、Bash、JavaScript等。

**references 目录:**

智能体可以主动查阅的补充文档文件：

- REFERENCES.md：包含技能的详细文档、示例、最佳实践等。
- FORMS.md：包含技能的表单、表单字段、表单验证规则等。
- 特定领域的文档：如法律、数据分析、演示格式等。

各类文档参考内容应该尽量精简。智能体按需加载，文体量越小，占用的上下文才越少。

**assets 目录:**

智能体可以访问的静态资源：

- 模板文件（doc模板、配置模板等）
- 图像文件
- 数据文件（查找表、原理图）

### 渐进式展示

| 组件 | 上下文大小 | 加载时机 |
| --- | --- | --- |
| Metadata | ~100 tokens | AI 启动后，会立即加载所有技能的 name 和 description 字段，用于快速识别和匹配技能。 |
| Instructions | < 5000 tokens（推荐） | 技能被激活时，加载完整的 SKILL.md 主体内容。 |
| Resources | 按需加载 | 文件（如 scripts/、references/、assets/ 中的文件）仅在需要时才加载。 |

保持主要SKILL.md文件在500行以下。必要时将内容拆分至多个文件至references目录。

### 文件引用示例

在skill.md引用references目录下的文件时，需要从当前skill根目录下的相对路径引用。

```markdown
See [the reference guide](references/REFERENCE.md) for details.

Run the extraction script:
scripts/extract.py
```

reference目录官方表示，避免进行嵌套。

### 验证

可以使用`skills-ref`技能参考库验证自定义的skills：

```bash
skills-ref validate ./my-skill
```

---

## 参考

1. [agent_skills](https://agentskills.io/home)
