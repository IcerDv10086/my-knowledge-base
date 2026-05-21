# Windows Subsystem for Linux（WSL）

## 1. WSL 是什么

WSL 是 Windows 提供的 Linux 运行环境，让开发者在 Windows 中直接使用 Linux 命令行与工具链。

## 2. WSL1 与 WSL2 差异

| 对比项 | WSL1 | WSL2 |
| --- | --- | --- |
| 内核机制 | 系统调用翻译层 | 轻量虚拟机 + 真实 Linux 内核 |
| 系统调用兼容性 | 较弱 | 更完整 |
| Docker 支持 | 受限 | 完整支持 |
| 文件系统性能 | 访问 Windows 文件较快 | 访问 Linux 文件较快 |
| 隔离性 | 进程级 | VM 级 |

工程建议：默认优先使用 WSL2。

## 3. 安装与初始化（Win10/Win11）

在管理员 PowerShell 执行：

```powershell
wsl --install
```

通常会自动完成：

- 启用 WSL 组件。
- 下载 Linux 内核。
- 安装默认发行版（常见 Ubuntu）。

重启后按提示设置 Linux 用户名与密码。

## 4. 常用命令

```powershell
wsl --status
wsl --version
wsl --list --online
wsl --list --verbose
wsl -d Ubuntu
wsl --shutdown
wsl --terminate Ubuntu
```

指定用户进入：

```powershell
wsl -d Ubuntu -u root
```

## 5. 配置文件

- Linux 内：`/etc/wsl.conf`（自动挂载、默认用户等）。
- Windows 侧：`%UserProfile%/.wslconfig`（内存、CPU、swap 等全局资源限制）。

## 6. 使用建议

1. 开发文件优先放在 Linux 文件系统路径下，减少跨文件系统性能损耗。
2. 遇到异常先执行 `wsl --shutdown` 再重试。
3. 团队脚本写明目标环境（Windows PowerShell 还是 WSL Bash）。
