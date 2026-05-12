# sequence 机制

> 本文聚焦 UVM sequence / sequencer 的源码结构与使用方式，目标是把“事务、序列、仲裁、响应”四条线串起来，避免只记接口名不理解执行过程。

## 1. 先理解几个对象

### 1.1 事务与序列项

`uvm_transaction` 是事务根类，但工程上更常用的是 `uvm_sequence_item` 作为用户事务基类。

继承链可简化为：

```txt
uvm_object
  -> uvm_transaction
  -> uvm_sequence_item
  -> uvm_sequence_base
  -> uvm_sequence #(REQ, RSP)
```

事务的本质是一次完整交互，而不是单个时钟周期上的信号切换。

### 1.2 uvm_transaction 的作用

它提供事务生命周期相关能力：

- 时间戳：`accept_time`、`begin_time`、`end_time`
- 生命周期方法：`accept_tr()`、`begin_tr()`、`end_tr()`
- 回调：`do_accept_tr()`、`do_begin_tr()`、`do_end_tr()`

这些能力更多服务于事务记录与调试，不是 sequence 机制本身的核心。

## 2. uvm_sequence_item 做了什么

`uvm_sequence_item` 在事务基础上补齐序列上下文：

- `m_sequence_id`：序列 ID。
- `m_sequencer`：当前 item 所属 sequencer。
- `m_parent_sequence`：父序列上下文。

它增加的重点是：让一个事务知道自己来自哪里、要交给谁。

## 3. uvm_sequence_base 的职责

`uvm_sequence_base` 是所有 sequence 的真正执行基类，重点是“运行、控制、响应、锁”。

### 3.1 主要状态

- 序列状态：`UVM_CREATED`、`UVM_PRE_START`、`UVM_BODY`、`UVM_ENDED`、`UVM_FINISHED`、`UVM_STOPPED` 等。
- 优先级：`m_priority`。
- 响应队列：`response_queue`、`response_queue_depth`。
- 子序列管理：`children_array`。

### 3.2 核心生命周期

- `start()`：启动序列。
- `pre_start()` / `post_start()`：启动前后回调。
- `pre_body()` / `post_body()`：body 前后回调。
- `body()`：用户必须实现的主体任务。

### 3.3 item 发送主线

常见 sequence item 流程：

1. `create_item()` 创建 item。
2. `start_item()` 发起请求。
3. `wait_for_grant()` 等待仲裁授权。
4. 随机化 item。
5. `send_request()` 送出请求。
6. `finish_item()` 完成。
7. `wait_for_item_done()` 等待完成通知。

### 3.4 锁与抢占

- `lock()`：普通锁。
- `grab()`：抢占式锁。
- `unlock()` / `ungrab()`：释放锁。

这些接口的意义是控制多个 sequence 在同一个 sequencer 上的访问顺序。

### 3.5 响应处理

- `put_response()`：放入响应。
- `get_base_response()`：取出响应。
- `response_handler()`：自定义响应处理逻辑。

当协议有 request/response 语义时，这部分特别重要。

## 4. 参数化 sequence

`uvm_sequence #(REQ, RSP)` 是参数化子类，通常用于用户自定义 sequence。

常见参数：

- `REQ`：请求类型。
- `RSP`：响应类型。
- `sequencer_t`：参数化 sequencer 类型。

它主要解决两件事：

1. 类型安全。
2. 让请求和响应保持一致的事务类型语义。

## 5. sequencer 的角色

sequencer 的职责可以概括为三件事：

1. 管理等待中的 sequence。
2. 做仲裁。
3. 把授权后的 item 送给 driver。

继承链可简化为：

```txt
uvm_object
  -> uvm_component
  -> uvm_sequencer_base
  -> uvm_sequencer_param_base #(REQ, RSP)
  -> uvm_sequencer #(REQ, RSP)
```

## 6. 工作流怎么串起来

一个完整流程可以记成：

1. sequence 启动。
2. sequence 请求 sequencer 授权。
3. sequencer 根据仲裁策略决定谁先发。
4. driver 执行 item。
5. response 返回给 sequence。

