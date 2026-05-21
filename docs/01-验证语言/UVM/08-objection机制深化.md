---
title: objection机制深化
---

## objection机制深化

> 备注：本文基于 UVM 1800.2-2020.3.1 的 `src/base/uvm_objection.svh` 源码整理，重点聚焦传播、收敛（drain/all_dropped）和排障。

---

## 1. 本质：objection 在解决什么

objection 不是“延时器”，而是运行期阶段结束的分布式计数协议。

- raise：声明“我还有工作未完成”
- drop：声明“我这部分工作完成了”
- 当某层 total 归零后，进入 drain/all_dropped，再决定是否继续向父层传播 drop

一句话：

- objection 的本质是“带层级传播和收敛缓冲”的引用计数。

---

## 2. 关键类与数据结构

`uvm_objection` 内部关键状态：

- `m_source_count[obj]`：对象自身发起的 objection 数
- `m_total_count[obj]`：对象及其后代总 objection 数
- `m_drain_time[obj]`：total 降到 0 后的缓冲时间
- 三组 drain 队列/上下文：`m_scheduled_list`、`m_forked_list`、`m_forked_contexts`
- `m_drain_proc[obj]`：活跃 drain 进程句柄，便于 re-raise 时 kill
- 传播模式 `m_prop_mode`：控制是否逐级传播，或直接折叠到 top 处理

---

## 3. 关键调用链（按时间顺序）

### 3.1 raise 路径

1. `raise_objection(obj, desc, count)`
2. `m_raise(obj, source_obj=obj, ...)`
3. 增加 source/total 计数
4. 调用 `raised()` 回调链
5. 若存在同 obj 的 pending drain：
   - 从 scheduled/forked/forked_contexts 抓到 context
   - 计算 `diff_count`
   - 必要时 kill drain 进程并重新传播
6. 否则按 `m_prop_mode` 正常向父层传播

### 3.2 drop 路径

1. `drop_objection(obj, desc, count)`
2. `m_drop(obj, source_obj=obj, ...)`
3. source/total 减计数，若减成负值直接 fatal（OBJTN_ZERO）
4. 调用 `dropped()` 回调链
5. 若 total 未到 0：继续向父层传播 drop
6. 若 total 到 0：
   - 创建 objection context
   - 放入 `m_scheduled_list`
   - 后台任务 `m_execute_scheduled_forks()` 异步执行 drain

### 3.3 收敛与取消（核心）

`m_forked_drain` 流程可概括为：

1. 等待 `drain_time`
2. 调用 `all_dropped` 回调
3. 若期间发生 re-raise：该 drain 可能被 kill，中止向上 drop 传播
4. 若未被打断，再向父层传播最终 drop

这就是“为什么刚 drop 完 run_phase 没立刻结束”的直接原因。

---

## 4. 常见误区与排障手段

1. 误区：raise/drop 必须成对且只能在同一对象上

正确说法：source 计数需自洽；total 按层级汇总。**对象不一致会让计数关系复杂且易错。**

1. 误区：drop 到 0 就立即 all_dropped

实际会经过 drain + 异步 fork，期间可被新 raise 打断。

1. 误区：sequence 的 parent 一定是 component 父层级

`m_get_parent` 对 sequence 的父对象优先是 parent sequence，其次是 sequencer。

1. 排障建议

- 开启 `+UVM_OBJECTION_TRACE`
- 关注 `OBJTN_ZERO` / `OBJTN_CLEAR` 报告
- 对长时间不收敛场景检查：
      - 是否漏 drop
      - 是否循环 re-raise
      - drain_time 是否设置过大

---

## 5. 复习速记

1. objection 是层级化引用计数，不是简单开关
2. total 到 0 只是进入收敛流程，不代表立刻结束
3. re-raise 会中断 pending drain，这是设计特性
4. source_count 看“谁提的”，total_count 看“整棵子树”
5. 调试首选 `+UVM_OBJECTION_TRACE`
