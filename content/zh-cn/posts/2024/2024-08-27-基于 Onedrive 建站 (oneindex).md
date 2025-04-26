---
slug: 180
# layout: post
title: '基于 Onedrive 建站 (oneindex)'
author: yexca
date: 2024-08-27T11:00:24+08:00
# permalink: /archives/180
categories:
    - 开发实践
tags:
    - OneDrive
    - 建站实践
---  

文章写于 2022.05.09，文章并无最新适配，可能无法复现。另推荐使用较新的项目

通过 Onedrive 可以建立网盘网站，例如~~我的VRChat网盘~~已经没了，还可以直接建立网页，或者图床

## oneindex 简介

Github: <https://github.com/avedu/oneindex>

Onedrive Directory Index，不占用服务器空间，不走服务器流量，直接列出 OneDrive 目录，文件直链下载。

## 环境

需要 PHP5.6+，需打开 curl 支持，新手(~~比如我~~)建议直接使用[宝塔](https://bt.cn)直接部署，方便快捷

另因宝塔隐私泄露风波，有一些其他版本如[宝塔纯净版](https://support.hostcli.com/)，请自行识别安装

## 安装

从 Github 将仓库下载并上传至网站根目录并解压缩

然后访问网站，进入安装程序

### 检测

首先是同意条款，如果点击同意会返回，请将地址栏中最后的 `&mdui-dialog` 删除后按回车

如果环境正常点击`下一步`

### 程序安装

点击`获取应用ID和机密(分两个页面显示，请注意保存)`，然后登陆微软账号

保持出现的`应用机密`，然后点击`知道了，返回快速启动`

**注意**：此机密仅会显示一次，请妥善保存

然后选择一门语言，比如`Python`，点击`Get a client ID`，复制获得的`Client ID`

返回安装程序界面，输入`应用机密`和`Client ID`，然后点击`下一步`

点击`绑定账号`，选择`接受`即可

**如果出现错误**

请返回输入`应用机密`和`Client ID`的界面，打开 [Azure 的应用注册](https://portal.azure.com/#blade/Microsoft_AAD_IAM/ActiveDirectoryMenuBlade/RegisteredApps)，这里应该有俩应用

找到名为 `oneindex` 的应用，复制它的`应用程序(客户端)ID`填入`Client ID`

还有一个没什么用，可以直接删除

## 管理

安装完成后会有`管理后台`和`访问网站`选项

可以进入管理后台修改网站名称，主题，后台密码等 (初始密码: oneindex)

后台网址为`您的域名/?/admin`

## 伪静态

设置 apache 或者 nginx 的 rewrite (使用 wordpress 的配置即可)

以上为原作者的原话(部分内容有修改)，因我使用[宝塔](https://bt.cn)，遂说明宝塔如何配置

在宝塔面板的`网站`进入设置里`伪静态`，选择`wordpress`保存即可

在网站管理后台将`伪静态`勾选保存即可

这样链接中`?`会去除，访问后台可直接`您的域名/admin`

## 特殊文件实现功能

~~Markdown 语法可参考我写的文章~~：[Markdown 笔记](https://blog.yexca.net/archives/43)

**在文件夹底部添加说明:** 

> 在 OneDrive 的文件夹中添加`README.md`文件，使用 Markdown 语法。

**在文件夹头部添加说明:** 

> 在 OneDrive 的文件夹中添加`HEAD.md` 文件，使用 Markdown 语法。

**加密文件夹:** 

> 在 OneDrive 的文件夹中添加`.password`文件，填入密码，密码不能为空。  

**直接输出网页:**

> 在 OneDrive 的文件夹中添加`index.html` 文件，程序会直接输出网页而不列目录。
> 配合 文件展示设置-直接输出 效果更佳。

因为直接输出网页，可以直接搭网站

## 访问其他文件正常，但访问图片出现404

这是由于服务器软件 (Nginx/Apache) 接管了图片处理，删除相关配置即可

以下为使用宝塔

在`网站`-`配置文件`将下述代码注释掉即可

```nginx
location ~ .*\.(gif|jpg|jpeg|png|bmp|swf)$
{
  expires      30d;
  access_log on; 
}
```

改为

```nginx
#location ~ .*\.(gif|jpg|jpeg|png|bmp|swf)$
#{
#  expires      30d;
#  access_log on; 
#}
```

此问题参考[部署 OneDrive for business (PHP)客户端程序 OneIndex 详细教程 - VirCloud's Blog - Learning&Sharing](https://vircloud.net/media/oneindex.html#selection-997.0-997.84)

## 命令行功能

仅能在 PHP CLI 模式下运行

**清除缓存:** 

```
php one.php cache:clear
```

**刷新缓存:** 

```
php one.php cache:refresh
```

**刷新令牌:** 

```
php one.php token:refresh
```

**上传文件:** 

```
php one.php upload:file 本地文件 [OneDrive文件]
```

**上传文件夹:**

```
php one.php upload:folder 本地文件夹 [OneDrive文件夹]
```

例如：

```bash
//上传demo.zip 到OneDrive 根目录  
php one.php upload:file demo.zip  

//上传demo.zip 到OneDrive /test/目录  
php one.php upload:file demo.zip /test/  

//上传demo.zip 到OneDrive /test/目录并将其命名为 d.zip  
php one.php upload:file demo.zip /test/d.zip  

//上传up/ 到OneDrive /test/ 目录  
php one.php upload:file up/ /test/
```

## 计划任务

[可选]**推荐配置**，非必需。后台定时刷新缓存，可增加前台访问的速度。

```
# 每小时刷新一次token
0 * * * * /具体路径/php /程序具体路径/one.php token:refresh

# 每十分钟后台刷新一遍缓存
*/10 * * * * /具体路径/php /程序具体路径/one.php cache:refresh
```

## Docker安装运行

请参考 [TimeBye/oneindex](https://github.com/TimeBye/oneindex)