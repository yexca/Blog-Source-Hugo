---
slug: 159
# layout: post
title: 'Linux 定时任务 crontab'
author: yexca
date: 2024-02-26T21:34:15+08:00
# permalink: /archives/159
categories:
    - Linux
    - LinuxTools
--- 

~~这篇文章还是有点久远的，书写习惯和现在不同，甚至看着有点不习惯~~

通过 crontab 命令，我们可以在固定的间隔时间执行指定的系统指令或 shell script 脚本。时间间隔的单位可以是分钟、小时、日、月、周及以上的任意组合

## 命令格式

```bash
crontab [-u user] file crontab [-u user] [ -e | -l | -r ]
```

## 命令参数

- -u user：用来设定某个用户的 crontab 服务
- file：file 是命令文件的名字,表示将 file 做为 crontab 的任务列表文件并载入 crontab。如果在命令行中没有指定这个文件，crontab 命令将接受标准输入（键盘）上键入的命令，并将它们载入 crontab
- -e：编辑某个用户的 crontab 文件内容。如果不指定用户，则表示编辑当前用户的 crontab 文件
- -l：显示某个用户的 crontab 文件内容，如果不指定用户，则表示显示当前用户的 crontab 文件内容
- -r：从 /var/spool/cron 目录中删除某个用户的 crontab 文件，如果不指定用户，则默认删除当前用户的 crontab 文件
- -i：在删除用户的 crontab 文件时给确认提示

## 文件格式

执行 `crontab -e` 命令后，会打开当前用户的 *crontab* 文件，在这个文件中，以 `#` 开头的语句是注释语句

在 *crontab* 文件中，通过 `m h dom mon dow command` 这六个字段来设置定时任务，每一行对应一个定时任务。这六个字段的含义说明如下：

- m：对应分钟 (minute)
  指定要在一小时之中的第几分钟执行该任务。取值范围是 0-59.

- h：对应小时 (hour)
  指定要在一天之中的第几个小时执行该任务。取值范围是 0-23.

- dom：对应日期 (day of month)
  指定要在一月之中的第几天执行该任务。取值范围是 0-31.

- mon：对应月份 (month)
  指定要在一年之中的第几月执行该任务。取值范围是 1-12。
  也可以通过月份英文名称的前三个字母来指定，不区分大小写。例 如，一月的英文单词是 january，那么这里可以用 jan 来指定一月。

- dow：对应星期几 (day of week)
  指定要在一周之中的星期几执行该任务。取值范围是 0-7，0 和 7 都对应星期天。
  也可以通过星期英文名称的前三个字母来指定，不区分大小写。例如，星期一的英文单词是 monday，那么这里可以用 mon 来指定星期一。

- command：对应具体的操作
  
  提供具体的命令来指定进行什么操作，可以提供脚本文件的路径来执行该脚本文件。
  
  这六个字段要求用空格隔开。且每个字段都必须提供值，不能省略某个字段的值。从第五个字段之后的所有内容都属于第六个字段，也就是要执行的操作

***

前五个字段可以使用下面的特殊字符来指定一些特殊的时间：

- 星号（**\***）：代表**所有**可能的值，例如month字段如果是星号，则表示在满足其它字段的制约条件后每月都执行该命令操作
- 逗号（**,**）：可以用逗号隔开的值指定一个列表**范围**，例如，"1,2,5,7,8,9"
- 中杠（**-**）：可以用整数之间的中杠表示一个**整数范围**，例如 "2-6" 表示 "2,3,4,5,6"
- 正斜线（**/**）：可以用正斜线指定时间的**间隔频率**，例如 "0-23/2" 表示每两小时执行一次。同时正斜线可以和星号一起使用，例如 */10，如果用在 minute 字段，表示每十分钟执行一次

在 *command* 字段中，可以使用换行符、或者 % 字符来分隔命令内容

在第一个 % 之前的内容会传递给 shell 来执行，这个 % 自身会被替换成换行符，在 % 之后、直到行末的内容都作为标准输入传递

如果需要提供 % 字符自身，需要用 `\%` 进行转义

## 常用方法

向 cron 进程提交一个 crontab 文件之前，首先要设置环境变量 EDITOR。cron 进程根据它来确定使用哪个编辑器编辑 crontab 文件

由于默认使用 nano 编辑器不是特别好用，可以改为 vi，通过编辑 $HOME 目录下的 . profile 文件，在其中加入这样一行:

```bash
EDITOR=vi; export EDITOR
```

然后保存并退出。不妨创建一个名为 <user> cron 的文件，其中 <user> 是用户名，例如， yexcacron。在该文件中加入要定时执行的内容，例如

```bash
# (put your own initials here)echo the date to the console every
# 15minutes between 6pm and 6am
0,15,30,45 18-06 * * * /bin/echo 'date' > /dev/console
```

在上面的例子中，系统将每隔 15 分钟向控制台输出一次当前时间。如果系统崩溃或挂起，从最后所显示的时间就可以一眼看出系统是什么时间停止工作的。在有些系统中，用 tty1 来表示控制台，可以根据实际情况对上面的例子进行相应的修改。为了提交你刚刚创建的 crontab 文件，可以把这个新创建的文件作为 cron 命令的参数:

```bash
crontab yexcacron
```

现在该文件已经提交给 cron 进程，它将每隔 15 分钟运行一次。同时，新创建文件的一个副本已经被放在 /var/spool/cron 目录中，文件名就是用户名 (即 yexca)

## 执行脚本

如果执行脚本需要使用变量

```bash
30 6 * * * . /etc/profile;/bin/sh /root/zfile/bin/restart.sh
```

以上为每日 6.30 执行 zfile 的 restart.sh

## 参考文章

[crontab 定时任务 — Linux Tools Quick Tutorial](https://linuxtools-rst.readthedocs.io/zh_CN/latest/tool/crontab.html)

[Linux技巧：介绍设置定时周期执行任务的方法 - SegmentFault 思否](https://segmentfault.com/a/1190000023186565)

[Linux命令之Crontab——定时任务 - SegmentFault 思否](https://segmentfault.com/a/1190000021815907)

[nano编辑器使用教程 - VPS侦探](https://www.vpser.net/manage/nano.html)

[Linux vi/vim - 菜鸟教程](https://www.runoob.com/linux/linux-vim.html)
