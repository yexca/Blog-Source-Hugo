---
slug: 53
title: 使用 Git 上传文件至 Github
date: '2022-08-03T12:49:27+08:00'
author: hiyoung
# layout: post
# permalink: /archives/53
views:
    - '231'
categories:
    - 软件工具
    - 编码相关
tags:
    - Git
    - Github
---

> 该文章由 [Hiyoung](https://blog.hiyoung.xyz/) 编写
> 
> 其文章: <https://blog.hiyoung.xyz/2022/08/03/0b5e2db181ab/>

最近经常要使用Github保存我学习过程中的代码，发现无法直接上传文件夹，遂在网上查了一下使用Git上传，所以写个博文记录一下

## Github端操作

### 1. 复制仓库地址

![使用Git上传文件至Github_1](https://cdn.statically.io/gh/hiyoung3937/img_hiyoung@master/bolg/%E4%BD%BF%E7%94%A8Git%E4%B8%8A%E4%BC%A0%E6%96%87%E4%BB%B6%E8%87%B3Github_1.3syztscmys80.jpg)

## 本地端操作

### 1. 在本地新建一个空文件夹

![使用Git上传文件至Github_2](https://cdn.statically.io/gh/hiyoung3937/img_hiyoung@master/bolg/%E4%BD%BF%E7%94%A8Git%E4%B8%8A%E4%BC%A0%E6%96%87%E4%BB%B6%E8%87%B3Github_2.5nufudqca7w0.jpg)

<center style="font-size:15px;color:#FFFFF;text-decoration:underline">我这里已经clone完成</center>### 2. 在文件夹内呼出Git Bash框

![使用Git上传文件至Github_3](https://cdn.statically.io/gh/hiyoung3937/img_hiyoung@master/bolg/%E4%BD%BF%E7%94%A8Git%E4%B8%8A%E4%BC%A0%E6%96%87%E4%BB%B6%E8%87%B3Github_3.5l2lii1fkd80.jpg)

### 3. Clone远程仓库

```
<pre class="language-bash" data-info="bash" data-role="codeBlock"><span class="token function">git</span> clone + 你的仓库地址
<span class="token function">git</span> clone https://github.com/hiyoung3937/study_code.git  //示例
```

### 4. 直接将需要上传的文件拖入即可

### 5. 上传

```
<pre class="language-bash" data-info="bash" data-role="codeBlock"><span class="token builtin class-name">cd</span>  study_code.git   //根据自己的远程仓库名输入
<span class="token function">git</span> init
<span class="token function">git</span> <span class="token function">add</span> <span class="token builtin class-name">.</span>
<span class="token function">git</span> commit -m “你的提交信息”
<span class="token function">git</span> push
```

- - - - - -

## 命令说明

| clone + 仓库地址 | 克隆你的仓库至本地 |
|---|---|
| cd + 你的远程仓库名 | 进入到远程仓库内(根据自己的仓库名输入) |
| git init | 初始化Git |
| git add . | 将工作区的文件添加到暂存区（”.”是当前目录下的所有文件，也可只输入文件夹名称） |
| git commit -m “你的提交信息” | 将暂存区的文件添加到本地仓库 |
| git push | 提交到远程仓库（可能需要你输入帐号和密码） |