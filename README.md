# 电源模式一键切换

适用于 Windows 11 / ThinkPad 的电源模式切换工具。双击程序，或绑定到自定义快捷键（如 ThinkPad F12 星标键），即可轻松切换电源模式为“最佳能效”或“最佳性能”。

## 功能

- 同时设置 AC（已接通电源）和 DC（使用电池）的电源模式。
- 当前两者都是“最佳性能”时，切换为“最佳能效”。
- 其他状态（包括“最佳能效”、平衡模式或两者不一致）统一切换为“最佳性能”。
- 屏幕顶部居中显示一个置顶提示当前模式。
- 首次运行需要一次管理员授权安装文件（完全卸载见下方说明），之后双击或快捷键启动不再弹出 UAC。
- 无网络访问、无遥测、无第三方依赖。

## 使用

1. 双击 `电源模式切换.exe`。
2. 第一次运行会弹出一次 UAC，同意后程序会安装后台组件并立即执行一次切换。
3. 之后直接双击即可切换，或把该 EXE 绑定到自定义快捷键。
4. 切换完成后会显示 `最佳性能` 或 `最佳能效` 提示。

提示显示约 1.5 秒，自动消失。

## 一次性授权说明

Windows 的电源模式写入需要管理员权限。为了让后续快捷键启动不必每次都确认 UAC，程序首次运行时会：

- 将后台副本安装到 `C:\ProgramData\PowerModeToggle\PowerModeToggleTask.exe`。
- 在任务计划程序中注册 `\PowerModeToggle\PowerModeToggleTask`。
- 任务以最高权限、仅当前用户交互登录时运行。
- 后台目录 ACL 仅允许管理员和 SYSTEM 修改，普通用户只有读取/执行权限。

之后主程序通过任务计划 COM 接口按需启动该任务，因此无需 UAC。

## 完全卸载

在命令行中运行：

```powershell
.\电源模式切换.exe --uninstall
```

程序会请求一次管理员权限，然后删除计划任务和后台目录。

## 命令行参数

| 参数 | 说明 |
| --- | --- |
| 无参数 | 切换电源模式 |
| `--preview` | 仅显示提示窗口，不切换 |
| `--install` | 重新安装后台授权，并切换一次 |
| `--uninstall` | 删除后台授权 |
| `--help` | 显示帮助 |

## 构建

环境要求：

- Windows 11 x64
- 系统自带 .NET Framework 4.8
- 无需安装 .NET SDK 或 Visual Studio

在项目根目录运行：

```powershell
powershell -ExecutionPolicy Bypass -File .\build.ps1
```

构建脚本使用 Windows 自带的 C# 编译器和 WPF 程序集，输出：

- `电源模式切换.exe`
- `电源模式切换.exe.config`

## 原理

程序调用 Windows 未公开但系统设置界面使用的接口：

- `PowerGetUserConfiguredACPowerMode`
- `PowerGetUserConfiguredDCPowerMode`
- `PowerSetUserConfiguredACPowerMode`
- `PowerSetUserConfiguredDCPowerMode`

它们位于 `powrprof.dll`，分别对应“已接通电源”和“使用电池”的电源模式。模式 GUID：

- 最佳能效：`961cc777-2547-4f9d-8174-7d86181b8a7a`
- 最佳性能：`ded574b5-45a0-4f42-8737-46345c09c238`

提示窗口使用 WPF 分层透明窗口实现圆角半透明背景，不做淡入淡出或其他动画。

## 已知限制

- 仅适用于 Windows 11。Windows 10 没有上述 AC/DC 独立电源模式接口。
- 如果电源模式被 OEM 软件、组策略或“节能模式”接管，切换可能被系统拒绝。
- 计划任务依赖当前用户交互登录；锁屏或未登录时不会显示提示。
- 如果修改或删除后台目录中的文件，需要重新以管理员身份运行 `--install`。

## 许可证

本项目采用 **Apache License 2.0** 开源许可证。

- SPDX 标识：`Apache-2.0`
- 版权声明：Copyright 2026 The Xentura
- 许可证全文：[LICENSE](LICENSE)
- 附加声明：[NOTICE](NOTICE)

你可以按照 Apache License 2.0 的条款使用、修改和分发本项目。分发时请保留 `LICENSE` 和 `NOTICE` 文件。
