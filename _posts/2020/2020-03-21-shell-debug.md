---
layout: post
title: "Shell 脚本常用调试方法"
date: "2020-03-21 19:00"
category: Shell
tags: Shell bashdb shellcheck set
author: jiangliheng
---
* content
{:toc}

曾经我刚开始学习 shell 脚本时，除了知道用 echo 输出一些信息外，并不知道其他方法，仅仅依赖 echo 来查找错误，比较难调试且过程繁琐、效率低下。本文介绍下我常用的一些 shell 脚本调试方法，希望能对 shell 的初学者有所帮助。



# sh 命令调试选项（推荐）

选项 | 说明
:--- | :---
-c | 从```-c```后的字符串中读取命令。
-n | 检查是否存在语法错误，但不会实际执行。
-x | 将执行的每一条命令和结果依次打印出来。
-v | 执行过的脚本命令打印到标准输出。

**使用方法：**

字符串读取脚本。

```bash
$ sh -c 'if [ 1 -lt 2 ];then echo "true"; else echo "false"; fi'
true
```

**注：临时测试 shell 语法或者小段脚本时使用。**

检查脚本是否存在语法错误。

```bash
$ sh -n daodaotest.sh
```

跟踪调试 shell 脚本，将执行的每一条命令结果依次打印出来。

```bash
$ sh -x daodaotest.sh
+ '[' 2 -lt 2 ']'
+ COUNT=3
+ PARAMETER=daodaotest
+ (( i = 1 ))
+ (( i <= 3 ))
+ echo '第 1 遍打印：daodaotest'
第 1 遍打印：daodaotest
+ (( i++ ))
+ (( i <= 3 ))
+ echo '第 2 遍打印：daodaotest'
第 2 遍打印：daodaotest
+ (( i++ ))
+ (( i <= 3 ))
+ echo '第 3 遍打印：daodaotest'
第 3 遍打印：daodaotest
+ (( i++ ))
+ (( i <= 3 ))
+ exit 0
```

跟踪调试 shell 脚本，将执行的每一条命令和结果依次打印出来。

```bash
$ sh -xv daodaotest.sh
#!/bin/bash
# 调试脚本示例

# 使用方法
usage() {
  echo "Usage: sh $0 COUNT PARAMETER"
  echo "\t COUNT 循环打印次数"
  echo "\t PARAMETER 打印字符串"
  echo "示例："
  echo "\t 1. 打印 daodaotest 2 次"
  echo "\t sh $0 2 daodaotest"
}

# 判断参数
if [ $# -lt 2 ];
then
  usage
  exit 1
fi
+ '[' 2 -lt 2 ']'

# 打印次数
COUNT=$1
+ COUNT=3
# 打印字符串
PARAMETER=$2
+ PARAMETER=daodaotest

# 循环打印
for (( i = 1; i <= $COUNT; i++));
do
echo "第 $i 遍打印：$PARAMETER"
done
+ (( i = 1 ))
+ (( i <= 3 ))
+ echo '第 1 遍打印：daodaotest'
第 1 遍打印：daodaotest
+ (( i++ ))
+ (( i <= 3 ))
+ echo '第 2 遍打印：daodaotest'
第 2 遍打印：daodaotest
+ (( i++ ))
+ (( i <= 3 ))
+ echo '第 3 遍打印：daodaotest'
第 3 遍打印：daodaotest
+ (( i++ ))
+ (( i <= 3 ))

exit 0
+ exit 0
```

**注：本人最常用```-x```参数，能解决 90% 的脚本调试问题。**

# set 方法
在脚本中用```set```命令。
- ```set -xv```表示启用；
- ```set +xv```表示禁用。

**使用方法：**

```bash
$ cat daodaotest.sh
set -xv
..... 省略
set +xv

$ sh daodaotest.sh 2 daodaotest
# 使用方法
usage() {
  echo "Usage: sh $0 COUNT PARAMETER"
  echo "\t COUNT 循环打印次数"
  echo "\t PARAMETER 打印字符串"
  echo "示例："
  echo "\t 1. 打印 daodaotest 2 次"
  echo "\t sh $0 2 daodaotest"
}

# 判断参数
if [ $# -lt 2 ];
then
  usage
  exit 1
fi
+ '[' 2 -lt 2 ']'

# 打印次数
COUNT=$1
+ COUNT=2
# 打印字符串
PARAMETER=$2
+ PARAMETER=daodaotest

# 循环打印
for (( i = 1; i <= $COUNT; i++));
do
echo "第 $i 遍打印：$PARAMETER"
done
+ (( i = 1 ))
+ (( i <= 2 ))
+ echo '第 1 遍打印：daodaotest'
第 1 遍打印：daodaotest
+ (( i++ ))
+ (( i <= 2 ))
+ echo '第 2 遍打印：daodaotest'
第 2 遍打印：daodaotest
+ (( i++ ))
+ (( i <= 2 ))

exit 0
+ exit 0
```

