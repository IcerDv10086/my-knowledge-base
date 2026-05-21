# VCS 常用命令速查

本文按“编译选项”和“运行选项”整理 VCS 常见参数，便于日常回归与问题定位快速查阅。

## 1. 编译阶段

### 1.1 基础模式

- `-sverilog` / `-v2k`：启用 SystemVerilog、Verilog-2001 语法。
- `-full64`：启用 64 位编译模式。
- `-l <file>`：指定编译日志文件。
- `-o <simv_name>`：重命名输出的可执行仿真文件。

### 1.2 调试与可视化

- `-debug_access+all` 或 `-debug_all`：打开调试信息，便于 UCLI / Verdi 调试。
- `-gui`：编译后直接进入图形界面（视本地工具链配置而定）。
- `+vcs+initreg+<0|1|x|z>`：统一初始化寄存器，减少随机 X 传播造成的噪声。

### 1.3 代码与库管理

- `-v <file>`：指定 Verilog 库文件，解析未在当前文件列表定义的模块。
- `-y <dir>`：指定库目录。
- `+libext+<ext>`：与 `-y` 联用，指定库文件后缀（如 `.v`、`.sv`）。
- `+incdir+<dir>`：追加头文件搜索路径。
- `+define+<MACRO>[=<value>]`：传入预编译宏。
- `-top <module>`：显式指定顶层模块。

### 1.4 覆盖率编译

- `-cm <line|cond|fsm|tgl|branch|assert>`：指定覆盖率类型。
- `-cm_hier <file>`：按层级配置文件筛选覆盖率采集范围。
- `-cm_count`：记录命中次数而非仅记录“是否命中”。

### 1.5 时序与精度控制

- `-timescale=<unit>/<precision>`：为未声明 ``timescale` 的文件补充时间单位/精度。
- `+notimingcheck`：关闭 setup/hold 等时序检查（常用于前仿功能验证）。
- `+nospecify`：忽略 `specify` 路径延时。

## 2. 运行阶段

### 2.1 基础运行

- `./simv -l <file>`：指定运行日志输出。

### 2.2 随机种子

- `+ntb_random_seed=<seed>`：固定或指定随机种子，便于问题复现。

### 2.3 覆盖率运行

- 与 `-cm` 系列选项配合使用，在运行阶段产出覆盖率数据库。

## 3. 实战建议

- 功能前仿优先保证可复现：固定种子、固定日志命名、记录命令行。
- 时序后仿优先保证真实性：尽量避免 `+notimingcheck` / `+nospecify`。
- 若涉及 SDF 反标，建议在编译阶段提前打开必要的路径检查与告警开关，避免将语法和映射问题拖到运行期才暴露。
