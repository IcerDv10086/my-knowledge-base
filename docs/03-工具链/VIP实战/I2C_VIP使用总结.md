# Synopsys I2C VIP 架构学习总结

## 1. 背景与适用范围

本文档基于 **Synopsys I2C VIP** 的未加密代码进行分析，旨在深入理解 VIP 的设计架构、物理层实现及 UVM 集成方法。

- **核心价值**：掌握 VIP 内部机制，便于后续工作中灵活配置、调试及二次开发。
- **适用范围**：I2C 协议验证，此架构思路同样适用于 **I3C VIP**。

---

## 2. 核心架构

Synopsys I2C VIP 基于 **SystemVerilog UVM** 构建，针对 I2C 的物理特性（Open-Drain）采用 **Wrapper + BFM** 的分层设计。

### 2.1 物理层集成

I2C 总线为双向开漏电路，需外部上拉电阻。VIP 通过 Wrapper 模块精确模拟这一特性。

| 组件 | 角色 | 详细描述 |
| :--- | :--- | :--- |
| `svt_i2c_*_wrapper` | 物理层桥梁 | 连接 DUT（直接连接 SCL/SDA）；模拟开漏（内部实现上拉电阻与三态驱动逻辑）；实例化接口（将电平信号转为逻辑信号传递给 BFM）。 |
| BFM | 协议状态机 | 协议转换（将 UVM Transaction 转换为具体的时序）；非可综合（作为硬件与验证环境的边界）。 |
| `svt_i2c_if` | 信号通道 | 承载总线信号，连接 Wrapper/BFM 与 UVM Agent 的 Monitor 和 Driver。 |

> **说明**：Wrapper 和 Interface 一般例化在 tbench 顶层，Interface 需连接到 DUT 的 I2C 总线接口信号。BFM 本质是 Wrapper 内的模块，内部定义了丰富的事件和信号，可协助 Debug。

### 2.2 UVM 环境集成

VIP 采用标准的 UVM 树形结构，顶层由 `svt_i2c_system_env` 管理系统级拓扑。

```text
i2c_env (User Top Env)
└── i2c_system_env (svt_i2c_system_env)
    ├── slave[n] (svt_i2c_slave_agent)
    │   ├── monitor
    │   ├── sequencer
    │   └── driver ──> (Forwarding transaction to BFM)
    ├── master[m] (svt_i2c_master_agent)
    │   ├── monitor
    │   ├── sequencer
    │   └── driver ──> (Forwarding transaction to BFM)
    └── i2c_system_cfg (系统配置对象)
```

> **说明**：Master 和 Slave 的个数由系统配置对象决定，详见下节。

---

## 3. 配置对象详解

VIP 的配置分为三个层次，分别对应物理参数、代理行为和系统拓扑。

| 配置类 | 作用域 | 核心功能 | 关键参数示例 |
| :--- | :--- | :--- | :--- |
| `svt_i2c_configuration` | 物理层/时序 | 定义总线电气特性和时序参数 | Bus Speed (`STANDARD`, `FAST`, `HIGH_SPEED`)、Timing (`tHigh`, `tLow`)、Address Mode (7-bit/10-bit)、协议检查开关 |
| `svt_i2c_agent_configuration` | 逻辑层/组件 | 定义单个 Master 或 Slave 的行为模式 | `is_active`、`cvg_enable`、`monitor_enable`、日志打印开关 |
| `svt_i2c_system_configuration` | 系统/拓扑 | 定义整个网络的拓扑结构 | `num_masters`、`num_slaves`、`slave_addr_cfg`、总线冲突检测 |

> **使用提示**：用户通常只需实例化并配置顶层的 `svt_i2c_system_configuration`，其内部已包含 `master_cfg` 和 `slave_cfg` 的动态数组。

---

## 4. 事务类详解

Transaction 是 UVM 序列与 Driver 交互的基本单位。

### 4.1 类继承关系

| 类名 | 角色 | 继承关系 | 主要用途 |
| :--- | :--- | :--- | :--- |
| `svt_i2c_transaction` | 通用基类 | `uvm_sequence_item` | 定义地址、数据、命令等基础属性 |
| `svt_i2c_master_transaction` | Master | extends `svt_i2c_transaction` | 控制 Master 发起/结束传输，支持错误注入 |
| `svt_i2c_slave_transaction` | Slave | extends `svt_i2c_transaction` | 控制 Slave 响应行为（回传数据、ACK/NACK） |

### 4.2 Master Transaction 关键参数

| 字段类别 | 变量 | 说明 |
| :--- | :--- | :--- |
| 基本控制 | `addr` | 目标从机地址 |
| | `cmd` | `I2C_READ` 或 `I2C_WRITE` |
| 数据 | `data[]` | 写入的数据数组（写操作时有效） |
| 时序控制 | `sr_or_p_gen` | 控制 Start/Stop：`Sr` (Repeated Start)、`P` (Stop)、空 (拼接复杂传输) |
| 特殊功能 | `send_start_byte` | 是否发送 Start Byte（唤醒慢速设备） |
| | `bus_speed` | 动态改变当前传输的总线速度 |
| 错误注入 | `enable_stop_check` | 不发送 Stop 结束传输，测试 Slave Timeout |

### 4.3 Slave Transaction 关键参数

| 字段类别 | 变量 | 说明 |
| :--- | :--- | :--- |
| 响应数据 | `data[]` | 预存的回读数据（Master 读取时按顺序发送） |
| 握手控制 | `nack_addr` | 地址匹配后回复 ACK (0) 或 NACK (1) |
| | `nack_data_count` | 指定第几个数据字节回复 NACK（中断传输） |
| 流控模拟 | `scl_low_timeout` | 模拟 Clock Stretching 的时间，测试 Master 等待能力 |

---

## 5. 编译与环境构建

### 5.1 关键宏定义

编译时必须指定 UVM 版本及厂商扩展。

| 宏名称 | 作用 | 示例 |
| :--- | :--- | :--- |
| `SVT_UVM_TECHNOLOGY` | 指定使用 UVM 基类库 | `+define+SVT_UVM_TECHNOLOGY` |
| `SYNOPSYS_SV` | 启用 Synopsys SV 扩展 | `+define+SYNOPSYS_SV` |

### 5.2 文件搜索路径

| 类型 | 路径示例 |
| :--- | :--- |
| Header Files | `.../i2c_svt_last/sverilog/include`（含 `svt_i2c_defines.svi`） |
| Source Files | `.../i2c_svt_last/sverilog/src` |
| Package | 需在 Filelist 中显式编译 `svt_i2c.uvm.pkg` |

---

## 6. 验证与 Debug 技巧

### 6.1 Config Loader

在 `build_phase` 使用 `uvm_config_db` 或直接赋值，建立 `system_cfg` -> `master_cfg` 的映射，确保时序参数正确下发。

### 6.2 Virtual Sequence 协调

使用 `i2c_virtual_sequencer` 协调 Master 和 Slave：

1. Master 发送 Write Transaction
2. Fork 开启 Slave Response Sequence（配置为 ACK 且准备好数据）
3. Master 发送 Read Transaction
4. 检查读回数据是否与 Slave 预存数据一致

### 6.3 波形 Debug

利用 BFM Module 内部的 Event 和 Variable 辅助调试波形。