**注: 在脚本非常复杂时，```set```可以进行局部调试，在需要调试的代码块前后设置即可。**

# 工具
## shellcheck
shell 脚本静态检查工具，可以帮助你写出刚好的脚本。

> 官网：https://www.shellcheck.net/
手册：https://github.com/koalaman/shellcheck

**使用方法：**

```bash
$ shellcheck daodaotest.sh

In daodaotest.sh line 8:
  echo "\t COUNT 循环打印次数"
       ^---------------^ SC2028: echo may not expand escape sequences. Use printf.


In daodaotest.sh line 9:
  echo "\t PARAMETER 打印字符串"
       ^------------------^ SC2028: echo may not expand escape sequences. Use printf.


In daodaotest.sh line 11:
  echo "\t 1. 打印 daodaotest 2 次"
       ^-----------------------^ SC2028: echo may not expand escape sequences. Use printf.


In daodaotest.sh line 12:
  echo "\t sh $0 2 daodaotest"
       ^---------------------^ SC2028: echo may not expand escape sequences. Use printf.


In daodaotest.sh line 28:
for (( i = 1; i <= $COUNT; i++));
                   ^----^ SC2004: $/${} is unnecessary on arithmetic variables.

For more information:
  https://www.shellcheck.net/wiki/SC2028 -- echo may not expand escape sequen...
  https://www.shellcheck.net/wiki/SC2004 -- $/${} is unnecessary on arithmeti...
```

## BASH Debugger
bashdb 是一个类 GDB 的调试工具，可以运行断点设置、变量查看等常见调试操作。

> 官网：http://bashdb.sourceforge.net/

常用参数：
```
h：查看帮助。
help 命令：命令的具体信息。

n：执行下一条语句。
s n：单步执行 n 次。
b n：在行号 n 处设置断点。
c n：一直执行到行号 n 处。
l：显示上下文代码。

print：打印变量值，例如 print $COUNT。
finish：执行到程序最后或断点处。

q or exit：退出。

R：重新执行。
```

**使用方法：**

```bash
$ bashdb --debug daodaotest.sh
 bash debugger, bashdb, release 5.0-1.1.2

Copyright 2002-2004, 2006-2012, 2014, 2016-2019 Rocky Bernstein
This is free software, covered by the GNU General Public License, and you are
welcome to change it and/or distribute copies of it under certain conditions.

(/Users/jlh/tmp/daodaotest.sh:15):
15:	if [ $# -lt 2 ];

# 执行下一条语句。
bashdb<0> n
(/Users/jlh/tmp/daodaotest.sh:22):
22:	COUNT=$1

# 显示上下文代码。
bashdb<1> l
 17:      usage
 18:      exit 1
 19:    fi
 20:
 21:    # 打印次数
 22: => COUNT=$1
 23:    # 打印字符串
 24:    PARAMETER=$2
 25:
 26:    # 循环打印

# 单步执行 3 次。
bashdb<2> s 3
(/Users/jlh/tmp/daodaotest.sh:27):
27:	for (( i = 1; i <= $COUNT; i++));
((i <= 3))
bashdb<3> l
 22:    COUNT=$1
 23:    # 打印字符串
 24:    PARAMETER=$2
 25:
 26:    # 循环打印
 27: => for (( i = 1; i <= $COUNT; i++));
 28:    do
 29:    echo "第 $i 遍打印：$PARAMETER"
 30:    done
 31:

# 打印 $COUNT 参数值
bashdb<4> print $COUNT
3

# 执行到最后
bashdb<5> finish
第 1 遍打印：daodaotest
第 2 遍打印：daodaotest
第 3 遍打印：daodaotest
Debugged program terminated normally. Use q to quit or R to restart.

# 退出
bashdb<6> q
bashdb: That's all, folks...
```

> 微信公众号：daodaotest
