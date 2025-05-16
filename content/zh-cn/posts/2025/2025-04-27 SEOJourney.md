---
slug: 246
title: '为了能搜到自己的名字：一次小站 SEO 排查实践'
# draft: true
author: yexca
date: '2025-04-27T17:38:16+09:00'
categories:
    - 技术实践
tags:
    - SEO
---

## 引言

又一次的自我介绍时，我不由得又想到了我的博客

过去我总是直接打开 Google 搜索 `yexca` 点开第一个结果，进入我的博客  
但自从更换域名以来，无数次搜索 `yexca` ，我的博客却始终消失在搜索之外

起初我并没有太在意，以为可能有一些惩罚机制吧，毕竟 Google 建议更换域名后最好做一年的 301 重定向，而我当时只做了半年就到原域名过期时间了

但，已经两年了吧，再怎么说，也该恢复了吧？

而且更离谱的是，排名靠前的反倒是一些早就不再维护的网站  
而我，每天更新、调优、折腾着的这个博客，却仿佛被世界遗忘了一样

## 于是，我开始寻找原因

打开我的博客，查看 `<head>` 部分  
嗯？`<meta name='description'>` 怎么是页面左边那句标语？

啊这，配置的时候也只是说会出现在那里，我以为和 Argon 的配置类似呢，那么也就是说，这个站点的描述根本意义不明啊

不过，这句话几乎陪伴了我整个博客生涯，我不想轻易放弃它  
既然如此，就让 JSON-LD 来承担结构化描述的任务吧！

于是我在主题自定义 `<head>` 部分，加入

```html
<script type="application/ld+json">
{
  "@context": "https://schema.org",
  "@type": "Blog",
  "name": "yexca'Blog",
  "url": "{{ .Site.BaseURL }}",
  "inLanguage": "{{ .Site.Language.Lang }}",
  "description": "{{ .Site.Params.description }}",
  "author": {
    "@type": "Person",
    "name": "yexca"
  }
}
</script>
```

嗯，同时考虑到是多语言网站，部分内容当然要使用变量适配比较好

## 在语言的迷宫中探索

说到多语言网站，那么我换个语言环境搜索的话如何呢？  
于是我在 `google.com.hk` 、 `google.com.tw` 、 `google.com.jp` 搜索 `yexca`

结果日文环境是可以搜到我的博客的，但是中文环境是搜不到的，对于英文环境，因为英文也没几个文章，也无所谓吧

那就很奇怪了，也就是 Google 应该是可以标识到 <https://blog.yexca.net> 就是 `yexca` 吧，但其他语言版本怎么那么惨捏

继续排查下去，我意识到可能还缺少 hreflang 配置，于是我加上了

```html
<link rel="alternate" hreflang="x-default" href="https://blog.yexca.net/" />
<link rel="alternate" hreflang="zh-CN" href="https://blog.yexca.net/" />
<link rel="alternate" hreflang="zh-TW" href="https://blog.yexca.net/zh-tw/" />
<link rel="alternate" hreflang="ja" href="https://blog.yexca.net/ja/" />
<link rel="alternate" hreflang="en" href="https://blog.yexca.net/en/" />
```

明确告诉搜索引擎，不同语言的用户可以访问不同语言版本，这都是同一个网站

> 顺便，这段 `<link>` 也会出现在文章页面，因为没有做判断，但我的博客并不是所有文章都有全部语言版本，所以这并不需要改变，Google 也会自动理解的

## 一点点地补上遗漏

但是我随意打开一个文章，哎呀，上面还是 JSON-LD 的站点介绍，这多少有点奇怪  
于是，我又做了个判断逻辑

```html
{{ if .IsHome }}
  <!-- 首页 JSON-LD -->
  <script type="application/ld+json">
  {
    "@context": "https://schema.org",
    "@type": "Blog",
    "name": "yexca'Blog",
    "url": "{{ .Site.BaseURL }}",
    "inLanguage": "{{ .Site.Language.Lang }}",
    "description": "{{ .Site.Params.description }}",
    "author": {
      "@type": "Person",
      "name": "yexca"
    }
  }
  </script>

{{ else if .IsPage }}
  <!-- 文章页 JSON-LD -->
  <script type="application/ld+json">
  {
    "@context": "https://schema.org",
    "@type": "BlogPosting",
    "mainEntityOfPage": {
      "@type": "WebPage",
      "@id": "{{ .Permalink }}"
    },
    "headline": "{{ .Title }}",
    "name": "yexca'Blog",
    "author": {
      "@type": "Person",
      "name": "yexca"
    },
    "publisher": {
      "@type": "Person",
      "name": "yexca'Blog"
    },
    "datePublished": "{{ .Date.Format "2006-01-02" }}",
    "dateModified": "{{ .Lastmod.Format "2006-01-02" }}",
    "inLanguage": "{{ .Site.Language.Lang }}",
    "description": "{{ .Params.description | default .Site.Params.description }}"
  }
  </script>
{{ end }}
```

这样首页和文章都会有不同的 JSON-LD，不仅语义更准确，也更符合 Google 的结构化数据推荐规范

## 小小的期盼

现在，一切终于完善了  
虽然效果不会立刻显现，但我知道，信号，已经发出去了

现在唯一需要做的，就是等待，等待 Google 再一次理解我的存在

我希望，下次在介绍我博客的时候  
可以直接打开 Google，搜索 `yexca`
