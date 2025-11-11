---
title: 西电微机原理实验 - 环境配置
published: 2025-10-03
image: ''
tags: [西电-微机原理]
category: 'Course'
draft: false 
lang: ''
---

# 前言

国庆前，老师要求我们假期对第三章后面的几道题进行实操。不过由于实操环境是 8086（即 x86 指令集架构的 16 位处理器），但我们现在用的电脑基本都是 64 位的操作系统，更别说架构了，因此需要在实操之前先琢磨琢磨 8086 编程环境的配置。

# 开始

我们打开课程资料，可以看到里面给我们提供了 DOS-BOX 虚拟磁盘系统：

![](./images/resource.webp)

简单地阅读了说明文档，大致了解了我们需要做的事情：

1. 安装 DOS-BOX 到自己的电脑上
2. 配置 VSCode 插件
3. 编写汇编代码并在 DOS-BOX 中运行

# 安装 DOS-BOX

这一步我们直接解压 `DOS-BOX.rar` 然后根据里面的 `Install_Help.txt` 进行安装即可。当然，你也可以到官方网站进行下载 [DOS-BOX](https://www.dosbox.com/download.php?main=1)。

# 配置 VSCode

在 VSCode 中，我们只需要安装 MASM/TASM 插件，打开插件市场，搜索并下载：

![](./images/plugin.webp)

接着是一些必要的基础配置，点击 VSCode 左下角选择我们之前安装的 DOS-BOX 虚拟环境，推荐使用 MASM 语法：

![](./images/env.webp)


> [!TIP]
> 这里如果你还没装 DOS-BOX，其实也可以选择 jsdos 这个在线模拟器，运行后会通过 VSCode 打开一个基于 js 模拟的 DOS 窗口。

# 在 DOS-BOX 中运行汇编代码

环境配置完成之后，第一步当然是跑一个 `Hello World` 啦！代码如下，可以直接使用：

```asm title='hello.asm'
.model small
.stack 100h

.data
msg db 'Hello World!', '$'

.code
main proc
    mov ax, @data
    mov ds, ax

    mov ah, 09h
    mov dx, offset msg
    int 21h

    mov ah, 4ch
    int 21h
main endp
end main
```

右键我们创建好的汇编代码文件，选择**运行当前程序**（为了方便也可以自己定义快捷键）：

![](./images/run.webp)

至此，我们的微机实验环境就配置完成了！