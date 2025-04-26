---
slug: 107
title: 'Honkai: Star Rail 国际服分流规则'
date: '2023-05-16T23:13:21+08:00'
author: yexca
layout: post
permalink: /archives/107
views:
    - '32'
categories:
    - 折腾经验
tags:
    - Game
    - miHoYo
---

针对中国大陆无法进入游戏分流规则

## Key

在进入游戏时需要 udp 连接，而多数协议不支持 udp，即遇到 udp 会自动拒绝，所以将 udp 连接的 IP 设置为直连即可，下面是我抓取到的两个 IP

```bash
8.209.196.179
# 第二个貌似直连不会连接，只用设置第一个就行
47.245.63.117
```

当然，为了安全起见 (指 tcp 连接与 udp 连接不一致可能造成的安全隐患) ，建议将相关域名都设置直连，即以下这些

```bash
*.starrails.com
*.hoyoverse.com
8.209.196.179/8
```

## OpenClash

在 `全局设置-规则设置` ，启用 `自定义规则` ，第一个框的 `rules:` 下输入

```yaml
#rules:
- DOMAIN-SUFFIX, starrails.com, DIRECT
- DOMAIN-SUFFIX, hoyoverse.com, DIRECT
- IP-CIDR, 8.209.196.179/8, DIRECT
```

## Quantumult X

编辑配置文件，跳转至 `[filter_local]` ，输入以下内容

```conf
#SR
host-suffix, starrails.com, direct
host-suffix, hoyoverse.com, direct
ip-cidr, 8.209.196.179/8, direct
```
