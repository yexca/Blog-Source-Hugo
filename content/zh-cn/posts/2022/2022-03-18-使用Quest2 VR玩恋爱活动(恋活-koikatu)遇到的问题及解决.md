---
slug: 30
title: '使用 Quest2 VR 玩恋爱活动 (恋活 / koikatu) 遇到的问题及解决'
date: '2022-03-18T16:54:23+08:00'
lastmod: '2025-01-03T21:01:01+09:00'
author: yexca
# layout: post
# permalink: /archives/30
views:
    - '7351'
categories:
    - 折腾经验
tags:
    - Game
    - VR
---

## 引言

在玩了 Beat Saber 和 VRchat 等 VR 游戏后突然想到 i 社有部分游戏支持 VR，本人最喜欢玩恋活，于是试着进行游玩，但却遇到相关问题，本文仅作记录。另 VR 版无剧情，且本人使用原版所以遇到问题较少。

## 前提/条件

以下图片和部分文字来自 [Oculus 官网 Support](https://support.oculus.com/articles/headsets-and-accessories/oculus-link/index-oculus-link)，部分英文自己进行了翻译，鉴于本人英文不是太好，请以官方原内容为准，以下列出主要内容，详情请参考 [Oculus Link 的兼容性要求](https://support.oculus.com/444256562873335)

### 数据线要求

Oculus Link 需使用能够支持数据和电源连接的优质 USB 数据线。为获得最佳舒适体验，您还应确保数据线长度至少为 3 米 (10 英尺)

### 电脑要求

| 配件     | 推荐配置                                    |
| -------- | ------------------------------------------- |
| CPU      | Intel i5-4590 / AMD 锐龙 5 1500X 或更高版本 |
| 显卡     | 请参阅下面的 GPU 表                         |
| 内存     | 8 GB+ 内存                                  |
| 操作系统 | Win10                                       |
| USB 接口 | 1 个 USB 接口                               |

- Oculus Link 支持的 GPU

|             NVIDIA GPU                   |   支持   |  暂时不支持     |
| ---------------------------------------- | -------- | -------------- |
| NVIDIA Titan Z                           |          | X              |
| NVIDIA Titan X                           | X        |                |
| NVIDIA GeForce GTX 970                   | X        |                |
| NVIDIA GeForce GTX 1060 **Desktop, 3GB** |          | X              |
| NVIDIA GeForce GTX 1060 **Desktop, 6GB** | X        |                |
| NVIDIA GeForce GTX 1060M                 |          | X              |
| NVIDIA GeForce GTX 1070(all)             | X        |                |
| NVIDIA GeForce GTX 1080(all)             | X        |                |
| NVIDIA GeForce GTX 1650                  |          | X              |
| NVIDIA GeForce GTX 1650 Super            | X        |                |
| NVIDIA GeForce GTX 1660                  | X        |                |
| NVIDIA GeForce GTX 1660 TI               | X        |                |
| NVIDIA GeForce RTX 20-series (all)       | X        |                |
| NVIDIA GeForce RTX 30-series (all)       | X        |                |

|    AMD GPU      |   支持   |   暂时不支持    |
| --------------- | -------- | -------------- |
| ****     | **** | **** |
| AMD 200 Series  |          | X              |
| AMD 300 Series  |          | X              |
| AMD 400 Series  | X        |                |
| AMD 500 Series  | X        |                |
| AMD 5000 Series | X        |                |
| AMD 6000 Series | X        |                |
| AMD Vega Series | X        |                |

## 一、进入游戏

进入游戏 VR 版本直接打开 KoikatuVR.exe 即可，由于使用 steam 串流，故可提前进入 steamVR

进入游戏前请确保戴上耳机，房门锁紧等以预防突发情况，如不能做到请注意行为 (doge)

### 问题一：无法进入 steamVR

#### 一、确保安装相关软件

##### 1）steamVR 安装

首先打开 steam，然后按 `Win+R`，输入 `steam://run/250820`，按下回车便会自动安装 steamVR

##### 2）Oculus 安装

访问[官网下载](https://www.oculus.com/setup/)，注意：安装完成后会下载相关文件，完成后会要求登录账户，请确保网络环境正常

如要求提供支付方式，可寻找 `跳过` 按钮

###### 如果登录一直在加载，无法成功登录

可通过修改 Hosts 解决，推荐使用火绒打开 hosts 文件修改

如果不使用火绒，打开 `C:\Windows\System32\drivers\etc`，找到 hosts 这个文件，用记事本打开

在文件末尾添加如下内容

```markdown
157.240.11.49 graph.oculus.com
157.240.11.49 www2.oculus.com
157.240.8.49 scontent.oculuscdn.com
157.240.8.49 securecdn.oculus.com
```

然后保存即可，如果不是使用火绒，请保存到一个地方然后移动回原目录并将后缀 `.txt` 去除

#### 二、确保 Link 线正常

事实上，连接 Quest2 的时候 Oculus 软件会有一步选择是否检测 Link 线，可通过此进行检测，如果当时未进行检测，可选择 `设备 - Quest2 和 Touch-USB 检测` 进行检测

#### 三、确保设置正常

##### 1）Quest2 设备设置

使用 USB 连接 PC 和 Quest2 时 Quest 会弹出 `允许访问数据`，请选择拒绝，如果选择了 `允许`，请拔下再重新连接选择 `拒绝`

##### 2）Oculus 软件设置

其实直接打开 steamVR 会有弹窗 `是否允许未知来源` 此时选择允许即可

当然，可以在软件的 `设置 - 通用 - 未知来源` 进行打开

#### 四、还是无法打开？换个姿势试试

如果以上都没问题但还是无法打开 steamVR，则可使用下述方式

##### 1）Quest2 设备

当连接 PC 后一般会有弹窗 `启用 Oculus Link`，此时选择 `启用` 即可

如果上述未选择 `启用` 或没有弹窗，可在下方任务栏的最左方即 `快速设置` 中找到 `Oculus Link`，点击即可打开

##### 2）启动 steamVR

不会有人不知道 steamVR 怎么启动吧 (doge)

如果先连接 VR 设备再打开 steam，那么 steam 的界面右上方应该有 “VR” 标识，点击即可打开

如果无此标识，可在任务栏 (或者说右下托盘) 里找到 steam 图标，鼠标右击，倒数第二个即为 steamVR  
当然，可以在 steam 库中将 `工具` 也显示，这样可以在 steam 库中看见 steamVR

## 二、开始游戏

我不知道这里应该写什么，分这个标题是因为问题二与游戏有关，那就写其他的吧 (doge)

点击 KoikatuVR.exe 会自动打开 steamVR，所以可以在 Quest2 设备打开 Oculus Link 后直接打开 KoikatuVR 即可  
**注意：游戏会在桌面有一个窗口，可通过 `Win+D` 最小化所有窗口，但当摘下头显再次戴上时好像会再次出现，请注意**

### 问题二：无法开始游戏/不知如何开始

如果您阅读其他文章或观看相关视频可能会得到仅支持部分设备 (支持啥我忘了)，如果和我一样去测试了 VR kanojo 能否正常运行，也可能会以为是靠注视，其实不然 ~~(我就是想多写点)~~

**只需要按下 “摇杆” 即可出现选择线，按下前 “扳机键” 即可选择** (更多操作请看 *三、操作说明*)

### 问题三：开始游戏后一直白屏，电脑上  一直在一半

可进[コイカツ！ DL 版](https://dlsoft.dmm.co.jp/detail/illusion_0023/)，点击下方 `体験版・無料ダウンロード` 中的 `コイカツ VR パッチ` 进行下载

文件直链：[コイカツ VR パッチ](https://sample9.dmm.co.jp/digital/pcgame/illusion_0023/vr_patch.zip)

下载解压后会有一个可执行文件，运行后会出现 `コイカツ！VR_0531 更新版` 文件夹，将里面 `setup` 文件夹内容移动到游戏根目录并覆盖即可

注意：此方法来源作者指出姿势会变成只有三个，由于我并未游玩，所以我没有姿势 (本来想着 VR 玩剧情的，但 VR 不能玩剧情)，以下为作者给出解决 (部分内容有修改)

> 姿势是在故事模式里用过什么姿势，在 vr 里才能用，所以在故事模式里战斗的时候把所有姿势都点一遍，点完就换就行，然后到晚上存档，在退出换 vr，然后姿势就齐了

原文地址：[兄弟们有没有玩了 vr 的](https://tieba.baidu.com/p/7620121778?prev=frs&source=frs#/)

## 三、操作说明

**此部分为自行游玩得出，仅作部分说明**，其他操作请自行参考其他文章

### 1）开始游戏

进入游戏后会有俩选项，分别为

スタート，即 start，即开始

エンド，即 End，即结束

按下 "摇杆" ，会出现一条线，可进行选择 (前 "扳机键")

### 2）进入正文

左右手手腕部分会出现文字，可通过左右控制器上方按键即 `Y` 和 `B` 进行切换

一共有俩个，进入战斗后有三个 (多了一个 `移動`)，分别为

| 日文       | 英文   | 中文 | 作用                                                         |
| ---------- | ------ | ---- | ------------------------------------------------------------ |
| アクション | Action | 行动 | 前 "扳机键" 可进行各种操作 侧 "扳机键" 可打开菜单 按下” 摇杆 “可进行选择 |
| システム   | system | 系统 | 前 "扳机键" 可重置位置                                       |
| 移動       | Move   | 移动 | 前 "扳机键" 按住可改变视角                                   |

### 3）注意

仅可在 "アクション" 时按下 "摇杆" 可以进行选择

## 参考文章

[Oculus Link](https://support.oculus.com/articles/headsets-and-accessories/oculus-link/index-oculus-link)

[兄弟们有没有玩了 vr 的](https://tieba.baidu.com/p/7620121778?prev=frs&source=frs#/)

[Oculus 客户端在 Win10 上面无法安装或者登陆的解决方法_国韵的博客 - CSDN 博客_oculus 无法连接服务器](https://blog.csdn.net/mo_qi_qi/article/details/83795668)

[中国移动的日文](http://www.ichacha.net/jp/中国移动.html) ~~(别问我为什么会参考这个，问就是不会日语)~~

![yexca-30](https://count.getloli.com/@yexca-30)
