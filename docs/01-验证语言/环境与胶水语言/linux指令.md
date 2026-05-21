# IC 验证常用 Linux 指令指南

## 1. 基础文件与目录管理

### 1.1 目录导航

- `cd`：切换目录。
  - `cd -`：返回上一个目录。
  - `cd ..`：返回上一级。
  - `cd ~`：回到用户主目录。
- `pwd -P`：显示物理路径（解析软链接）。

### 1.2 列表与文件操作

- `ls -lrt`：按时间倒序查看（常用于追最新日志）。
- `ls -a`：显示隐藏文件。
- `ls -h`：人类可读尺寸。
- `mkdir -p <dir>`：递归创建目录。
- `touch <file>`：更新时间戳/创建空文件。
- `ln -s <src> <dst>`：创建软链接。
- `cp -rf <src> <dst>`：递归复制。
- `mv <src> <dst>`：移动或重命名。
- `rm -rf <path>`：强制删除（高风险命令，谨慎使用）。

### 1.3 空间管理

- `du -sh *`：查看当前目录各项大小。
- `du --max-depth=1 -h`：按层级统计空间。
- `df -h`：查看挂载与磁盘剩余空间。
- `quota -s`：查看个人配额。

## 2. 日志分析与文本处理

### 2.1 搜索

- `grep -r "pattern" .`：递归搜索。
- `grep -n "pattern" file.log`：显示命中行号。
- `grep -A 3 -B 3 "ERROR" run.log`：显示上下文。
- `grep -E`：使用扩展正则。
- `find . -name "*.log"`：按名称查找。
- `find . -mtime -1`：查 24 小时内变更文件。

### 2.2 文本流处理

- `sed -i 's/Old/New/g' file`：替换文本。
- `sed -n '10,20p' file`：输出指定行。
- `awk '{print $NF}' file`：输出最后一列。

### 2.3 文件查看与比对

- `less file`：分页查看。
- `tail -f run.log`：实时跟踪日志。
- `head -n 20 file`：查看文件前 20 行。
- `diff a b` / `vimdiff a b`：差异比对。

## 3. LSF 作业相关

- `bsub`：提交作业。
- `bjobs -l`：查看作业详情。
- `bhist -l <jobid>`：查看历史退出信息。
- `bkill <jobid>`：终止作业。
- `bstop` / `bresume`：挂起/恢复作业。

## 4. 进程与环境

- `top -u <user>`：看指定用户进程。
- `ps -ef | grep <name>`：筛进程。
- `kill -9 <pid>`：强制终止进程。
- `free -g`：按 GB 查看内存。
- `lscpu`：查看 CPU 信息。
- `watch -n 1 <cmd>`：周期执行。

环境模块：

- `module avail`：可用版本。
- `module list`：已加载版本。
- `module load/unload <tool>`：加载/卸载工具。

## 5. 远程与传输

- `ssh <user>@<host>`：远程登录。
- `ssh -X <user>@<host>`：开启 X11 转发。
- `vncserver -geometry 1920x1080`：启动 VNC。
- `tar -zcvf out.tgz dir/`：打包压缩。
- `tar -zxvf out.tgz`：解压。

## 6. 用户与权限

- `su -`：完整切换用户环境（推荐）。
- `sudo -u <user> <cmd>`：以指定用户运行命令。
- `id`：查看 UID/GID/用户组。
- `whoami`：查看当前用户。
