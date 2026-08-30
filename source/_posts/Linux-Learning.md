---
title: Linux Learning
date: 2026-08-22 16:18:08
categories:
  - Linux
tags:
  - Linux
---

- blog中学习所使用的是VMware虚拟机
- Ubuntu 22.04 iso镜像

# Linux系统目录结构

Linux的目录结构为树状结构，最顶级的目录为根目录 **/** 。

## 目录结构

键入命令 **ls /** ,可以看到根目录下的各个文件。

```bash
virtual-machine:~$ ls /
bin   cdrom  etc   lib    lib64   lost+found  mnt  proc  run   snap  swapfile  tmp  var
boot  dev    home  lib32  libx32  media       opt  root  sbin  srv   sys       usr


- `swapfile` 是一个交换文件
 - 当物理内存(RAM)不足时，系统会把一部分暂时不用的数据从内存中转移到`swapfile`中，腾出内存给正在运行的程序
 - `swapfile`也可在休眠时使用，系统将内存中的所有数据保存到交换空间，然后断电；恢复时再读回内存。
```

|   目录    |                             作用                             |          示例          |
| :-------: | :----------------------------------------------------------: | :--------------------: |
|   **/**   | 根目录，所有挂载点和路径的起点。包含系统必须的子目录和入口结构。 |           -            |
| **/bin**  | 存放基本用户命令，这些命令在系统启动、单用户模式或紧急修复时也必须可用。 |  ls, cp, mv, rm, bash  |
| **/sbin** |               系统管理与维护命令，面向 root。                | fsck, reboot, shutdown |

