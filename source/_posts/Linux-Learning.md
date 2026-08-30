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
```

- `swapfile` 是一个交换文件

  - 当物理内存(RAM)不足时，系统会把一部分暂时不用的数据从内存中转移到`swapfile`中，腾出内存给正在运行的程序

  - `swapfile`也可在休眠时使用，系统将内存中的所有数据保存到交换空间，然后断电；恢复时再读回内存。

|          目录          |                             作用                             |                             示例                             |
| :--------------------: | :----------------------------------------------------------: | :----------------------------------------------------------: |
|         **/**          | 根目录，所有挂载点和路径的起点。包含系统必须的子目录和入口结构。 |                              -                               |
|        **/bin**        | 存放基本用户命令，这些命令在系统启动、单用户模式或紧急修复时也必须可用。 |                `ls`, `cp`, `mv`, `rm`, `bash`                |
|       **/sbin**        |              系统管理与维护命令，面向 `root`。               |                 `fsck`, `reboot`, `shutdown`                 |
|        **/usr**        |   包含了系统的大部分用户应用程序、库、文档和其它共享资源。   |             `/usr/bin`, `/usr/lib`, `/usr/share`             |
|      **/usr/bin**      |                  常规用户程序的主要放置目录                  | `python3`, `vim`, `git`, `gcc`, `ls`（在现代系统中，`/bin/ls` 实际上是 `/usr/bin/ls` 的符号链接） |
|     **/usr/sbin**      |                      管理工具的扩展集合                      | `useradd`, `apache2ctl`<br />`/sbin` 实际上是一个指向 `/usr/sbin` 的符号链接，`/sbin/useradd` 和 `/usr/sbin/useradd` 是同一个文件 |
|      **/usr/lib**      |              `/usr` 内程序所依赖的动态库与模块               | 应用程序运行所需的共享库（动态链接库，如 `libc.so.6`、`libssl.so`）。  <br />某些程序的内部模块、插件或架构相关的数据。<br />内核模块（通常在 `/usr/lib/modules/内核版本/` 下）。 |
|     **/usr/local**     |                   安装或编译软件的独立区域                   |                /usr/local/bin, /usr/local/lib                |
|     **/usr/share**     |     存放不依赖特定硬件架构的数据，可被所有架构的系统共享     | `/usr/share/doc/`：软件文档、版权信息、示例配置文件。 <br />`/usr/share/man/`：手册页（man pages）。<br /> `/usr/share/icons/`：图标主题。<br /> `/usr/share/fonts/`：字体文件。 <br />`/usr/share/locale/`：本地化翻译文件。<br /> `/usr/share/backgrounds/`：桌面壁纸。 <br />`/usr/share/applications/`：`.desktop` 文件，用于桌面菜单项。 |
| **/lib** or **/lib64** |              系统启动核心库、动态链接器所在位置              |                  `libc.so.6`, `ld-linux.so`                  |
|        **/etc**        |          系统级配置中心，统一存放所有服务和系统配置          |        `passwd`, `group`, `fstab`, `ssh/sshd_config`         |
|        `/home`         |                   普通用户的个人主目录集合                   |         `/home/user/.bashrc`, `/home/user/Documents`         |
|        `/root`         |                      root 用户的主目录                       |                        `/root/.ssh/`                         |

