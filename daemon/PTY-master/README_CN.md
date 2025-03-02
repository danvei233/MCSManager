# Pseudo-teletype App

[![--](https://img.shields.io/badge/Go_Version-1.19-green.svg)](https://github.com/MCSManager)
[![--](https://img.shields.io/badge/Support-Windows/Linux-yellow.svg)](https://github.com/MCSManager)
[![--](https://img.shields.io/badge/License-MIT-red.svg)](https://github.com/MCSManager)

仿真终端应用程序，支持运行**所有 Linux/Windows 程序**，可以为您的更高层应用带来完全终端控制能力。

中文 | [English](README.md)

<div align=center>

![terminal image](https://user-images.githubusercontent.com/18360009/202891148-e7e5bf63-c4a9-454f-8f62-c91dc594cefa.png)


</div>

> 图片中表示的是，使用仿真终端运行 Minecraft 服务器，并且按下 Tab 键来选取提示。

<br />

## 什么是 PTY/TTY？

tty = "teletype"，pty = "pseudo-teletype"

众所周知，程序拥有输入与输出流，但是数据流与显示器之间有一个区别，那便是缺少行和高的排列维度。简而言之，PTY 的中文意义就是伪装设备终端，让我们的程序伪装成一个拥有固定高宽的显示器，接受来自程序的输出内容。

<br />

## 使用

开一个 PTY 并执行命令，设置固定窗口大小，IO 流直接转发。

- 注意：-cmd 接收的是一个数组, 命令的参数以数组的形式传递，且需要序列化，如：`[\"java\",\"-jar\",\"ser.jar\",\"nogui\"]`

```bash
go build
./pty -dir "." -cmd [\"bash\"] -size 50,50
```

接下来您会得到一个设置好大小宽度的窗口，并且您可以像 SSH 终端一样，进行任何交互。

```
ping google.com
top
htop
```

<br />

## 参数：

```
  -cmd string
        command
  -coder string
        Coder (default "UTF-8")
  -dir string
        command work path (default ".")
  -size string
        Initialize pty size, stdin will be forwarded directly (default "50,50")
  -test
        Test whether the system environment is pty compatible
```

<br />

## 兼容性

- 支持所有现代主流版本 Linux 系统。
- 支持 Windows 7 到 Windows 11 所有版本系统，包括 Server 系列。
- 支持 windows amd64 / linux amd64 & arm64。


<br />

## MCSManager

MCSManager 是一款开源，分布式，开箱即用，支持 Minecraft 和其他控制台应用的程序管理面板。

这个程序是专门为了 MCSManager 而设计，您也可以尝试嵌入到您自己的程序中。

More info: [https://github.com/mcsmanager](https://github.com/mcsmanager)

<br />

## 贡献

此程序属于 MCSManager 的最重要的核心功能之一，非必要不新增功能。

- 如果您想为这个项目提供新功能，那您必须开一个 `issue` 说明此功能，并提供编程思路，我们一起经过讨论后再决定是否开发

- 如果您是修复 BUG，可以直接提交 PR 并说明情况

<br />

## MIT license

遵循 [MIT License](https://opensource.org/licenses/MIT) 开源协议。

版权所有 [zijiren233](https://github.com/zijiren233) 和贡献者们。
