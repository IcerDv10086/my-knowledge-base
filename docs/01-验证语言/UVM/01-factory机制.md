---
title: factory机制
---

# UVM factory 机制

## 1. factory 是什么

factory 是 UVM 中负责“类型注册、重载和创建”的核心机制。它解决的不是“怎么 new 一个对象”，而是“怎么在不改调用点的情况下替换对象类型”。

核心价值：

- 统一管理组件和对象的创建入口。
- 支持测试场景按需替换默认实现。
- 保持层次化树结构与命名一致。

## 2. 核心对象

- `uvm_factory`：抽象接口。
- `uvm_default_factory`：默认实现。
- `uvm_object_wrapper`：类型代理，负责把类型注册和创建解耦。

UVM 通过代理模式把“类型本身”和“类型如何创建”分开处理。

## 3. 基本能力

factory 主要提供四类能力：

1. 类型注册：把组件/对象类型登记到工厂。
2. 类型重载：把某个类型替换成另一个类型。
3. 实例重载：只替换特定路径下的实例。
4. 类型查找：根据名称或 wrapper 找到已注册类型。

## 4. 注册流程

类型注册通常由宏完成：

- 组件使用 `uvm_component_utils(T)`。
- 对象使用 `uvm_object_utils(T)`。

宏展开后会生成：

- `type_id`。
- `get_type()`。
- `get_object_type()`。
- `create_component()` / `create_object()`。
- `get_type_name()`。

注册的本质是：把类型代理放入 factory 的内部表中，后续通过代理来创建实例。

## 5. 实例创建与重载

创建实例时，factory 的判断顺序通常是：

1. 查实例重载。
2. 查类型重载。
3. 决定最终类型代理。
4. 调用代理创建实例。

常用接口：

- `set_type_override_by_type`
- `set_type_override_by_name`
- `set_inst_override_by_type`
- `set_inst_override_by_name`

## 6. 需要注意的三个条件

override 要真正生效，通常要满足：

1. 原始类和替换类都已注册到 factory。
2. 原始类必须通过 `type_id::create()` 创建。
3. override 必须发生在对象创建之前。

## 7. 工程理解

可以把 factory 理解成“验证环境的可插拔创建层”：

- 默认测试用标准类。
- 特定用例替换成带监控、带错误注入或带简化逻辑的派生类。
- 不需要改动主流程代码。

## 8. 与树形结构的关系

factory 负责“创建什么类型”，`uvm_component` 负责“把创建出来的对象挂到哪棵树上”。

因此两者经常一起出现，但职责不同：

- factory 管类型。
- component 管层次。

## 9. 小结

factory 是 UVM 灵活性的来源之一。只要把“注册、代理、重载、创建”四个动作理顺，后续理解 `sequence`、`driver`、`env` 的派生策略会轻松很多。

#### 宏注册的实现细节

以 `uvm_component_utils(T)` 宏为例，其展开逻辑如下：

