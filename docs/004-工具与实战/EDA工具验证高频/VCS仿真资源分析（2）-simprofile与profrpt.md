# VCS工具的仿真资源分析（2）

> 来源：<https://www.cnblogs.com/cmyxjcc/p/18980524>
> 说明：按原文风格整理，重点保留选项含义和 `profrpt` 视图使用方式。

`Unified Simulation Profiler` 是 VCS 自带的资源分析能力，适合在“仿真能跑但跑得慢/吃内存”的阶段做瓶颈定位。相比 `-reportstats`，它会更重，但也更细。

## 1. 主要功能

- 性能分析：定位 CPU 时间花在哪些模块/实例。
- 内存分析：定位内存峰值来源和增长路径。
- 报告输出：生成可追踪、可对比的分析报告。

## 2. 使用方法

典型流程：

1. 编译阶段增加 `-simprofile`。
2. 仿真阶段增加 `-simprofile <args>`。
3. 使用 `profrpt` 基于数据库生成分析报告。

示例：

```shell
vcs -simprofile [compile_options]
./simv -simprofile time [run_options]
profrpt simprofile_dir -view time_all -format text
```

## 3. 选项 simprofile 及其参数

常见参数：

- `time`：采集 CPU 时间相关信息。
- `mem`：采集内存相关信息。
- `noprof`：不做采样（用于控制性能影响）。
- `noreport`：不直接生成报告文件。

常见产物：

- `simprofile_dir`：核心数据库。
- `profileReport` / `profileReport.html`：概览信息。

注意事项：

- 编译和运行阶段的 profile 配置要一致。
- 已有旧目录时要明确输出路径，避免混淆。
- 建议按场景重命名报告，便于回归对比。

## 4. 脚本 profrpt

命令格式：

```shell
profrpt simprofile_dir -view view1[+view2[+...]] \
 [-format text|html|ALL] [-output <name>] \
 [-snapshot [delta|incr|delta+incr]] \
 [-timeline [dynamic_memory_type_or_class+...]]
```

### 4.1 视图类型说明

时间相关常用视图：

- `time_summary`
- `time_inst`
- `time_mod`
- `time_constr`
- `time_solver`
- `time_callercallee`
- `time_all`

内存相关常用视图：

- `mem_summary`
- `mem_inst`
- `mem_mod`
- `mem_constr`
- `dynamic_mem`
- `dynamic_mem+stack`
- `mem_solver`
- `mem_callercallee`
- `mem_all`

### 4.2 内存快照机制

```shell
profrpt simprofile_dir -view mem_all -snapshot delta
```

- `delta`：内存有变化就抓快照。
- `incr`：只在内存增长时抓快照。

### 4.3 时间线机制

```shell
profrpt simprofile_dir -view mem_all -timeline vcs_DA+vcs_AA
```

可按动态内存类型过滤时间线，例如动态数组、队列、关联数组等。

### 4.4 累计与比较视图

- 累计视图：合并多个数据库观察整体趋势。
- 比较视图：对比两次仿真差异。

```shell
profrpt -view ALL -diff simprofile_dir simprofile_dir.1 -output diff
```

## 5. 实战建议

- 先看 summary，再下钻 inst/mod，避免一上来就看细节视图。
- 时间和内存问题分开采样，报告更干净、结论更明确。
- 对比实验必须固定 case、seed、工具版本和运行环境。
