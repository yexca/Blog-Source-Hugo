---
slug: 47
title: 原神自动签到 (Linux 服务器 Docker)
date: '2022-06-09T13:33:42+08:00'
author: yexca
# layout: post
# permalink: /archives/47
views:
    - '700'
categories:
    - 日常
tags:
    - Docker
    - miHoYo
---

# 引言

由于腾讯云函数从六月开始收费，于是便弃用改在自己的服务器上搭建

既然六月收费为什么现在才写文章呢？因为~~可能还有三个月的免费试用~~我米游社的Cookie过期更换，故记录

# 工具&原教程

[原神签到小助手 每日福利不用愁 - 银弹博客 (yindan.me)](https://www.yindan.me/tutorial/genshin-impact-helper.html)

由于原文介绍了多种使用方法，自己的阅读体验不是太好，故写此文

# 腾讯云函数处理

请将腾讯云函数冻结已确保不会收取费用

当然，如果没什么其他需求可直接注销账号，但注销账号需要手持身份证照片，请注意

# 前提

服务器可以连接上米忽悠的服务器<https://mihoyo.com>

可在SSH命令行窗口输入`ping mihoyo.com`测试是否可以连接

~~我的一个服务器就连接不上，只好换一个，唉~~

# Docker安装

可以直接使用一键脚本进行安装，实测Debian10和CentOS7正常安装(请使用root账户)

安装命令如下：

```bash
curl -fsSL https://get.docker.com | bash -s docker --mirror Aliyun
```

也可以使用国内 daocloud 一键安装命令：

```bash
curl -sSL https://get.daocloud.io/docker | sh
```

# 脚本安装

使用以下命令即可

```bash
docker pull yindan/genshinhelper
```

## CentOS错误

我使用CentOS安装时出现错误`Can't Connect to Docker Daemon`

请确保使用root账户，然后输入以下命令

```bash
systemctl start docker
```

# 简易使用

## Cookie获取

获取米游社Cookie请参考：[原神树脂查看/推送 – yexca'Blog](https://yexca.xyz/archives/12)

**注意**：Cookie 应包含`account_id`和`cookie_token`两个字段

多账号在不同Cookie中间加`#`即可，例如`Cookie1#Cookie2#Cookie3`

## 简易配置

```bash
docker run -d --name=genshinhelper \
-e COOKIE_MIHOYOBBS="<COOKIE_MIHOYOBBS>" \
--restart always \
yindan/genshinhelper:latest
```

将自己的`Cookie`替换上述命令的`<COOKIE_MIHOYOBBS>`即可

## 重新配置/更新Cookie

重新配置好像需要卸载再重装，然后再进行配置

或者使用配置文件只需替换Cookie就可以了吧(没用过，~~Cookie有效期很长的~~)

## 常用命令

```bash
# 查看Docker所有的容器
docker ps -a

# 查看日志
docker logs -f genshinhelper --tail 100

# 重启
docker restart genshinhelper

# 更新
docker pull yindan/genshinhelper
docker rm -f genshinhelper
# 之后依据基本使用或高级使用重新部署

# 卸载
docker rm -f genshinhelper
docker image rm genshinhelper
```

# 配置文件

### 示例配置文件

> * Github: [config.json](https://gitlab.com/y1ndan/genshin-checkin-helper/-/raw/main/genshincheckinhelper/config/config.example.json)
> * Telegram: https://t.me/genshinhelperupdates/5

可下载示例文件修改

## 安装

假设配置文件位于服务器的 `/etc/genshin/config.json`，使用以下命令已映射配置

```bash
docker run -d --name=genshinhelper \
-v /etc/genshin:/app/genshincheckinhelper/config \
--restart always \
yindan/genshinhelper:latest
```

## 配置

配置文件可以只留下需要的参数，把非必须的参数删除，例如只需要`Cookie`

则配置文件除了保持完整也可以写成：

```json
{
  "COOKIE_MIHOYOBBS": "<COOKIE_MIHOYOBBS>",
}
```

## 配置文件新增

RANDOM_SLEEP_SECS_RANGE：随机延迟休眠秒数范围，单位：秒。设置成"0-0"为取消延迟。
CHECK_IN_TIME：每日签到时间。该时间和运行环境的时间有关，和时区无关。如果是docker，可以用TZ=Asia/Shanghai设置时区。
CHECK_RESIN_SECS：原神原粹树脂检测间隔时间，单位：秒。
COOKIE_RESIN_TIMER：需要开启原粹树脂检测账号的cookie。
SHOPTOKEN：微信积分商城的token，通过抓包获取。
ONEPUSH：推送配置。notifier为推送名字，params为所需参数。详见后文。

# OnePush推送参数一览

推送名称 / notifier: bark  
参数大全 / params:  
{'required': ['key'], 'optional': ['title', 'content', 'sound', 'isarchive', 'icon', 'group', 'url', 'copy', 'autocopy']}

推送名称 / notifier: custom  
参数大全 / params:  
{'required': ['url'], 'optional': ['method', 'datatype', 'data']}

推送名称 / notifier: dingtalk  
参数大全 / params:  
{'required': ['token'], 'optional': ['title', 'content', 'secret', 'markdown']}

推送名称 / notifier: discord  
参数大全 / params:  
{'required': ['webhook'], 'optional': ['title', 'content', 'username', 'avatar_url', 'color']}

推送名称 / notifier: pushplus  
参数大全 / params:  
{'required': ['token', 'content'], 'optional': ['title', 'topic', 'markdown']}

推送名称 / notifier: qmsg  
参数大全 / params:  
{'required': ['key'], 'optional': ['title', 'content', 'mode', 'qq']}

推送名称 / notifier: serverchan  
参数大全 / params:  
{'required': ['sckey', 'title'], 'optional': ['content']}

推送名称 / notifier: serverchanturbo  
参数大全 / params:  
{'required': ['sctkey', 'title'], 'optional': ['content', 'channel', 'openid']}

推送名称 / notifier: telegram  
参数大全 / params:  
{'required': ['token', 'userid'], 'optional': ['title', 'content', 'api_url']}

推送名称 / notifier: wechatworkapp  
参数大全 / params:  
{'required': ['corpid', 'corpsecret', 'agentid'], 'optional': ['title', 'content', 'touser', 'markdown']}

推送名称 / notifier: wechatworkbot  
参数大全 / params:  
{'required': ['key'], 'optional': ['title', 'content', 'markdown']}

**例子**
telegram  
ONEPUSH={"notifier":"telegram","params":{"markdown":false,"token":"xxxx","userid":"xxx"}}

discord  
ONEPUSH={"notifier":"discord","params":{"markdown":true,"webhook":"https://discord.com/api/webhooks/xxxxxx"}}

docker 配置文件映射目录为：/etc/genshin:/app/genshincheckinhelper/config

# 所有变量

<table><thead><tr><th><strong>Variable Name</strong></th><th align="center"><strong>Required</strong></th><th align="center"><strong>Default</strong></th><th><strong>Description</strong></th></tr></thead><tbody><tr><td>LANGUAGE</td><td align="center">❌</td><td align="center">en</td><td>项目语言。目前支持中文(zh)和英文(en)。</td></tr><tr><td>MAX_SLEEP_SECS</td><td align="center">❌</td><td align="center">300</td><td>最大休眠秒数。自v1.5.0添加了运行前随机延迟，设置此参数可自定义延迟，秒数应该＞10</td></tr><tr><td>RUN_ENV</td><td align="center">❌</td><td align="center">prod</td><td>运行环境。设置为任意非默认值即可跳过随机延迟。</td></tr><tr><td>COOKIE_MIHOYOBBS</td><td align="center">❌</td><td align="center"> </td><td>Cookie from miHoYo bbs. <span class="external-link"><a class="no-external-link" href="https://bbs.mihoyo.com/ys/" target="_blank">https://bbs.mihoyo.com/ys/<svg xmlns="http://www.w3.org/2000/svg" width="16px" height="16px" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="feather feather-external-link"><path d="M18 13v6a2 2 0 0 1-2 2H5a2 2 0 0 1-2-2V8a2 2 0 0 1 2-2h6"></path><polyline points="15 3 21 3 21 9"></polyline><line x1="10" y1="14" x2="21" y2="3"></line></svg></a></span></td></tr><tr><td>COOKIE_BH3</td><td align="center">❌</td><td align="center"> </td><td>和 COOKIE_MIHOYOBBS 一样</td></tr><tr><td>COOKIE_MIYOUBI</td><td align="center">❌</td><td align="center"> </td><td>Cookie from miHoYo bbs. <span class="external-link"><a class="no-external-link" href="https://bbs.mihoyo.com/ys/" target="_blank">https://bbs.mihoyo.com/ys/<svg xmlns="http://www.w3.org/2000/svg" width="16px" height="16px" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="feather feather-external-link"><path d="M18 13v6a2 2 0 0 1-2 2H5a2 2 0 0 1-2-2V8a2 2 0 0 1 2-2h6"></path><polyline points="15 3 21 3 21 9"></polyline><line x1="10" y1="14" x2="21" y2="3"></line></svg></a></span></td></tr><tr><td>COOKIE_HOYOLAB</td><td align="center">❌</td><td align="center"> </td><td>Cookie from HoYoLAB community. <span class="external-link"><a class="no-external-link" href="https://webstatic-sea.mihoyo.com/ys/event/signin-sea/index.html?act_id=e202102251931481&amp;lang=en-us" target="_blank">https://webstatic-sea.mihoyo.com/ys/event/signin-sea/index.html?act_id=e202102251931481&amp;lang=en-us<svg xmlns="http://www.w3.org/2000/svg" width="16px" height="16px" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="feather feather-external-link"><path d="M18 13v6a2 2 0 0 1-2 2H5a2 2 0 0 1-2-2V8a2 2 0 0 1 2-2h6"></path><polyline points="15 3 21 3 21 9"></polyline><line x1="10" y1="14" x2="21" y2="3"></line></svg></a></span></td></tr><tr><td>COOKIE_WEIBO</td><td align="center">❌</td><td align="center"> </td><td>Parameters from Sina Weibo intl. app.   aid=xxx; gsid=xxx; s=xxx; from=xxx</td></tr><tr><td>COOKIE_KA</td><td align="center">❌</td><td align="center"> </td><td>Cookie from <span class="external-link"><a class="no-external-link" href="https://m.weibo.cn" target="_blank">https://m.weibo.cn<svg xmlns="http://www.w3.org/2000/svg" width="16px" height="16px" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="feather feather-external-link"><path d="M18 13v6a2 2 0 0 1-2 2H5a2 2 0 0 1-2-2V8a2 2 0 0 1 2-2h6"></path><polyline points="15 3 21 3 21 9"></polyline><line x1="10" y1="14" x2="21" y2="3"></line></svg></a></span></td></tr><tr><td>BARK_KEY</td><td align="center">❌</td><td align="center"> </td><td>iOS Bark app's IP or device code. For example: <span class="external-link"><a class="no-external-link" href="https://api.day.app/xxxxxx" target="_blank">https://api.day.app/xxxxxx<svg xmlns="http://www.w3.org/2000/svg" width="16px" height="16px" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="feather feather-external-link"><path d="M18 13v6a2 2 0 0 1-2 2H5a2 2 0 0 1-2-2V8a2 2 0 0 1 2-2h6"></path><polyline points="15 3 21 3 21 9"></polyline><line x1="10" y1="14" x2="21" y2="3"></line></svg></a></span></td></tr><tr><td>BARK_SOUND</td><td align="center">❌</td><td align="center">healthnotification</td><td>iOS Bark app's notification sound. Default: healthnotification</td></tr><tr><td>COOL_PUSH_SKEY</td><td align="center">❌</td><td align="center"> </td><td>SKEY for Cool Push. <span class="external-link"><a class="no-external-link" href="https://cp.xuthus.cc/" target="_blank">https://cp.xuthus.cc/<svg xmlns="http://www.w3.org/2000/svg" width="16px" height="16px" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="feather feather-external-link"><path d="M18 13v6a2 2 0 0 1-2 2H5a2 2 0 0 1-2-2V8a2 2 0 0 1 2-2h6"></path><polyline points="15 3 21 3 21 9"></polyline><line x1="10" y1="14" x2="21" y2="3"></line></svg></a></span></td></tr><tr><td>COOL_PUSH_MODE</td><td align="center">❌</td><td align="center">send</td><td>Push method for Cool Push. Choose from send(私聊),group(群组),wx(微信). Default: send</td></tr><tr><td>CRON_SIGNIN</td><td align="center">❌</td><td align="center">0 6 <em> </em> *</td><td>Docker custom runtime</td></tr><tr><td>CUSTOM_NOTIFIER</td><td align="center">❌</td><td align="center"> </td><td>Custom notifier configuration</td></tr><tr><td>DD_BOT_TOKEN</td><td align="center">❌</td><td align="center"> </td><td>钉钉机器人WebHook地址中access_token后的字段.</td></tr><tr><td>DD_BOT_SECRET</td><td align="center">❌</td><td align="center"> </td><td>钉钉加签密钥.在机器人安全设置页面,加签一栏下面显示的以SEC开头的字符串.</td></tr><tr><td>DISCORD_WEBHOOK</td><td align="center">❌</td><td align="center"> </td><td>Webhook of Discord.</td></tr><tr><td>IGOT_KEY</td><td align="center">❌</td><td align="center"> </td><td>KEY for iGot. For example: <span class="external-link"><a class="no-external-link" href="https://push.hellyw.com/xxxxxx" target="_blank">https://push.hellyw.com/xxxxxx<svg xmlns="http://www.w3.org/2000/svg" width="16px" height="16px" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="feather feather-external-link"><path d="M18 13v6a2 2 0 0 1-2 2H5a2 2 0 0 1-2-2V8a2 2 0 0 1 2-2h6"></path><polyline points="15 3 21 3 21 9"></polyline><line x1="10" y1="14" x2="21" y2="3"></line></svg></a></span></td></tr><tr><td>PUSH_PLUS_TOKEN</td><td align="center">❌</td><td align="center">一对一推送</td><td>pushplus 一对一推送或一对多推送的token.不配置push_plus_user则默认为一对一推送. <span class="external-link"><a class="no-external-link" href="https://www.pushplus.plus/doc/" target="_blank">https://www.pushplus.plus/doc/<svg xmlns="http://www.w3.org/2000/svg" width="16px" height="16px" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="feather feather-external-link"><path d="M18 13v6a2 2 0 0 1-2 2H5a2 2 0 0 1-2-2V8a2 2 0 0 1 2-2h6"></path><polyline points="15 3 21 3 21 9"></polyline><line x1="10" y1="14" x2="21" y2="3"></line></svg></a></span></td></tr><tr><td>PUSH_PLUS_USER</td><td align="center">❌</td><td align="center"> </td><td>pushplus 一对多推送的群组编码.在'一对多推送'-&gt;'您的群组'(如无则新建)-&gt;'群组编码'里查看,如果是创建群组人,也需点击'查看二维码'扫描绑定,否则不能接收群组消息.</td></tr><tr><td>SCKEY</td><td align="center">❌</td><td align="center"> </td><td>SCKEY for ServerChan. <span class="external-link"><a class="no-external-link" href="https://sc.ftqq.com/3.version/" target="_blank">https://sc.ftqq.com/3.version/<svg xmlns="http://www.w3.org/2000/svg" width="16px" height="16px" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="feather feather-external-link"><path d="M18 13v6a2 2 0 0 1-2 2H5a2 2 0 0 1-2-2V8a2 2 0 0 1 2-2h6"></path><polyline points="15 3 21 3 21 9"></polyline><line x1="10" y1="14" x2="21" y2="3"></line></svg></a></span></td></tr><tr><td>SCTKEY</td><td align="center">❌</td><td align="center"> </td><td>SENDKEY for ServerChanTurbo. <span class="external-link"><a class="no-external-link" href="https://sct.ftqq.com/" target="_blank">https://sct.ftqq.com/<svg xmlns="http://www.w3.org/2000/svg" width="16px" height="16px" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="feather feather-external-link"><path d="M18 13v6a2 2 0 0 1-2 2H5a2 2 0 0 1-2-2V8a2 2 0 0 1 2-2h6"></path><polyline points="15 3 21 3 21 9"></polyline><line x1="10" y1="14" x2="21" y2="3"></line></svg></a></span></td></tr><tr><td>TG_BOT_API</td><td align="center">❌</td><td align="center">api.telegram.org</td><td>Telegram robot api address. Default: api.telegram.org</td></tr><tr><td>TG_BOT_TOKEN</td><td align="center">❌</td><td align="center"> </td><td>Telegram robot token. Generated when requesting a bot from @botfather</td></tr><tr><td>TG_USER_ID</td><td align="center">❌</td><td align="center"> </td><td>User ID of the Telegram push target.</td></tr><tr><td>WW_ID</td><td align="center">❌</td><td align="center"> </td><td>企业微信的企业ID(corpid).在'管理后台'-&gt;'我的企业'-&gt;'企业信息'里查看. <span class="external-link"><a class="no-external-link" href="https://work.weixin.qq.com/api/doc/90000/90135/90236" target="_blank">https://work.weixin.qq.com/api/doc/90000/90135/90236<svg xmlns="http://www.w3.org/2000/svg" width="16px" height="16px" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="feather feather-external-link"><path d="M18 13v6a2 2 0 0 1-2 2H5a2 2 0 0 1-2-2V8a2 2 0 0 1 2-2h6"></path><polyline points="15 3 21 3 21 9"></polyline><line x1="10" y1="14" x2="21" y2="3"></line></svg></a></span></td></tr><tr><td>WW_APP_SECRET</td><td align="center">❌</td><td align="center"> </td><td>企业微信应用的secret.在'管理后台'-&gt;'应用与小程序'-&gt;'应用'-&gt;'自建',点进某应用里查看.</td></tr><tr><td>WW_APP_USERID</td><td align="center">❌</td><td align="center">@all</td><td>企业微信应用推送对象的用户ID.在'管理后台'-&gt;' 通讯录',点进某用户的详情页里查看.默认: @all</td></tr><tr><td>WW_APP_AGENTID</td><td align="center">❌</td><td align="center"> </td><td>企业微信应用的agentid.在'管理后台'-&gt;'应用与小程序'-&gt;'应用',点进某应用里查看.</td></tr><tr><td>WW_BOT_KEY</td><td align="center">❌</td><td align="center"> </td><td>企业微信机器人WebHook地址中key后的字段. <span class="external-link"><a class="no-external-link" href="https://work.weixin.qq.com/api/doc/90000/90136/91770" target="_blank">https://work.weixin.qq.com/api/doc/90000/90136/91770<svg xmlns="http://www.w3.org/2000/svg" width="16px" height="16px" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="feather feather-external-link"><path d="M18 13v6a2 2 0 0 1-2 2H5a2 2 0 0 1-2-2V8a2 2 0 0 1 2-2h6"></path><polyline points="15 3 21 3 21 9"></polyline><line x1="10" y1="14" x2="21" y2="3"></line></svg></a></span></td></tr></tbody></table>

