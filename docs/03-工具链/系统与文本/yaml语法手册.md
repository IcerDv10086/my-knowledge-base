# YAML 语法手册

YAML（YAML Ain't Markup Language）是一种**注重可读性的数据序列化格式**，非常适合用于配置文件和数据描述。  
个人常用场景：`mkdocs.yml`（知识库配置）、`.github/workflows/pages.yml`（CI/CD 工作流）、自定义脚本的配置文件。

---

## 基本语法规则

- **大小写敏感**：`True` 和 `true` 不等价。
- **缩进表示层级**：只允许使用空格，**禁止 Tab**。缩进数量不限，但同一层级必须对齐（建议统一 2 空格）。
- **注释**：以 `#` 开头，可出现在行尾；`#` 后内容全部被忽略。
- **文档标识符**：`---` 表示文档开始，`...` 表示文档结束。同一文件可含多个文档。

```yaml
# 这是注释
---
name: Alice   # 行尾注释
age: 30
...
```

---

## 核心数据结构

YAML 主要支持三种结构：**标量（Scalars）**、**序列（Sequence）**、**对象（Mapping）**。

### 标量（基本值）

| 类型 | 写法示例 | 说明 |
| --- | --- | --- |
| 字符串 | `name: Alice` | 一般不需要引号 |
| 整数 | `age: 30` | 直接书写 |
| 浮点数 | `ratio: 3.14` | 直接书写 |
| 布尔值 | `active: true` | 推荐全小写 `true` / `false` |
| 空值 | `memo: null` | 或用 `~` |
| 日期 | `date: 2025-01-01` | `yyyy-MM-dd` 格式 |

**字符串引号规则：**  
通常不需要引号，以下情况需要加引号：

- 包含特殊字符：`# : , { } [ ] | > < & * ! % @ =`
- 字符串开头或结尾有空格
- 与 YAML 内置关键字同名：`true`、`false`、`null`、纯数字

```yaml
a: hello world         # 不需要引号
b: "yes"               # 需要引号（yes 会被解析为 true）
c: "name: Alice"       # 含 : 需要引号
d: '带 # 号的内容'     # 含 # 需要引号
```

**多行字符串：**

```yaml
# | 字面块：保留所有换行和格式
description: |
  第一行
  第二行
  第三行

# > 折叠块：换行替换为空格，变成一行
summary: >
  这是一段很长的
  描述文字内容
```

---

### 序列（列表）

```yaml
# 多行写法（推荐）
fruits:
  - Apple
  - Orange
  - Strawberry

# 单行写法
fruits: [Apple, Orange, Strawberry]
```

---

### 对象（字典/映射）

```yaml
# 多行写法（推荐）
person:
  name: John Doe
  age: 30
  married: true

# 单行写法
person: {name: John Doe, age: 30, married: true}
```

---

### 嵌套结构

序列和对象可以相互嵌套，实际配置文件中非常常见：

```yaml
# 对象列表（列表中每个元素是对象）
servers:
  - host: server1.example.com
    port: 8080
  - host: server2.example.com
    port: 9090

# 对象中包含列表
config:
  allowed_ips:
    - 192.168.1.1
    - 10.0.0.1
```

---

## 锚点与别名（避免重复配置）

YAML 支持用 `&` 定义锚点，用 `*` 引用，`<<` 合并键值，适合在同一文件中复用配置：

```yaml
defaults: &defaults     # 定义锚点 defaults
  timeout: 30
  retries: 3

production:
  <<: *defaults         # 合并 defaults 的所有内容
  host: prod.example.com

staging:
  <<: *defaults         # 同样合并
  host: staging.example.com
```

---

## PyYAML（Python 读写 YAML）

当需要用脚本批量处理 YAML 配置文件时使用：

```bash
pip install pyyaml
```

```python
import yaml

# 读取 YAML 文件
with open('config.yaml', 'r', encoding='utf-8') as f:
    data = yaml.safe_load(f)    # 推荐 safe_load，比 load 更安全
print(data)

# 写入 YAML 文件
data = {'name': '李四', 'age': 30, 'active': False}
with open('output.yaml', 'w', encoding='utf-8') as f:
    yaml.dump(data, f,
              default_flow_style=False,  # 使用多行格式
              allow_unicode=True,        # 支持中文
              sort_keys=False)           # 保持键的原始顺序
```

---

## 参考资料

- [YAML 官方规范](https://yaml.org/)
- [原博客园总结](https://www.cnblogs.com/cmyxjcc/p/19093897)
