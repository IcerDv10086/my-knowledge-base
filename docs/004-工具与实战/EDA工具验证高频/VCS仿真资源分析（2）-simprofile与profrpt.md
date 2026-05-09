# VCS 仿真资源分析（2）：simprofile 与 profrpt

> 来源：<https://www.cnblogs.com/cmyxjcc/p/18980524>
> 说明：本文为原文解析整理版，聚焦 Unified Simulation Profiler 的落地流程。

## 1. 工具定位

`simprofile` 用于在仿真过程中采集 CPU/内存画像，`profrpt` 用于生成可分析报告。

适合场景：

- case 运行时间异常增长
- 内存峰值难以定位
- 优化前后需要量化对比

## 2. 基本流程

1. 编译阶段添加 `-simprofile`。
2. 运行阶段添加 `-simprofile <args>`。
3. 使用 `profrpt` 对 `simprofile_dir` 进行报表分析。

示例：

```shell
vcs -simprofile [compile_options]
./simv -simprofile time [run_options]
profrpt simprofile_dir -view time_all -format text
```

## 3. 常见参数

`-simprofile` 常见 args：

- `time`：CPU 时间分析
- `mem`：内存分析
- `noprof`：不采样（降低影响）
- `noreport`：不直接生成报告

实践建议：

- 不要同时盲目收集所有数据，先明确目标（时间或内存）。
- 编译、运行两阶段选项要配套，避免“开了但没采到”。

## 4. 关键产物

仿真后通常会得到：

- `simprofile_dir`：核心数据库
- `profileReport` / `profileReport.html`：概览信息

建议：

- 用独立路径保存数据库，避免与旧结果混淆。
- 用可读文件名记录场景（如 case、seed、日期）。

## 5. profrpt 常用视图

命令骨架：

```shell
profrpt simprofile_dir -view view1[+view2] -format text|html|ALL -output <name>
```

常见时间视图：

- `time_summary`
- `time_inst`
- `time_mod`
- `time_constr`
- `time_all`

常见内存视图：

- `mem_summary`
- `mem_inst`
- `mem_mod`
- `dynamic_mem`
- `dynamic_mem+stack`
- `mem_all`

## 6. 进阶分析

- 快照：

```shell
profrpt simprofile_dir -view mem_all -snapshot delta
```

- 时间线：

```shell
profrpt simprofile_dir -view mem_all -timeline vcs_DA+vcs_AA
```

- 差分对比：

```shell
profrpt -view ALL -diff simprofile_dir simprofile_dir.1 -output diff
```

## 7. 落地经验

- 先用 `time_summary`/`mem_summary` 粗定位，再下钻实例/模块视图。
- 基准对比必须保证输入一致（case/seed/tool version）。
- 将报告纳入回归产物，形成长期性能趋势。
