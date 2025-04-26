---
slug: 67
title: 萌百黑幕在 Argon 主题 WordPress 实现
date: '2022-09-05T21:51:23+08:00'
author: yexca
# layout: post
# permalink: /archives/67
views:
    - '256'
categories:
    - 开发实践
tags:
    - WordPress
---

## 引言

这个黑幕很好玩啊，非常好玩啊，可惜首页无法渲染出来，而且 Markdown 编写渲染也难

## 使用

编写文章时选择*作为HTML编辑*，插入以下语句

```html
<span class="heimu" title="黑幕弹框里的字">需要隐藏的文字</span>
```

## 插入CSS

本是想着实现首页也有黑幕，但实际测试发现首页不会渲染 ~~(为什么不渲染啊啊啊啊啊啊啊！！！！！！！试试想象 Warma 的声音)~~

进入后台设置，找到*页脚*设置，输入以下代码，或者在WP的自定义CSS处插入，不过需要去掉标签

```html
<style>
.heimu, .heimu a, a .heimu, .heimu a.new 
{
  background-color: #252525;
  color: #252525;
  text-shadow: none;
}
.heimu:hover, .heimu:active,
.heimu:hover .heimu, .heimu:active .heimu 
{
  color: white !important;
}
.heimu:hover a, a:hover .heimu,
.heimu:active a, a:active .heimu 
{
  color: lightblue !important;
}
.heimu:hover .new, .heimu .new:hover, .new:hover .heimu,
.heimu:active .new, .heimu .new:active, .new:active .heimu 
{
  color: #BA0000 !important;
}
</style>
```

注：因Argon不会渲染注释，所以我不把以下内容放入代码中：

`/*阅读更多：https://zh.moegirl.org/MediaWiki:Mobile.css 本文引自萌娘百科(https://zh.moegirl.org)，文字内容默认使用《知识共享 署名-非商业性使用-相同方式共享 3.0》协议。*/`

## 参考文章

[Re：萌娘百科上的黑幕实现 – Vanilla\_chan – 博客园](https://www.cnblogs.com/Vanilla-chan/p/12355387.html)

[萌百黑幕CSS代码-Hiyoung’blog](https://hiyoungssr.xyz/2022/08/22/%E8%90%8C%E7%99%BE%E9%BB%91%E5%B9%95CSS%E4%BB%A3%E7%A0%81/)
