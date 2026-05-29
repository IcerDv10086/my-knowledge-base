# fsdb 工具簇

## 1. fsdb2vcd

fsdb2vcd 是用于将 FSDB 文件转换为 VCD 格式的实用工具，支持 EVCD、MDA 和结构类型数据。

目前遇到使用到此命令的场景是：

- 后端同事动态功耗分析需要 vcd 文件
- 模拟同事在无混合仿真时需要 vcd 文件（数模接口相关的信号）作为输入进行仿真

**常用参数：**

1. **-bt_time**：设置转换开始时间
2. **-et_time**：设置转换结束时间
3. **-flat_bus**：将 bus 信号展平为单 bit 信号
4. **-s**：指定信号或层级路径，分隔符为 `/` 而非 `.`
5. **-sv**：支持 SystemVerilog 格式
6. **-ignore_2G**：转换后 VCD 文件超过 2G 仍继续转换
7. **-f**：按指令文件内容转换指定信号

**使用示例：**

- **直接指令方式**
    - 按指定起止时间转换：`bsub -I fsdb2vcd test.fsdb -o analog.vcd -bt 10140 -et 0631206`
    - 转换全部仿真时间：`bsub -Is fsdb2vcd test.fsdb -o analog.vcd`
- **指定文件方式**
    - 命令：`bsub -Is fsdb2vcd test.fsdb -o analog.vcd -f ****.txt`
    - 指令文件格式：
        - `fuSetTimeRange 0ns,100ns`：指定时间范围
        - `fuSetSignal /tb_top/*******`：指定层级或信号
        - 一个文件可指定多个信号
- 根据实际情况选择，直接指令会转换所有的signal，转换全面；指定文件方式会只转换指令文件中指定的signal，执行速度快
- **模拟的仿真分析工具一般默认使用单bit信号**，需要使用 `-flat_bus` 参数将 bus 信号展平为单 bit 信号
