# 设计代码规范

IC设计（特别是数字IC设计）的编码规范（Coding Guideline）是确保代码质量、可维护性和可综合性的关键。以下是个人总结的核心规范要点：

## 一、命名规范

1. **文件与模块**：文件应只包含一个模块，且文件名与模块名一致（小写+下划线）。
2. **信号命名**：
   - 全部小写，单词间用下划线分隔
   - 时钟信号加`clk`前缀，复位信号加`rst`前缀
   - 低有效信号加`_n`后缀（如`rst_n`）
   - 寄存器信号可加`_ff1`、`_ff2`等表示流水级
3. **常量与参数**：使用大写字母。

## 二、编码风格

1. **缩进与格式**：统一使用4空格缩进，每行不超过80-120字符。
2. **括号与空格**：运算符两侧加空格，复杂表达式用括号明确优先级。
3. **位宽明确**：赋值和比较时需指明位宽（如`5'd5`而非`5`），避免位宽不匹配警告。

## 三、可综合性设计

1. **避免不可综合结构**：禁止使用`initial`、`#delay`、`fork/join`、`while`等仿真专用语句。
2. **always块规则**：
   - 时序逻辑用`always_ff @(posedge clk)`，使用非阻塞赋值（`<=`）
   - 组合逻辑用`always_comb`（SystemVerilog）或`always @(*)`，使用阻塞赋值（`=`）
   - 避免在多个always块中对同一变量赋值
3. **防止锁存器**：组合逻辑中必须覆盖所有分支，提供默认赋值。

## 四、时钟、复位与同步设计

1. **时钟处理**：
   - 尽量使用单一时钟域
   - 避免用组合逻辑生成时钟（应使用时序逻辑+时钟使能）
   - 禁止在always块中同时使用时钟双边沿
2. **复位策略**：
   - 优先使用同步复位
   - 异步复位需做同步释放处理
3. **跨时钟域（CDC）**：必须使用同步器（如两级触发器）或握手协议。

## 五、状态机设计

1. **推荐三段式**：状态寄存器、次态逻辑、输出逻辑分离。
2. **状态编码**：根据场景选择One-Hot（FPGA/高速）、Binary（ASIC/状态少）或Gray码。
3. **使用enum定义状态**（SystemVerilog），提高可读性。

## 六、代码结构与维护

1. **模块化**：每个模块功能单一，便于复用和验证。
2. **注释要求**：
   - 文件头包含功能、作者、日期、修改记录
   - 复杂逻辑需注释设计意图
3. **参数化设计**：使用`parameter`或`localparam`定义常量，提高代码可配置性。

## 七、工具与验证

1. **静态检查**：使用Lint工具（如Synopsys SpyGlass、Cadence JasperGold）进行代码规范检查。
2. **仿真验证**：Testbench应分层（Driver、Monitor、Scoreboard），采用约束随机测试。
3. **综合后验证**：检查时序、面积、功耗报告，确保满足约束。

---

## 资源参考

- 的[《RTL Coding Styles That Yield Simulation and Synthesis Mismatches》](http://staff.ustc.edu.cn/~wyu0725/FPGA/snug_collection/1Sunburst%20Design/RTL%20Coding%20Styles%20That%20Yield%20Simulation%20and%20Synthesis%20Mismatches.pdf)
- KTH学院的[《Style guide for Verilog and SystemVerilog RTL》](https://silago.eecs.kth.se/docs/Guideline/Style-guide-Verilog/)
