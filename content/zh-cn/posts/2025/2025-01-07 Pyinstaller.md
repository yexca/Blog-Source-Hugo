---
slug: 213
title: 'Python pyinstaller 打包'
# draft: true
author: yexca
date: '2025-01-07T17:26:09+09:00'
categories:
    - 编程基础
    - 编程语言
tags:
    - Python
---

Python 打包是根据当前系统环境的，Windows 下是打包出 exe 可执行程序，Linux 下打包出 ELF 格式的二进制文件，不支持跨平台打包

## 安装

通过 pip 安装

```bat
pip install pyinstaller
```

## 打包为单文件

使用参数 `--onefile`

```bat
pyinstaller --onefile main.py
```

## 常用参数

对于 Windows 程序的打包，常用参数有

* `--windowed`: 不显示终端窗口 (如果是自己实现 GUI 的话)
* `--icon=icon.ico`: 为程序添加图标
* `--hidden-import`: 显示指定需要的依赖包 (避免自动分析遗漏)
* `--add-data`: 添加额外的资源文件到打包中
* `--debug`: 启用调试信息

## 打包为多文件

打包为多文件时，使用 `--onedir` 参数

```bash
pyinstaller --onedir main.dy
```

依赖文件会放在 `_internal` 文件夹下，非常不友好

仅 pyinstaller 6.1.0 以上版本可以使用该参数

```bat
pyinstaller --contents-directory . .\main.py
```

这样打包的依赖和入口就在一个目录

## 打包配置项

假设现在工程的配置放在 `project/conf/settings.json`

打包时想把这个文件也包含，首先打包为多文件

```bash
pyinstaller --name my_program --contents-directory . .\main.py
```

会在当前目录生成 `my_program.spec` 文件，修改 `a` 的 `data`，以元组的方式将想打包的文件输入，如

```groovy
datas=[('conf/settings.json', 'conf/')],
```

然后删除 `dist/` 文件夹下所有文件后运行命令 (不删除也行，后面问题同意即可)

```bash
pyinstaller my_program.spec
```

## 项目

通过这种方法，把 <https://blog.yexca.net/archives/211> 的软件现在已经打包为可执行程序了 ~~(虽然还是没做错误处理)~~

## 参考文章

<https://www.cnblogs.com/yqbaowo/p/17863429.html>
