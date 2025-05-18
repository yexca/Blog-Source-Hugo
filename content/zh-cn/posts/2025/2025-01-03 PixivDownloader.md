---
slug: 211
title: 'Pixiv 下载器'
# draft: true
author: yexca
date: '2025-01-03T20:05:44+09:00'
lastmod: '2025-05-18T16:20:33+09:00'
categories:
    - 开发实践
tags:
    - Pixiv
    - Python
    - PyQt6
---

> 2025-05-18 更新  
> 我写了个 SQLite 版的，不用配置数据库了，详情访问: <https://blog.yexca.net/archives/248>

耗时三天写出来了个大概能用的版本，不过没有做错误处理 ~~遇到错误直接重启吧~~

项目地址: <https://github.com/yexca/PixivDownloader-MySQL>

## 引言

这要从[数据库记录已下载画师作品](https://blog.yexca.net/archives/94/)开始说起了，当时我整了个数据库记录我下载过的作品，时间久了之后，觉得这玩意是在做重复作业啊，说到重复作业那必然是计算机来做啊，正好最近不经意间产生了做程序的想法，也正好对其不满意: <https://github.com/yexca/yasumiProject>，同时又是过年比较空闲，这就开写

## 说明

虽说我是写出来了，不过没有错误处理之类的，只能说勉强能用吧。同时代码很乱 (第一次开发比较大的 GUI 软件啦)，乱到我不想去整理和做国际化支持了

并且开发过程中突然想到都写程序了为什么不用 SQLite 呢，这还要开一个 MySQL 多麻烦，不过都做了，就做到最后吧

最后本来想打包的，因为使用配置文件，打包好麻烦 ~~(刚开始不知道打包出来被 Windows 当病毒了来着)~~，我实在懒得折腾了，就这样吧

## 界面

背景图: <https://www.pixiv.net/artworks/83273073>

- 首页

![home](https://github.com/yexca/picx-images-hosting/raw/master/2025/01-PixivDownloader/home.4ckyo63bny.webp)

- 设置

![settings](https://github.com/yexca/picx-images-hosting/raw/master/2025/01-PixivDownloader/settings.5fknz20gje.webp)

## 配置

因为基于我现有数据库开发，所以几乎没有自定义程度，数据库建表语句为

```sql
CREATE TABLE pic (
    ID varchar(99),
    # 唯一标识
    name varchar(255),
    # 画师昵称
    downloadedDate datetime,
    # 下载/更新时间
    lastDownloadID varchar(255),
    # 最新作品ID
    platform varchar(50),
    # 平台
    url varchar(255),
    # 链接
    PRIMARY KEY(ID)
)ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;
```

然后因为 API 的使用需要登陆并且不可以用账号密码登录，根据 <https://gist.github.com/ZipFile/c9ebedb224406f4f11845ab700124362> 获取 Auth Token 后使用

背景图片我并没有上传到 git 仓库，路径为 `app\resources\images\background.png`

然后是需要安装 Python，安装依赖

```bash
pip install -r requirements.txt
```

## 使用

运行程序

```bash
python main.py
```

在 Pixiv 验证界面 和 设置 界面完成相关配置后返回首页

填入画师 ID **或者** 某一作品的 ID 就会自动爬取该画师全部作品了

## 结尾

我其实都觉得自己都不去使用它，这算是我开发经历的一小步吧

![yexca-211](https://count.getloli.com/@yexca-211)
