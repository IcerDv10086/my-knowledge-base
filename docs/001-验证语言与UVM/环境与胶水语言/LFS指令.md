# LSF 常用指令速查

## 1. 提交作业：`bsub`

### 1.1 常用参数

- `-J <job_name>`：指定作业名，便于检索。
- `-o <stdout.log>`：标准输出重定向。
- `-e <stderr.log>`：标准错误重定向。
- `-q <queue>`：指定队列。
- `-n <num>`：申请 CPU 核数。
- `-R "rusage[mem=4096]"`：申请资源（如内存）。
- `-m <host>`：绑定指定执行节点。
- `-w "done(<jobid>)"`：依赖控制（前序任务完成后执行）。

### 1.2 交互模式

- `-I`：交互式运行。
- `-Is`：交互 shell。
- `-Ip`：交互图形（依赖环境支持）。

## 2. 常见组合示例

批处理仿真：

```bash
bsub -J sim_regress -q normal -n 8 -R "rusage[mem=8192]" -o sim.%J.out -e sim.%J.err "./run_regress.sh"
```

带依赖提交：

```bash
bsub -J report_gen -w "done(123456)" -q normal "python3 gen_report.py"
```

## 3. 查询与控制（补充）

- `bjobs`：查看作业状态。
- `bjobs -l <jobid>`：查看详细信息。
- `bhist -l <jobid>`：查看历史与退出原因。
- `bkill <jobid>`：终止作业。

## 4. 实战建议

1. 统一日志命名：建议带 `%J`（作业号）避免覆盖。
2. 资源按需申请，避免过度抢占导致排队时间过长。
3. 回归任务建议固定模板命令，便于复现与追溯。
