---
slug: 76
title: '虚拟机中 Arch Linux 安装记录'
date: '2022-10-02T13:39:26+08:00'
author: yexca
# layout: post
# permalink: /archives/76
views:
    - '260'
categories:
    - 技术学习
tags:
    - Arch Linux
---

## 引言

使用虚拟机安装，软件为 Fedora 的 *盒子*

## 安装前准备

### 下载镜像

从[官方下载页面](https://archlinux.org/download/)下载，推荐使用 BT 下载 (请使用正规 torrent 客户端，例如 [qBittorrent](https://www.qbittorrent.org/))

然后放到虚拟机里

### 验证引导模式

列出 efivars 目录

```bash
ls /sys/firmware/efi/efivars
```

如果正确显示目录并且没有报告错误，则系统以 UEFI 模式引导，如果目录不存在，则可能以 [BIOS](https://zh.wikipedia.org/wiki/BIOS) 模式引导 (或 CSM 模式)

这个虚拟机中使用 BIOS 模式

### 连接到互联网

默认开启网络接口与 DHCP 服务，无需配置

### 更新系统时间

开启与网络时间服务器 (NTP) 同步

```bash
timedatectl set-ntp true
```

可使用 `timedatectl status` 检查服务状态

### 建立硬盘分区

使用了传统的 `fdisk` 命令分区 (MBR 分区)，因为引导是 BIOS，采用官方的分区示例，仅做了两个分区 (swap 交换分区与其他)

使用 `fdisk -l` 列出全部磁盘 (以 rom、loop 或 airoot 结尾的设备可以忽略)

使用 `fdisk /dev/设备名` 开始分区

| 命令 | 描述         |
| ---- | ------------ |
| n    | 新建分区     |
| p    | 检查分区     |
| t    | 改变分区类型 |
| w    | 保存更改     |

指定分区大小使用 `+`+`num`+`K/M/G/T/P` ，若无后缀 (`K/M/G/T/P`) 则分配扇区

### 格式化分区

* 创建交换分区

```bash
mkswap /dev/交换空间分区
```

* 创建文件系统

根据文件系统不同命令不同，例如 ext4 文件系统

```bash
mkfs -t ext4 /dev/分区
```

### 挂载分区

将根分区挂载带 `/mnt`，若有多个分区，请务必先挂载根分区

```bash
mount /dev/分区 /mnt
```

启用交换空间

```bash
swapon /dev/交换空间分区
```

## 安装

### 选择镜像

文件 `/etc/pacman.d/mirrorlist` 定义了软件包从何处下载，在连接到互联网上后会自动更新，也可手动更改，我就不改了

### 安装软件包

使用 pacstrap 脚本，安装 base 软件包和 Linux 内核以及 vim，如果安装其他软件包，在下面命令后加上名字即可，当然也可以之后使用 pacman 安装

```bash
pacstrap /mnt base linux vim
```

## 配置系统

### Fstab

`/etc/fstab` 文件描述系统启动时如何自动挂载分区，可以使用以下命令自动生成 (使用 `-U` 或 `-L` 选项设置 UUID 或卷标，使用 UUID 以确保系统引导不会出错)

```bash
genfstab -U /mnt >> /mnt/etc/fstab
```

检查自动配置是否正确

```bash
cat /mnt/etc/fstab
```

### Chroot

Chroot 至新安装的系统

```bash
arch-chroot /mnt
```

### 时区

以上海时间为例

```bash
ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
```

生成 `/etc/adjtime`

```bash
hwclock --systohc
```

此时可使用命令 `date` 查看时间是否正确

### 本地化

编辑 `/etc/locale.gen` ，取消 `en_GB.UTF-8` 的注释

然后生成 locale 信息

```bash
locale-gen
```

创建 `/etc/locale.conf` 文件，编辑 LANG 变量，例如 `LANG=en_GB.UTF-8`

### 网络配置

创建 `/etc/hostname` 文件输入主机名

因为虚拟机使用 DHCP，就不[配置网络](https://wiki.archlinux.org/title/%E7%BD%91%E7%BB%9C%E9%85%8D%E7%BD%AE)了

### Root 密码

```bash
passwd
```

### 安装引导程序

一般安装 GRUB，我用的虚拟机为 BIOS+MBR，安装 grub 软件包

```bash
pacman -S grub
```

安装 grub (下属命令 `/dev/设备`，注意不是分区)

```bash
grub-install --target=i386-pc /dev/设备
```

生成配置文件

```bash
grub-mkconfig -o /boot/grub/grub.cfg
```

## 重启

使用 `exit` 或 `Ctrl+D` 退出 chroot 环境

使用 `umount -R /mnt` 卸载被挂载的分区

重启 `reboot`

## 参考资料

[Installation guide (简体中文) - ArchWiki](https://wiki.archlinux.org/title/Installation_guide_(%E7%AE%80%E4%BD%93%E4%B8%AD%E6%96%87))