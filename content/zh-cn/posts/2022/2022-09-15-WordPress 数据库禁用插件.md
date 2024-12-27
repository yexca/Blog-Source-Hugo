---
slug: 72
title: 'WordPress 数据库禁用插件'
date: '2022-09-15T22:17:02+08:00'
author: yexca
# layout: post
# permalink: /archives/72
views:
    - '207'
categories:
    - 网站建设
tags:
    - WordPress
---

## 引言

启用某插件后台 502 了

## 进入数据库

1. 选择进入 wp_options 表

2. 找到 active_plugins 条目，一般在第二页

3. 编辑此项目的 option_value 行

## 删除不要的插件

***注意：删除前先备份！！！***

1. 找到不需要的插件的名字

   删除从 `i` 开始到 `;`，例如 `i:1;s:23:"elementor/elementor.php";`

2. 更改序号，也就是 `i,` 后的数字

3. 更改总数，也就是最开头 `a:` 后的数字

## 参考文章

[禁用数据库中的一个WordPress插件 - WordPress - GoDaddy 帮助 SG](https://sg.godaddy.com/zh/help/disable-one-wordpress-plugin-from-the-database-41199)