# 设计检查清单

本人从网上收集了一些设计检查清单，用于检查设计是否符合要求，以下为其中的一些检查项：

## 一、系统设计

- 设计符合MRD/PRD指标
- 设计符合工艺、电压、温度规范
- 第三方IP的功耗/时序/时钟/复位验证完整
- 文档齐全：RTL/netlist、firmware、仿真模型

## 二、功耗管理

- 低功耗设计检查：多电压域、动态电压关断
- 各模块上电顺序与复位释放时序
- upf检查：电源域定义、隔离单元、电平转换器
- 低功耗模式下信号状态控制（don't touch）
- POR期间IO状态检查（glitch/monster mode）
- 电源域crossing处理（isolation/level shift）

## 三、复位设计

- 复位类型确认：异步/同步释放
- 外部复位pad连接与上电速度控制
- scan chain中复位信号处理
- 复位与时钟同步处理

## 四、时钟设计

- 时钟树综合（CTS）检查
- 时钟在各PVT角落的变化范围
- OTP/FLASH时钟配置处理
- 门控时钟（clock gating）优化
- 跨时钟域（CDC）处理
- 时钟信号避免用作功能信号

## 五、IO/接口

- 输入PIN同步处理（两级/握手/FIFO）
- 同步器约束（max_delay、setup/hold）
- 输出信号约束（output_delay）
- 特殊接口：SPI/I2C/USB等协议检查

## 六、测试与可测性

- 内部状态寄存器可读写
- 关键信号可引出至管脚
- DFT模式支持：JTAG/SRAM/Flash
- 设计中避免multi-driven/undriven信号
- 优化单元命名为don't_touch

## 七、时序约束（SDC）

- spyclock rdc检查
- 所有phase的ilog/ireport review
- DC/log与tapeout版本一致性
- 警告/错误处理（waive确认）
- CDC检查报告分析

## 八、扫描链（SCAN）

- scan chain I/O配置
- at-speed测试支持
- 测试功耗分析
- PIN直连优化测试覆盖
- scan模式下信号固定处理
- 电源管理单元scan bypass

## 九、报告检查

- CDC/Timing/Gate/DRC报告
- FM/FV验证报告
- VCD/PTXP报告

## 十、其他关键项

- 异步设计中数据宽度一致性
- 上下拉电阻配置确认
- GPIO时序约束与模拟信号处理
- 行为模型（Behavioral model）与RTL一致性
- Zero Glitch风险评估