这条链路中：

- sequence 负责“我想发什么”。
- sequencer 负责“谁先发”。
- driver 负责“真正怎么发”。

## 7. 常见实践建议

- 用户事务优先继承 `uvm_sequence_item`。
- 所有 item 发送尽量走标准 `start_item/finish_item` 流程。
- 对有响应的协议，提前规划 response queue 深度与 handler。
- 对复杂流水线，使用 `lock/grab` 时要明确释放边界，避免序列饿死。

## 8. 小结

sequence / sequencer 的核心不是“接口很多”，而是“三个动作”：创建 item、申请授权、完成驱动。把这三个动作和 transaction 生命周期对应起来，后面的仲裁、抢占、响应处理就容易统一理解。
- 管理默认序列 ：支持按阶段启动默认序列
- 提供序列执行环境 ：为序列提供执行上下文和资源管理

### uvm_sequencer_param_base #(REQ, RSP)

uvm_sequencer_param_base 继承自 uvm_sequencer_base ，在其基础上增加了**类型参数化和响应处理**的功能。它是连接具体序列和驱动的桥梁，确保类型安全的同时，提供了更丰富的功能。

> uvm_sequencer_base、uvm_sequencer_param_base本质上作为一个中间层，而不是直接与驱动交互，因此我们一般不会直接使用。

### uvm_sequencer #(REQ, RSP)

uvm_sequencer 在 uvm_sequencer_param_base 的基础上，增加了与驱动交互的标准接口，使其成为一个完整的、可直接使用的 sequencer 实现。

增加的内容如下：

- **组件注册**：使用 `uvm_component_param_utils` 宏，支持工厂机制创建和配置
- **驱动交互接口**：
    - `seq_item_export`：与驱动的 `seq_item_port` 连接的导出端口
    - 标准方法：`get_next_item`、`try_next_item`、`item_done`、`put`、`get`、`peek`
- **序列管理**：`stop_sequences()` 方法，用于停止所有序列并重置状态
- **内部方法**：`item_done_trigger`、`item_done_get_trigger_data`、`m_find_number_driver_connections`
- **类型定义**：`uvm_virtual_sequencer` 类型，用于协调多个 sequencer

## sequence->sequencer->driver 链

### sequencer 与 driver 交互

具体的交互如下图所示，driver 和 sequencer 之间通过端口进行连接。
![sequence与driver的链接](https://img2024.cnblogs.com/blog/2184461/202603/2184461-20260311234338853-1160143276.png)

uvm_seq_item_pull_*是一种特殊的端口，具有更多的可操作方法。从一般端口的视角看， driver是控制端口的发起者，sequencer是接收者。

### sequence 实际发送流程

sequence按照sequence嵌套可以分为两种情况：

- **无子序列**
- **有子序列**

#### 无子序列

无子序列的发送流程如下：

![sequence_item flow](https://img2024.cnblogs.com/blog/2184461/202603/2184461-20260311234339430-786737834.png)

仿真的起点：

1. sequence 的 body 中，调用 `start_item()` 方法
2. driver 的 `seq_item_port` 端口，调用 `get_next_item()` 方法，获取 sequence_item
3. sequence_item 被 driver 处理后，调用 `item_done()` 方法，通知 sequencer 序列项处理完成
4. sequencer 收到通知后，路由到对应sequence，返回 rsp

#### 有子序列

sequencer 侧本质不调度序列，调度的对象永远都是序列项（sequence items）

sequence 侧的顺序如下,本质可以使用sequence嵌套链的展开：

- 调用 `uvm_do(subsequence) 宏（假定使用uvm_do 发送子序列）
- 调用 pre_do() 任务， 并设置 is_item = 0
- 调用 mid_do()
- 触发 subsequence.started 事件
- 调用 subsequence.body()
- 触发 subsequence.ended 事件
- 调用 post_do()
- 子序列执行完成

![subsequence flow](https://img2024.cnblogs.com/blog/2184461/202603/2184461-20260311234339636-363148139.png)
