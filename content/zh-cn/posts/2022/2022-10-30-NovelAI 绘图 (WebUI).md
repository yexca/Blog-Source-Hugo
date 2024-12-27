---
slug: 77
title: 'NovelAI 绘图 (WebUI)'
date: '2022-10-30T16:23:07+08:00'
author: yexca
# layout: post
# permalink: /archives/77
views:
    - '1092'
categories:
    - 日常
tags:
    - AI
    - Python
---

使用 Windows11 部署，其他系统 (如 Linux ) 请参考：[AUTOMATIC1111/stable-diffusion-webui: Stable Diffusion web UI](https://github.com/AUTOMATIC1111/stable-diffusion-webui#automatic-installation-on-linux)

## 空间占用

程序：5.3GiB (不含模型)

运行：5.5GiB 以上

请确保 C 盘空间至少 6GiB 再运行，否则电脑可能黑屏卡死

## 环境

首先是网络环境，请确保连接上互联网

1. Git

   官网：<https://git-scm.com/>

2. Python 3.10.6 以上 *(最新版本可能不稳定)*

   建议 3.10.8：<https://www.python.org/downloads/release/python-3108/>

   **勾选 `Add python.exe to PATH`**

3. 模型下载

   1. 官方模型 (偏写实)

      * 通过磁力下载（请使用正规种子客户端）

        ```bash
        magnet:?xt=urn:btih:3a4a612d75ed088ea542acac52f9f45987488d1c&dn=sd-v1-4.ckpt&tr=udp%3a%2f%2ftracker.openbittorrent.com%3a6969%2fannounce&tr=udp%3a%2f%2ftracker.opentrackr.org%3a1337
        ```

      * 其他下载方式

        访问 [AUTOMATIC1111/stable-diffusion-webui Wiki](https://github.com/AUTOMATIC1111/stable-diffusion-webui/wiki/Dependencies)

   2. Waifu 模型(二次元啦)

      访问 [hakurei/waifu-diffusion-v1-3 at main](https://huggingface.co/hakurei/waifu-diffusion-v1-3/tree/main) 选择下载

   3. 其他

      [Stable Diffusion Models (cyberes.github.io)](https://cyberes.github.io/stable-diffusion-models/)

## 克隆仓库

选择一个合适的位置，右击选择 *在终端中打开* ，然后输入以下命令

```bash
git clone https://github.com/AUTOMATIC1111/stable-diffusion-webui.git
```

更新时可以进入该目录 (stable-diffusion-webui) 后使用 `git pull` 命令

## 配置

1. 将下载的模型放入 `/models/Stable-diffusion` 目录

2. 配置 `/webui-user.bat` 文件
   在`set VENV_DIR=` 后任意输入字符串，然后保存退出

3. 运行 `/webui-user.bat` 文件

   > 下载文件过大 (6GiB 左右) ，可能会运行较长时间，期间无进度条提示（若感觉程序终止之类的，可通过网络带宽使用情况以判断是否正在下载）

---

* 如果您的显示卡是 GTX1660 或者算出来的图是黑色的

  编辑 `webui.bat` ，在开头加入以下文本

  ```bash
  set COMMANDLINE_ARGS=--precision full --no-half
  ```

---

## 其他

详细了解：[hua1995116/awesome-ai-painting: AI绘画资料合集（包含国内外可使用平台、使用教程、参数教程、部署教程、业界新闻等等） stable diffusion tutorial、disco diffusion tutorial、 AI Platform](https://github.com/hua1995116/awesome-ai-painting)

训练模型：[NovelAI hypernetwork 自训练教程 - 知乎](https://zhuanlan.zhihu.com/p/576041621)

[NovelAI软件获取 - novelai 资源站 咩小咩壁纸|NovelAI资源站](https://novelai.club/archives/js)

关键词例子

```bash
NSFW, Prhololive, uruha_rushia, 1girl, bangs, bare shoulders, red eyes, blue dress, blue green hair, blue sleeves, blush, bow, breasts, chick, collarbone, detached
collar, detached sleeves, double bun, eyebrows visible through hair, frills, hair orhament, medium hair, off-shoulder dress
```

## 参考文章

[最火的AI绘画教程！免费开源，包教会 - 零度解说](https://www.freedidi.com/6727.html)

[【心得】(NSFW) AI 色圖製作體驗 + 關鍵字 @場外休憩區 哈啦板 - 巴哈姆特](https://forum.gamer.com.tw/C.php?page=1&bsn=60076&snA=7378038)