# VCS 仿真资源分析（1）：-reportstats

> 来源：<https://www.cnblogs.com/cmyxjcc/p/18980436>
> 说明：本文为原文解析整理版，聚焦 `-reportstats` 的读法与资源概念。

## 1. 使用目的

当 case 运行变慢或内存异常增长时，可用 `-reportstats` 获取仿真结束时的资源统计快照。

## 2. 使用方式

在仿真阶段添加选项：

```shell
./simv -reportstats [run_options]
```

该统计会输出到终端，并可在仿真日志中查看。

## 3. 统计项怎么读

![reportstats 输出示例](assets/vcs-reportstats-sample.png)

常见字段含义：

- `Elapsed Time`：挂钟时间（从启动到结束）。
- `CPU Time`：所有相关进程累计 CPU 时间。
- `virtual memory size`：虚拟内存总量。
- `resident set size (RSS)`：常驻物理内存。
- `shared/private memory`：共享/私有内存占用。
- `major page faults`：主缺页次数（不一定是错误，但常对应 I/O 交换开销）。

## 4. 为什么要关注 swap 与虚拟内存

当物理内存不足时，系统会将部分页换出到磁盘（swap），常带来明显性能下降。

常见形态：

- 交换分区（Swap Partition）
- 交换文件（Swap File）

观察要点：

- RSS 是否持续逼近物理内存上限。
- major page faults 是否伴随仿真速度下降。
- 是否出现 profile 选项导致的额外资源开销。

## 5. 实战建议

- 首先用 `-reportstats` 建立基线，判断问题是“时间瓶颈”还是“内存瓶颈”。
- 若需要更细粒度定位，升级到 `-simprofile` + `profrpt` 深入分析。
- 固定相同输入条件（case/seed/机器）进行对比，避免噪声。
