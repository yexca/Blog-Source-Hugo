---
slug: 5
title: VsCode配置Python环境
date: '2021-11-07T11:28:58+08:00'
author: yexca
# layout: post
# permalink: /archives/5
views:
    - '242'
categories:
    - 软件工具
    - 编码相关
tags:
    - Python
    - 'VS Code'
---

安装完成VS Code和Python并配置环境变量后

打开VS Code，进入拓展搜索并下载Python

在资源管理器新建一个Python源文件(.py)后，资源管理器会在.vscode文件夹下生成setting.json文件（若没有自动生成可自己创建）

打开setting.json文件，并替换为以下代码

```json

{
    "python.linting.flake8Enabled": true,
    "python.linting.flake8Args": ["--max-line-length=248"],
    "python.linting.pylintEnabled": false

}

```

此时回到python文件，VS Code右下会弹出警告，点击下载

按CTRL+SHIFT+P键，输入Python: Select Interpreter (即Python：选择编译器)

然后选择您下载的编译器即可

如果.vscode文件夹下有launch.json文件，需要在该文件的”configurations”中加入以下代码

```json

{
            "name": "Python: 当前文件",
            "type": "python",
            "request": "launch",
            "program": "${file}",
            "console": "integratedTerminal"
        }

```

参考文章：[VsCode配置Python环境小白教程](https://blog.csdn.net/Amoduo1/article/details/111246209)

 [VSCode配置Python教程](https://blog.csdn.net/Zhangguohao666/article/details/105040139)