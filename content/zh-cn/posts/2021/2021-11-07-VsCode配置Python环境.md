---
slug: 5
title: VsCode 配置 Python 环境
date: '2021-11-07T11:28:58+08:00'
author: yexca
# layout: post
# permalink: /archives/5
views:
    - '242'
categories:
    - 技术学习
tags:
    - Python
    - 'VS Code'
---

## 正文

安装完成 VS Code 和 Python 并配置环境变量后

打开 VS Code，进入拓展搜索并下载 Python

在资源管理器新建一个 Python 源文件 (.py) 后，资源管理器会在.vscode 文件夹下生成 setting.json 文件（若没有自动生成可自己创建）

打开 setting.json 文件，并替换为以下代码

```json
{
    "python.linting.flake8Enabled": true,
    "python.linting.flake8Args": ["--max-line-length=248"],
    "python.linting.pylintEnabled": false

}
```

此时回到python文件，VS Code右下会弹出警告，点击下载

按 `CTRL+SHIFT+P` 键，输入 `Python: Select Interpreter` (即 Python：选择编译器)

然后选择您下载的编译器即可

如果 .vscode 文件夹下有 launch.json 文件，需要在该文件的 `configurations` 中加入以下代码

```json
{
            "name": "Python: 当前文件",
            "type": "python",
            "request": "launch",
            "program": "${file}",
            "console": "integratedTerminal"
        }
```

## 参考文章

[VsCode 配置 Python 环境小白教程](https://blog.csdn.net/Amoduo1/article/details/111246209)

[VSCode 配置 Python 教程](https://blog.csdn.net/Zhangguohao666/article/details/105040139)
