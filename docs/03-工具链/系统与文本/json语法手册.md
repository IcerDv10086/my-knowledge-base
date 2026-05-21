# JSON 语法手册

JSON（JavaScript Object Notation）是一种**轻量级数据交换格式**，基于纯文本，易于人阅读和机器解析，广泛用于 API 接口、配置文件、数据存储。

常见文件：vscode的配置文件setting.json

---

## 基本语法规则

- **纯文本格式**：`.json` 文件本质是 UTF-8 纯文本。
- **键必须用双引号**：`"key": "value"`，单引号不合法。
- **值类型严格**：字符串用双引号，数字直接写，布尔值用 `true`/`false`，空值用 `null`。
- **不支持注释**：JSON 规范不允许任何形式的注释（`//` 或 `/* */`）。
- **末尾不能有逗号**：数组或对象最后一项后面不能加 `,`（Trailing Comma）。

!!! warning "最常见错误"
    键名忘记加双引号，或对象/数组最后一项后面多了逗号，会导致解析失败。

---

## 核心数据结构

### 值类型（标量）

| 类型 | 示例 | 说明 |
| --- | --- | --- |
| 字符串 | `"hello"` | 必须双引号 |
| 数字 | `42`、`3.14`、`-1` | 整数或浮点数 |
| 布尔值 | `true`、`false` | 全小写 |
| 空值 | `null` | 全小写 |

---

### 对象（Object）

键值对的无序集合，用 `{}` 包裹：

```json
{
  "name": "Alice",
  "age": 30,
  "active": true
}
```

---

### 数组（Array）

有序值的集合，用 `[]` 包裹，元素类型可混合：

```json
["apple", "banana", "cherry"]

[1, "two", true, null]
```

---

### 嵌套结构

对象和数组可以任意嵌套：

```json
{
  "user": {
    "name": "Alice",
    "tags": ["admin", "editor"],
    "address": {
      "city": "Beijing",
      "zip": "100000"
    }
  }
}
```

---

## Python json 库

Python 内置 `json` 模块，无需安装。

### 核心函数

| 函数 | 作用 |
| --- | --- |
| `json.dumps(obj)` | Python 对象 → JSON 字符串 |
| `json.loads(s)` | JSON 字符串 → Python 对象 |
| `json.dump(obj, f)` | Python 对象 → 写入文件 |
| `json.load(f)` | 从文件读取 → Python 对象 |

### 常用示例

```python
import json

# 序列化：Python 对象 → JSON 字符串
data = {"name": "Alice", "age": 30, "tags": ["admin"]}
json_str = json.dumps(data, ensure_ascii=False, indent=2)

# 反序列化：JSON 字符串 → Python 对象
obj = json.loads(json_str)
print(obj["name"])  # Alice

# 写入文件
with open("data.json", "w", encoding="utf-8") as f:
    json.dump(data, f, ensure_ascii=False, indent=2)

# 从文件读取
with open("data.json", "r", encoding="utf-8") as f:
    obj = json.load(f)
```

### 常用参数

- `ensure_ascii=False`：允许输出中文，否则中文会被转义为 `\uXXXX`。
- `indent=2`：格式化输出，缩进 2 空格，便于阅读。

### Python ↔ JSON 类型对照

| Python | JSON |
| --- | --- |
| `dict` | `{}` 对象 |
| `list`、`tuple` | `[]` 数组 |
| `str` | `"字符串"` |
| `int`、`float` | 数字 |
| `True`、`False` | `true`、`false` |
| `None` | `null` |

---
