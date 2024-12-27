---
slug: 81
title: 'Linux 学习 第七章 编写简单的 shell 脚本'
date: '2022-12-05T20:47:43+08:00'
author: yexca
# layout: post
# permalink: /archives/81
views:
    - '132'
categories:
    - Linux
    - LinuxStudy
tags:
    - 读书笔记
---

## 第七章 编写简单的 shell 脚本

如果需要反复执行某一任务，而该任务又需要输入大量的命令行，那么可以通过写入 shell 脚本以实现一条命令完成所有任务

### 7.1 理解 shell 脚本

shell 脚本是一组包含命令、函数、变量或其他可以通过 shell 使用的功能。这些项目被输入一个纯文本文件中，而该文件可以作为一条命令来运行

> 类似于 Windows 中的批处理文件 (.bat)

#### 7.1.1 执行和调试 shell 脚本

shell 脚本的主要优点是可以在任何文本编辑器中打开以查看脚本的内容，最大的缺点是大型或复杂的 shell 脚本的执行通常比编译后的程序要慢。可以通过两种基本的方法执行 shell 脚本

1. 将脚本名称作为 shell 的一个参数，例如：`bash myscript`

2. 在 shell 脚本第一行添加解释器名称 (`#!/bin/bash`)，给该文件添加执行权限后 (`chmod +x myscript`)，通过在命令行输入脚本的路径运行，例如 (`./myscript.sh`)

在执行时跟在脚本名称后面的为命令行参数

> 注释为 `#`
>
> 可以在脚本开头添加 `set-x` 以使用 `$ bash -x myscript` 显示正在执行的命令

#### 7.1.2 理解 shell 变量

shell 变量中的变量名称是大小写敏感的，注意定义时等号 (`=`) 左右无空格，例如

`NAME=value`

可以为变量分配常量，例如文本、数字及下划线

也可为变量赋值一个命令，例如：`MYDATE=$(date)` 以将 date 命令的输出分配给变量 MYDATE

这样每次使用变量 MYDATE 将运行一次 date 命令并将结果赋值给 MYDATE 。可以将命令放在引号 `'` 中以获得赋值时命令的运行结果

---

特殊的 shell 字符：美元符号 (`$`)、引号 (`'`)、星号 (`*`)、感叹号 (`!`) 等

如果想在命令行输出显示 `$HOME` 需要转义 `$`，可使用 `echo '$HOME'` 或 `echo \$HOME` ，即：

如果想要 shell 从字面上解释单个字符，使用反斜杠 `\`

如果想从字面上解释一组字符，则使用单引号 (`'`) 包围这些字符

如果想从字面上解释一部分字符，使用双引号 (`"`) 包围一组文本，其中美元符号 (`$`)、引号 (`'`) 和感叹号 (`!`) 将被解释，而其他字符 (例如星号 `*`) 则不会被解释

> 为变量赋值直接使用变量名，而引用变量，即需要获取变量值时需要在变量名前加美元符号 (`$`)
>
> 例如将某变量的值赋值给新变量：`newVar="$oldVar"`

---

1. ##### 特殊的 shell 位置参数

*位置参数* ，或 *命令行参数* ，名为 `$0`、`$1`、`$2`...`$n`

其中 `$0` 为被调用脚本的名称，而其他的则被赋予从命令行传递而来的参数值，例如：

```bash
#!/bin/bash

echo "第一个参数是 $1 ，第二个参数是 $2 "
echo "该脚本名称为 $0 "
echo "一共传入了 $# 个参数"
echo "所有的参数为：$@ "
```

执行命令：`./myscript hello bye` ，运行结果为下：

```bash
第一个参数是 hello ，第二个参数是 bye
该脚本名称为 /home/yexca/tmp/myscript
一共传入了 2 个参数
所有的参数为：hello bye
```

还有一个有意思的参数 `$?` 接受最后一条被执行的命令的退出状态，一般正常退出会返回 `0`

---

2. ##### 读取参数

通过使用 `read` 命令读取用户输入

```bash
#!/bin/bash

read -p "请输入两个名词：" var1 var2
echo "刚刚输入了 $var1 和 $var2"
```

---

3. 在 Bash 中进行参数扩展

想获取一个变量的值，需要在变量名前加美元符号 (`$`) ，例如 `$var` ，这其实是 `${var}` 的简写 

Bash 有一些规则可以以不同方式扩展参数值，以下为比较常用的，以 `${var}` 为例

