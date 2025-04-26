---
slug: 18
title: 使用 xshell 连接虚拟机的 centos7 系统
date: '2021-12-26T15:34:59+08:00'
author: yexca
# layout: post
# permalink: /archives/18
views:
    - '357'
categories:
    - 技术学习
tags:
    - Xshell
    - CentOS
---

注意：此方法为临时连接，虚拟机重启或关机需要重新配置

## 虚拟机网络适配器设置

虚拟机的网络适配器一共有三个设置

- 桥接模式：指使用本机网络网段
- NAT模式：使用VMware Network Adapter VMnet8的网段
- Host-only(仅主机模式)：使用VMware Network Adapter VMnet1的网段

## 查看IP网段

打开VMware中左上“编辑-虚拟网络编辑器”即可看到VMnet1和VMnet8对应网段地址(子网地址)

桥接模式的网段需要打开“设置-网络和Internet-高级网络设置”找到相应的本机连接的网络

- 如果使用WiFi连接，点击“WLAN-查看其他属性”即可看到IP地址
- 如果使用有线连接，点击“以太网-查看其他属性”即可看到IP地址

注意：如果本机连接了以太网和WIFI，则可能需要对VMware中左上“编辑-虚拟网络编辑器” 进行相关设置

需要给VMware管理员权限，如图示选择想要VMware连接的网卡

![](https://cdn.statically.io/gh/yexca/picx-images-hosting@master/2021/12-xshell-连接虚拟机-centos7/vmware-网卡.6pvdvehvzz40.webp)

## 设置虚拟机的IP地址

注：我使用的是桥接模式，我的本机IP为192.168.1.116，那么我可以将虚拟机设置为192.168.1.0-192.168.1.255中任意一个(除192.168.1.116)，即前三段一样，最后一段不一样即可

首先，打开虚拟机并登录root用户，输入“ifconfig”命令查看网卡配置

![](https://cdn.statically.io/gh/yexca/picx-images-hosting@master/2021/12-xshell-连接虚拟机-centos7/虚拟机-IP-配置.56i9yy8tjfo0.webp)

如果如图出现“ens33”和“lo”或者“其他”和“lo”

输入命令

```bash

# ifconfig 设备名(本例为"ens33") 要分配的地址(这里选择的是192.168.1.110)
ifconfig ens33 192.168.1.110

```

如果仅出现“lo”

 输入命令

```bash

# ifconfig 设备名(一般为"eth0") 要分配的地址(这里选择的是192.168.1.110)
ifconfig eth0 192.168.1.110

```

配置完成后，可再次输入“ifconfig”命令查看网卡配置

![](https://cdn.statically.io/gh/yexca/picx-images-hosting@master/2021/12-xshell-连接虚拟机-centos7/虚拟机-IP-配置-2.2fy0k801s03.webp)

如上图IP成功改为192.168.1.110

可以打开“Windows终端”，输入“ping 192.168.1.110”查看是否生效

![](https://cdn.statically.io/gh/yexca/picx-images-hosting@master/2021/12-xshell-连接虚拟机-centos7/ping-连接.5un7zgka3b00.webp)

如图即可表示IP修改成功并可访问

## 使用Xshell连接

打开Xshell，点击“新建”，名称可自行决定，主机填写IP，然后点击“连接”

![](https://cdn.statically.io/gh/yexca/picx-images-hosting@master/2021/12-xshell-连接虚拟机-centos7/xshell-连接界面.7jwpta0nork0.webp)

选择“接受并保存”，然后跟随提示输入用户名(root)和密码即可

![](https://cdn.statically.io/gh/yexca/picx-images-hosting@master/2021/12-xshell-连接虚拟机-centos7/xshell-连接成功.752g8y4vdsg0.webp)