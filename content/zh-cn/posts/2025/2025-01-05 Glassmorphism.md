---
slug: 212
title: '毛玻璃效果'
# draft: true
author: yexca
date: '2025-01-05T16:19:36+09:00'
lastmod: '2025-02-26T22:40:36+09:00'
categories:
    - 开发实践
tags: 
    - 前端技术
---

## 引言

今天想对最近设计的半透明、毛玻璃和圆角进行总结来着，突然想到 2023-12-01 貌似做过一个什么东西，这便顺便也重整一下得了

## 页面背景

现代的 ~~(二次元)~~ 网页要有一个背景

```css
body{
    background-image: url('../img/77194247_p0.jpg');
    background-size: cover;
    background-attachment: fixed; /* 固定 */
    background-repeat: no-repeat; /* 不重复 */
    padding: 0;
    margin: 0;
}
```

## 半透明与毛玻璃

然后在背景上加一个蒙版，实现半透明与毛玻璃效果

```css
.layout {
    /* display: flex; */
    margin: 0;
    padding: 0;
    display: flex;
    width: 100vw;
    height: 100vh;
    backdrop-filter: blur(2px); /* 背景模糊效果 */
    -webkit-backdrop-filter: blur(2px); /* Safari 支持 */
    background: rgba(0, 0, 0, 0.4); /* 半透明黑色背景 */
}
```

## 网页的重构

说起来写文章还是很烦恼的，做的时候写会打扰，做完写又会很累不想写，所以我折中下，随便写写就好

项目地址: <https://github.com/yexca/MusicPlayer-Twinkle>

顺便更新了下之前的文章 <https://blog.yexca.net/archives/116/> 用此方法加了个示例: <https://twinkle.yexca.net>

2025-02-26: 我翻到当时是参考哪里啦: <https://blog.csdn.net/Wang_x_y_/article/details/123693127>

## 卡片效果

这也属于现代的设计

```css
.card {
  width: 300px;
  padding: 20px;
  background-color: rgba(255, 255, 255, 0.5); /* 半透明白色背景 */
  border-radius: 15px; /* 圆角 */
  box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1); /* 阴影 */
  backdrop-filter: blur(10px); /* 背景模糊效果 */
  border: 1px solid rgba(255, 255, 255, 0.3); /* 边框 */
  color: pink; /* 前景文字颜色 */
}
```

嗯，之后有时间以卡片为设计完善此项目 ~~(又开新坑)~~

## Twinkle

另外项目的内容为 Twinkle 的音乐，具体介绍可见

* <https://twinkle130527.wixsite.com/twinkle>
* <https://www.dojin-music.info/circle/1255>

SoundCloud

* <https://soundcloud.com/twintwinkles>