| 示例            | 描述                                                      |
| --------------- | --------------------------------------------------------- |
| ${var:-value}   | 如果变量未设置或为空，在将其扩展为 value                  |
| ${var#pattern}  | 从 var 的值的 *前面* 开始砍掉与 pattern 最 *短*  的匹配项 |
| ${var##pattern} | 从 var 的值的 *前面* 开始砍掉与 pattern 最 *长* 的匹配项  |
| ${var%pattern}  | 从 var 的值的 *末尾* 开始砍掉与 pattern 最 *短* 的匹配项  |
| ${var%%pattern} | 从 var 的值的 *末尾* 开始砍掉与 pattern 最 *长* 的匹配项  |

基于这些特性，可以有一些有用的应用，例如：

```bash
myFileName=/home/yexca/myfile.txt
# file 变为 myfile.txt
file=${myFileName##*/}
# dir 变为 /home/yexca
dir=${myFileName%/*}
# name 变为 myfile
name=${file%.*}
# extension 变为 txt
extension=${file##*.}
```

#### 7.1.3 在 shell 脚本中执行算法

Bash 使用了非类型化变量，除非使用 declare 告诉 Bash，否则变量被视为字符串。在进行运算时会自动转为整数，不需要在赋值时指定类型

可以使用内置 let 命令或外部 expr 命令或 bc 命令完成整数运算

如：`let result=$num/16` ，或 `let num=$RANDOM`

同时也有自增运算符，`i++` 和 `++i`

> let 命令要求每个操作数与数学运算符之间不能存在空格
>
> expr 命令则要求每个操作数和数字运算符之间存在空格
>
> 而 bc 命令对空格没有要求，可以完成浮点运算

#### 7.1.4 在 shell 脚本中使用编程结构

1. "if...then" 语句

```bash
if [ %var -eq 1 ]; then
    echo "The var is 1"
fi
```

如果比较数字，`-eq` 比较好，但若比较字符串值，等号 (`=`) 不失为一个更好的选择

```bash
if [ $str = "hello"]; then
    echo "hello"
fi
```

此外还有不等号 `!=`

通过使用 `elif` 语句，以提供更多的选择。使用 `else` 以代表其他情况

```bash
$str="$HOME"
if [ -f "$str"]; then
    echo "$str 是一个普通文件"
elif [ -d "$str"]
    echo "$str 是一个目录"
else
    echo "???"
fi
```

以下是一些可使用的测试条件

| 运算符    | 测试的内容                                                   |
| --------- | ------------------------------------------------------------ |
| -a file   | 文件是否存在，与 -e 相同                                     |
| -b file   | 文件是否为一个专用设备                                       |
| -c file   | 文件是否是特殊字符或字符设备。用来识别串行线路和终端设备     |
| -d file   | 文件是否是一个目录                                           |
| -e file   | 文件是否存在，与 -a 相同                                     |
| -f file   | 文件是否存在，是否为普通文件 (不是目录、套接字、管道、链接或设备文件) |
| -g file   | 文件是否设置了 SGID 位                                       |
| -h file   | 文件是否设置了一个符号链接，与 -L 相同                       |
| -k file   | 文件是否设置了粘滞位                                         |
| -L file   | 文件是否设置了一个符号链接，与 -h 相同                       |
| -n string | 字符串的长度是否大于 0 字节                                  |
| -O file   | 是否拥有该文件                                               |
| -p file   | 文件是否为命名管道                                           |
| -r file   | 文件是否可读                                                 |
| -s file   | 文件是否存在，并且大于 0 字节                                |
| -S file   | 文件是否存在，并且为套接字                                   |
| -t file   | 文件是否为连接到终端的描述符                                 |
| -u file   | 文件是否设置了 SUID 位                                       |
| -w file   | 文件是否可写                                                 |
| -x file   | 文件是否可执行                                               |
| -z string | 字符串的长度是否为 0 字节                                    |

以下为两个变量之间比较

| 运算符          | 测试的内容                                        |
| --------------- | ------------------------------------------------- |
| expr1 -a expr2  | 俩表达式是否都为真                                |
| expr1 -o expr2  | 有一个为真                                        |
| file1 -nt file2 | 第一个文件是否比第二个文件新 (使用修改时间戳)     |
| file1 -ot file2 | 第一个文件是否比第二个文件旧 (使用修改时间戳)     |
| file1 -ef file2 | 两个文件是否通过一个链接相关联 (硬链接或符号链接) |
| var1 = var2     | 第一个变量是否等于第二个变量                      |
| var1 -eq var2   | 第一个变量是否等于第二个变量                      |
| var1 -ge var2   | 第一个变量是否大于等于第二个变量                  |
| var1 -gt var2   | 第一个变量是否大于第二个变量                      |
| var1 -le var2   | 第一个变量是否小于等于第二个变量                  |
| var1 -lt var2   | 第一个变量是否小于第二个变量                      |
| var1 != var2    | 第一个变量是否不等于第二个变量                    |
| var1 -ne var2   | 第一个变量是否不等于第二个变量                    |

此外还可以把测试运算符与 `&&` 和 `||` 组合成类似于 C 语言中的三元运算符

C：`a>b ? a : b`

Shell：`[$a -gt $b] && $a || $b`

也可单独使用。例如
`[$a -eq $b] && $a` 为若 a 等于 b，则返回 a 的值

`[-d "$dirName"] || mkdir "$dirName"` 为若 `$dirName` 路径不存在，则执行命令 `mkdir "$dirName"`

---

2. case 命令

与 C 语言中的 switch 语句类似，用于选择。一般形式为

```bash
case "VAR" in
    Result1)
        body
        ;;
    Result2 | Result3)
        body
        ;;
    *)
        body
        ;;
easc
```

3. for...do 循环

for 循环一般用于遍历一个列表

```bash
for VAR in LIST
do
    body
done

# 或者这样

for VAR in LIST ; do
    body
done 
```

例如：

```bash
for num in 0 1 2 3 4
do
    echo "The number is $num"
done

# 或者将命令输出作为列表

for file in '/bin/ls' ; do
    echo $file
done
```

4. while...do 和 until...do 循环

结构如下

```bash
# while...do
while condition
do
    body
done

# until...do
until condition
do
    body
done
```

#### 7.1.5 使用一些有用的文本操作程序

最常用的程序包括 grep、cut、tr、awk、sed。大部分程序都设计为使用标准输入和输出

1. 一般正则表达式分析器

也就是 `grep` ，是一种查找文件或文本模式的方法。可以当成一个有用的搜索工具

格式 `grep 要查找的内容 输入`

通过查看 `man grep` 以了解更多

2. 删除文本的行段

`cut` 命令可以从文本或文件中提取字段。例如

`grep /home /etc/passwd | cut -d':' -f6 -`

首先 `grep` 命令从 /etc/passwd 文件获取包含 `/home` 的行，然后传入 `cut` 命令，`cut` 命令将这些行以 `:` 分割，然后取第六段 (`-f6`)

3. 转换或者删除字符

`tr` 命令是一个基于字符的转换器，可用于替换一个或一组字符，或者从文本行中删除一个字符

```bash
# 转换大写为小写
FOO = "AbcDEF"
echo $FOO | tr [A-Z] [a-z]

# 将该列表中文件名中空格转换为下划线
for file in *; do
    f='echo $file | tr [:blank:] [_]'
    ["$file" = "$f"] || mv -i -- "file" "$f"
done
```

4. 流编辑器

`sed` 命令是一个简单的脚本编辑器，只能执行一些简单的编辑，比如删除文本匹配特定模式的行，使用一种模式的字符替换另一种模式的字符等

过于复杂，请通过在线文档了解

#### 7.1.6 使用简单的 shell 脚本

电话列表的例子

```bash
#!/bin/bash
# (@)/ph
# A very simple telephone list
# Type "ph new name number" to add to the list, or
# just type "ph name" to get a phone number

PHONELIST=~/.phonelist.txt

# If no command line parameters ($#), there
# is a problem, so ask what they're talking about.
if [$# -lt 1]; then
    echo "Whose phone number did you want? "
    exit 1
fi

# Did you want to add a new phone number?
if [$1 = "new"]; then
    shift
    echo $*>> $PHONELIST
    echo $* added to database
    exit 0
fi

# Nope. But does the file have anything in it yet?
# This might be out first time using it, after all.
if [! -s $PHONELIST]; then
    echo "No names in the phone list yet!"
    exit 1
else
    grep -i -q "$*" $PHONELIST    # Quietly search the file
    if [$? -ne 0]; then    # Did we find anything?
        echo "Sorry, that name was not found in the phone ist"
        exit 1
    else
        grep -i "$*" $PHONELIST
    fi
fi
exit 0
```

### 7.2 小结

通过编写 shell 脚本，可以自动完成许多最常见的系统管理任务

# 