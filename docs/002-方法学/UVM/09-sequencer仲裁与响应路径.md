---
title: sequencer仲裁与响应路径
---

## sequencer仲裁与响应路径

> 备注：本文基于 UVM 1800.2-2020.3.1 的 `src/seq/uvm_sequencer_base.svh`、`uvm_sequencer.svh`、`uvm_sequence_base.svh`、`uvm_sequence.svh` 源码整理。

---

## 1. 本质：sequencer 在解决什么

sequencer 解决的是多 sequence 并发发包时，谁先拿到 driver 通道、如何回传响应、如何避免饥饿和死锁。

核心问题分三类：

1. 请求仲裁：多个 sequence 同时 `wait_for_grant`
2. 执行握手：`send_request` 与 driver `get_next_item/item_done`
3. 响应回路：response 如何回到正确 sequence

---

## 2. 关键类与数据结构

### 2.1 `uvm_sequencer_base`

关键队列/状态：

- `arb_sequence_q`：仲裁请求队列（REQ/LOCK）
- `lock_list`：当前锁拥有者
- `arb_completed[request_id]`：grant 完成标记
- `m_arbitration`：仲裁模式（FIFO/WEIGHTED/RANDOM/STRICT/USER）
- `m_wait_for_sequences_count`：每轮选择前等待次数，适配 stacked sequencer

### 2.2 `uvm_sequence_base`

- `start()` 承载 pre_start/body/post_start 生命周期
- priority、relevant、automatic phase objection 等行为都在这里

### 2.3 `uvm_sequencer #(REQ,RSP)` / `uvm_sequence #(REQ,RSP)`

- `item_done`、`put_response`、`get_response` 组成响应路径
- 内建请求/响应类型约束与 cast 检查

---

## 3. 关键调用链（按时间顺序）

### 3.1 请求路径（sequence -> driver）

1. sequence `wait_for_grant()` 提交请求到 `arb_sequence_q`
2. sequencer `m_select_sequence()` 循环选择：
   - `wait_for_sequences()`
   - `m_choose_next_request()`
3. `m_choose_next_request()` 先 `grant_queued_locks()`，再按模式挑选
4. 选中后 `m_set_arbitration_completed(request_id)`，sequence 获 grant
5. sequence 必须尽快 `send_request()`（允许 delta，不应消耗实时间）
6. driver 从 `get_next_item/get/peek/try_next_item` 取 item

### 3.2 仲裁策略细节

`m_choose_next_request()` 逻辑：

1. 过滤掉 zombie 进程（KILLED/FINISHED）
2. 只考虑 `is_blocked()==0` 且 `is_relevant()==1` 的请求
3. 按模式选择：
   - `UVM_SEQ_ARB_FIFO`
   - `UVM_SEQ_ARB_WEIGHTED`
   - `UVM_SEQ_ARB_RANDOM`
   - `UVM_SEQ_ARB_STRICT_FIFO/STRICT_RANDOM`
   - `UVM_SEQ_ARB_USER`（调用 `user_priority_arbitration`）

若 user arbitration 返回不在可选集合中的 index，会直接 fatal。

### 3.3 响应路径（driver -> sequence）

1. driver 完成 item 后调用 `item_done(rsp)` 或 `put(rsp)`
2. sequencer `item_done()`：
   - 清除 `get_next_item_called` 等状态
   - 检查 outstanding request 是否匹配
   - 可经 `seq_item_export.put_response(item)` 回送 response
3. sequence 侧：
   - `get_response()` 阻塞取响应
   - 或 `put_response()` 进入内部 response queue

响应队列默认深度有限，长期不取会溢出并报错。

---

## 4. 常见误区与排障手段

1. 误区：`wait_for_grant` 后可以任意延时再 `send_request`

实际应只允许 delta 级延时，否则 driver 端会出现阻塞/超时类问题。

1. 误区：`get_next_item` 可连续调用

`uvm_sequencer::get_next_item` 明确要求每次配对 `item_done`。

1. 误区：lock/grab 永远安全

若请求 lock 的进程被 kill，源码会报 `SEQLCKZMB` 并从队列移除以避免死锁。

1. 误区：默认仲裁一定公平

FIFO 只是先来先服务，不等于业务公平；需要按场景选 strict/weighted/user。

1. 排障建议

- 关注报错关键字：`TRY_NEXT_BLOCKED`、`SQRBADITMDN`、`SEQREQZMB`、`SEQLCKZMB`
- 对 stacked sequencer 调整 `wait_for_sequences_count`
- 用户仲裁模式务必保证返回 index 合法

---

## 5. 复习速记

1. 仲裁入口在 `arb_sequence_q`，核心选择函数是 `m_choose_next_request`
2. lock/grab 先于普通请求处理，且会影响可选集合
3. `wait_for_grant -> send_request -> item_done` 必须配对
4. response 回路堵塞常见根因是 sequence 未及时 `get_response`
5. `wait_for_sequences_count` 是 stacked sequencer 的关键调参项
