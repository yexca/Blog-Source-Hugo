---
slug: 68
title: Linux 下载 youtube 视频
date: '2022-09-10T19:29:58+08:00'
author: yexca
# layout: post
# permalink: /archives/68
views:
    - '330'
categories:
    - Linux
    - LinuxTools
tags:
    - Youtube
---

## 引言

既然都有Win命令行了，那怎么能没有Linux呢

Windows的文章：[命令行下载YouTube视频](http://blog.yexca.net/archives/52)

## 下载yt-dlp

建议配置好Python环境，然后到[Releases · yt-dlp/yt-dlp · GitHub](https://github.com/yt-dlp/yt-dlp/releases)下载*yt-dlp*，如果不想配置Python就下载*yt-dlp_linux*

下载完成后赋予执行权限放在`/usr/local/bin/`下

## 下载ffmpeg

参考官网[Download FFmpeg](https://ffmpeg.org/download.html#build-linux)

Fedora下使用以下命令

```bash
sudo dnf install ffmpeg
```

## 配置文件

切换配置目录

```bash
cd ~/.config
```

创建文件夹并进入

```bash
mkdir yt-dlp
cd yt-dlp
```

创建配置文件

```bash
vi config
```

我的配置文件如下

```bash
-f bv+ba/b -o ~/Videos/%(uploader)s/%(upload_date)s%(title)s%(id)s.%(ext)s --continue --merge-output-format mp4
```

解释一下

```bash
-f bv+ba/b # 最高画质和音频

-o # 输出文件夹配置
/%(uploader)s/ # 按频道名创建文件夹
%(upload_date)s # 上传时间
%(title)s # 视频标题
%(id)s # 视频id
.%(ext)s # 视频拓展名

--continue # 断点续传

--merge-output-format mp4 # 混合为mp4视频
```

## 参考文章

[ffmpeg批量转换视频格式](http://blog.yexca.net/archives/65) ~~话说自己的文章有必要放链接吗~~