```systemverilog
// uvm_component_utils宏的实现
`define uvm_component_utils(T) \
   `m_uvm_component_registry_internal(T,T) \
   `uvm_type_name_decl(`"T") \

// 其中m_uvm_component_registry_internal宏展开为：
`define m_uvm_component_registry_internal(T,S) \
  typedef uvm_component_registry#(T,`"S") type_id; \
  static function type_id get_type(); \
    return type_id::get(); \
  endfunction \
  virtual function uvm_object_wrapper get_object_type(); \
    return type_id::get(); \
  endfunction \
  function uvm_component create_component(string name, uvm_component parent); \
    uvm_component tmp; \
    tmp = type_id::create(name, parent); \
    return tmp; \
  endfunction

// uvm_type_name_decl宏展开为：
`define uvm_type_name_decl(N) \
  const static string type_name = N; \
  virtual function string get_type_name(); \
    return type_name; \
  endfunction
```

当在组件类中使用`uvm_component_utils(my_component)`时，宏会展开为：

```systemverilog
class my_component extends uvm_component;
  // 宏展开后的代码
  typedef uvm_component_registry#(my_component, "my_component") type_id;
  static function type_id get_type();
    return type_id::get();
  endfunction
  virtual function uvm_object_wrapper get_object_type();
    return type_id::get();
  endfunction
  function uvm_component create_component(string name, uvm_component parent);
    uvm_component tmp;
    tmp = type_id::create(name, parent);
    return tmp;
  endfunction
  const static string type_name = "my_component";
  virtual function string get_type_name();
    return type_name;
  endfunction
  
  function new(string name, uvm_component parent);
    super.new(name, parent);
  endfunction
endclass
```

### 6. 注册与创建的完整流程

1. **编译时**：
   - 组件/对象类使用宏注册，生成类型代理相关代码
   - 类型代理的静态初始化触发延迟注册

2. **初始化时**：
   - UVM核心初始化，处理延迟注册队列
   - 类型代理调用`initialize`方法，完成向factory的注册

3. **运行时**：
   - 通过`type_id::create()`或factory方法创建实例
   - factory根据重载规则确定最终类型
   - 类型代理创建并返回实例

### 7. 利用factory的重载的特点与方法

- factory机制支持组件和对象类型的重载，允许在**运行时**替换默认类型为其他类型，实现灵活的测试配置
- 重载分为两种类型：
    - **类型重载**：对所有该类型的实例进行替换
    - **实例重载**：仅对特定路径的实例进行替换
- 重载过程是在创建实例之前通过factory的方法设置，当调用`create`方法时，factory会根据重载规则创建相应的类型实例

- 内置重载方法:
    - **类型重载方法**:
        - `set_type_override_by_type(original_type, override_type, replace)`
        - `set_type_override_by_name(original_type_name, override_type_name, replace)`
    - **实例重载方法**:
        - `set_inst_override_by_type(relative_inst_path, original_type, override_type)`
        - `set_inst_override_by_name(relative_inst_path, original_type_name, override_type_name)`

- 命令行重载:
    - 可以在命令行中使用`-uvm_set_type_override`和`-uvm_set_inst_override`选项来设置类型和实例重载

- 覆盖优先级
    - 类型重载：根据注册顺序，后注册的类型会覆盖先注册的类型
    - 实例重载：根据路径顺序，后设置的实例重载会覆盖先设置的实例重载
    - 命令行重载：在命令行中使用`-uvm_set_type_override`和`-uvm_set_inst_override`选项来设置类型和实例重载，命令行设置的重载会覆盖程序中设置的重载

- **override机制必须满足的三个条件**:
    - 原始类（被覆盖的类）和覆盖类（新的派生类）都必须使用uvm_component_utils或uvm_object_utils宏在工厂中注册
    - 原始类必须通过工厂的type_id::create()方法创建
    - 覆盖操作（调用set_type_override或set_inst_override）必须在目标对象被创建之前进行

## UVM的树形结构实现

UVM的树形结构本质是通过`uvm_component`的**构造函数**实现的，具体机制如下：

uvm_component具有如下变量和方法：

- 变量
  1. m_parent：指向父组件的指针
  2. m_children：关联数组，存储子组件的指针
  3. m_children_by_handle：关联数组，根据组件句柄存储子组件的指针

- 方法
  1. new：构造函数，用于创建组件实例
  2. m_add_child：将子组件添加到当前组件的子组件列表中

### 1. 构造函数的核心逻辑

`uvm_component`的构造函数是树形结构形成的关键，其核心代码如下：

```systemverilog
function uvm_component::new (string name, uvm_component parent);
  uvm_root top;
  uvm_coreservice_t cs;
  
  super.new(name);
  
  // 获取核心服务和根节点
  cs = uvm_coreservice_t::get();
  top = cs.get_root();  // 获取uvm_root实例
  
  // 当parent为null时，自动设置为uvm_root
  if (parent == null) begin
    parent = top;
  end
  
  // 检查子组件名称唯一性
  if (parent.has_child(name) && this != parent.get_child(name)) begin
    // 处理名称冲突错误
  end
  
  // 设置父组件并添加到父组件的子列表中
  m_parent = parent;
  set_name(name);
  if (!m_parent.m_add_child(this)) begin
    m_parent = null;
  end
  
  // 继承父组件的域
  m_domain = parent.m_domain;
  
  // 其他初始化...
endfunction
```

### 2. 树形结构的形成过程

1. **根节点初始化**：
   - `uvm_root`是一个特殊的`uvm_component`，作为整个组件树的根节点
   - 通过`uvm_coreservice_t::get().get_root()`获取唯一的根节点实例
   - 核心代码:

        ```systemverilog
         static local uvm_root m_inst;  // 静态局部变量，存储**单例实例**
         
         function uvm_root uvm_root::new();
           super.new("**top**", null);
           // 防止创建多个根实例
           if (m_inst != null) begin
             `uvm_fatal_context("UVM/ROOT/MULTI",
             "Attempting to construct multiple roots",
             m_inst)
             return;
           end
           m_inst = this;  // 将当前实例赋值给静态变量
           // 其他初始化...
         endfunction
        ```

2. **父组件设置**：
   - 当创建组件时不指定父组件（设为`null`），系统会自动将其父组件设置为`uvm_root`
   - 这样确保了所有组件最终都归属于同一个根节点下
   - 通常，uvm_test的parent是null，其他组件的parent都是uvm_test

3. **子组件添加**：
   - 通过`m_parent.m_add_child(this)`将**当前组件添加到父组件的子组件列表**中

### 3. 树形结构的特点

- **唯一性**：每个组件在其父组件下都有唯一的名称
- **层次化**：通过点号分隔的层次化名称表示组件在树中的位置
- **完整性**：所有组件都属于同一个组件树，根节点是`uvm_root`
- **可遍历性**：通过`get_first_child`、`get_next_child`等方法可以遍历组件树
