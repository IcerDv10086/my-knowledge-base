---
title: transaction recording与printer_packer
---

## transaction recording与printer_packer

> 备注：本文基于 UVM 1800.2-2020.3.1 的 `src/base/uvm_component.svh`、`uvm_recorder.svh`、`uvm_printer.svh`、`uvm_packer.svh`、`src/seq/uvm_sequencer_base.svh`、`uvm_sequence_base.svh` 源码整理。

---

## 1. 本质：这三套机制各解决什么

1. transaction recording

把事务生命周期和字段写入交易数据库（tr_database/stream/recorder）。

1. printer

把对象结构按策略格式化输出（table/tree/line 视图）。

1. packer

把对象内容序列化为 bit/byte/int 流，并支持反序列化。

三者关系是：

- printer 面向“人读日志”，packer 面向“机器传输/存储”，recorder 面向“事务时间线与关联”。

---

## 2. 关键类与数据结构

### 2.1 recording 相关

- `uvm_component::begin_tr/end_tr/record_event_tr/record_error_tr`
- `uvm_tr_database`、`uvm_tr_stream`、`uvm_recorder`
- `uvm_recorder` 内部状态：open/close/free、stream DAP、recursion policy、id/radix 设置

### 2.2 printer 相关

- `uvm_printer`（policy）
- 关键 API：`print_object/print_field/print_string/...`
- knobs 控制深度、是否显示 type/name/size/radix、递归策略、输出文件等

### 2.3 packer 相关

- `uvm_packer`（policy）
- 关键 API：
    - pack：`pack_field/pack_object/pack_string/...`
    - unpack：`unpack_field/unpack_object/unpack_string/...`
- 支持默认 packer 全局配置（core service）

---

## 3. 关键调用链（按时间顺序）

### 3.1 transaction recording 链路

1. 组件或序列调用 `begin_tr(...)` 开始事务
2. 获得 recorder handle，进入 stream
3. 事务执行期间通过 `do_record` 记录字段
4. 调用 `end_tr(...)` 结束事务并落库
5. 可选 `free_handle` 释放句柄

在 sequence 场景：

- `uvm_sequence_base::start()` 会在根序列与子序列场景调用 `begin_tr`，并建立 parent_handle 关系，形成可追踪父子事务树。

### 3.2 自动 item recording

- `uvm_sequencer_base` 内有 `m_auto_item_recording` 开关（默认通常开启）
- 可通过 `disable_auto_item_recording()` 关闭
- 适用于对性能敏感或已有自定义 recording 管线的场景

### 3.3 print/pack 链路

1. 对象调用 `print/sprint` -> printer policy 遍历 `do_print`
2. 对象调用 `pack/unpack` -> packer policy 执行 `do_pack/do_unpack`
3. 若使用 field 宏，框架会自动把字段接入 print/pack/record 操作

---

## 4. 常见误区与排障手段

1. 误区：打开 recording 就一定有完整字段

字段是否记录取决于对象的 `do_record` / field 宏实现。

1. 误区：printer 与 recorder 输出总一致

两者走不同 policy 与开关；打印可见不代表已落事务数据库。

1. 误区：packer 不需要版本兼容策略

pack/unpack 顺序、metadata 选项变化都会影响兼容性。

1. 排障建议

- 检查组件 `recording_detail` 与 `set_recording_enabled`
- 对关键 transaction 明确 begin/end 配对
- 对 pack/unpack 建立回环测试（pack 后立刻 unpack 比对）

---

## 5. 复习速记

1. recorder 是“事务时间轴”，printer 是“可读文本”，packer 是“比特流序列化”
2. transaction 可形成父子层级，便于分析 sequence 与 item 关联
3. 自动 item recording 能省事，但也可能带来性能成本
4. 字段能否打印/打包/记录，根因在 `do_*` 与 field 宏
5. 工程里应把 print、pack、record 当三条独立可配置链路
