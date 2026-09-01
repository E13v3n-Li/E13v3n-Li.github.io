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

------

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



- /snap 存放啊snap软件包及其数据。snap是ubuntu主推的，snap应用安装在/snap/<软件名>/<版本号>/，挂载为只读。



- /cdrom 传统的光盘挂载点，用于挂载 CD/DVD 或 ISO 镜像。

| 目录                   | 作用                                                         | 示例/说明                                                    |
| :--------------------- | :----------------------------------------------------------- | :----------------------------------------------------------- |
| **/**                  | 根目录，所有挂载点和路径的起点。包含系统必须的子目录和入口结构。 | -                                                            |
| **/bin**               | 存放基本用户命令，这些命令在系统启动、单用户模式或紧急修复时也必须可用。 | `ls`, `cp`, `mv`, `rm`, `bash`                               |
| **/sbin**              | 系统管理与维护命令，面向 `root`。                            | `fsck`, `reboot`, `shutdown`                                 |
| **/usr**               | 包含了系统的大部分用户应用程序、库、文档和其它共享资源。     | `/usr/bin`, `/usr/lib`, `/usr/share`                         |
| **/usr/bin**           | 常规用户程序的主要放置目录                                   | `python3`, `vim`, `git`, `gcc`, `ls`（在现代系统中，`/bin/ls` 实际上是 `/usr/bin/ls` 的符号链接） |
| **/usr/sbin**          | 管理工具的扩展集合                                           | `useradd`, `apache2ctl`<br />`/sbin` 实际上是一个指向 `/usr/sbin` 的符号链接，`/sbin/useradd` 和 `/usr/sbin/useradd` 是同一个文件 |
| **/usr/lib**           | `/usr` 内程序所依赖的动态库与模块                            | 应用程序运行所需的共享库（动态链接库，如 `libc.so.6`、`libssl.so`）。  <br />某些程序的内部模块、插件或架构相关的数据。<br />内核模块（通常在 `/usr/lib/modules/内核版本/` 下）。 |
| **/usr/local**         | 安装或编译软件的独立区域                                     | /usr/local/bin, /usr/local/lib                               |
| **/usr/share**         | 存放不依赖特定硬件架构的数据，可被所有架构的系统共享         | `/usr/share/doc/`：软件文档、版权信息、示例配置文件。 <br />`/usr/share/man/`：手册页（man pages）。<br /> `/usr/share/icons/`：图标主题。<br /> `/usr/share/fonts/`：字体文件。 <br />`/usr/share/locale/`：本地化翻译文件。<br /> `/usr/share/backgrounds/`：桌面壁纸。 <br />`/usr/share/applications/`：`.desktop` 文件，用于桌面菜单项。 |
| **/lib** or **/lib64** | 系统启动核心库、动态链接器所在位置                           | `libc.so.6`, `ld-linux.so`                                   |
| **/etc**               | 系统级配置中心，统一存放所有服务和系统配置                   | `passwd`, `group`, `fstab`, `ssh/sshd_config`                |
| **/home**              | 普通用户的个人主目录集合                                     | `/home/user/.bashrc`, `/home/user/Documents`                 |
| **/root**              | root 用户的主目录                                            | `/root/.ssh/`                                                |
| **/var**               | 频繁变化的数据：日志、缓存、数据库运行文件等。               | `var/log`,`/var/lib`, `/var/cache`                           |
| **/var/log**           | 系统与服务日志                                               | `syslog`, `auth.log`, `kern.log`                             |
| **/var/cache**         | 应用和包管理缓存                                             | `/var/cache/apt/`                                            |
| **/var/lib**           | 服务的持久化状态数据                                         | `mysql/`, `docker/`                                          |
| **/tmp**               | 程序运行的临时文件区                                         | 临时文件、socket路径                                         |
| **/boot**              | 启动所需文件（内核、initramfs、引导配置）                    | `vmlinuz-*`, `initrd.img`, `grub/grub.cfg`                   |
| **/dev**               | 设备节点集合（Linux中文件即设备）                            | `/dev/sda`, `/dev/null`, `/dev/tty0`                         |
| **/proc**              | 内核提供的虚拟文件系统，展示系统与进程的实时信息             | `/proc/cpuinfo`, `/proc/<pid>/`                              |
| **/sys**               | 设备、驱动、内核子系统的状态接口                             | `/sys/class/net/`, `/sys/block/`                             |
| **/mnt**               | 手动挂载的临时挂载点                                         | `/mnt/usb`                                                   |
| **/media**             | 自动挂载外界设备的默认位置                                   | `/media/user/USB_DRIVE`                                      |
| **/run**               | 运行时的数据存放点，重启后清空                               | `*.pid`, 运行状态 `socket` 文件                              |
| **/lost+found**        | 当系统因为突然断电或崩溃导致文件系统损坏，运行`fsck`修复后找回的文件碎片 | 只存在于`ext2/ext3/ext4`等文件系统的分区中。每个`ext`文件系统的挂载点（如 `/`、`/home`）下都会有一个`lost+found` |
| **/lib32和/libx32 **   | /lib32：存放32位共享库，供32位程序在64位系统上运行<br />/libx32：存放x32 ABI的共享库 | -                                                            |
| **/opt**               | 用于安装第三方或商业软件（非官方仓库提供的软件）             | 例如，Google Chrome、一些开发工具等                          |
| **/srv**               | 用于存放系统服务（如 Web服务器、FTP服务器、版本控制仓库等）提供的数据 | Web服务器（如 Apache、Nginx）的网站文件可以放在/srv/www/可以放在/srv/git/或/srv/http/<br/>FTP服务器的文件可以放在/srv/ftp/<br/>Git仓库可以放在/srv/git/ |

