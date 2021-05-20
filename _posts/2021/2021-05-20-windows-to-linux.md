---
layout: post
title: "Unix\Linux 执行 shell 报错：“$'\r': 未找到命令” 的解决办法"
date: "2021-05-19 19:40"
category: Linux
tags: Linux dos2unix
author: jiangliheng
---
* content
{:toc}



# 原因

大多数原因是因为 shell 脚本是在 Windows 编写导致的换行问题，具体原因是 Windows 的换行符号为 ```CRLF```（```\r\n```），而 Unix\Linux 为 ```LF```（```\n```）。


**名称解释**

缩写|全称|ASCII转义|说明
:---|:---|:----|:----
CR|Carriage Return|\r|回车
LF|Linefeed|\n|换行，Unix\Linux 的换行符
CRLF|Carriage Return & Linefeed|\r\n|回车并换行，Windows 的换行符

# 方法一（推荐）：vim 转换为 Unix 换行

```
# 测试脚本
$ cat windows.sh
#!/usr/bin/env bash
date

# 重现报错
$ sh windows.sh
windows.sh:行2: $'date\r': 未找到命令

# 查看文件格式信息
$ file windows.sh
windows.sh: a /usr/bin/env bash\015 script, ASCII text executable, with CRLF line terminators

# 转换为 Unix 换行
$ vim windows.sh
:set ff=unix
:wq

# 再次查看文件格式信息
$ file windows.sh
windows.sh: a /usr/bin/env bash script, ASCII text executable
```

# 方法二：dos2unix

```
# 安装 dos2unix
$ yum install dos2unix

# 转换为 unix 格式
$ dos2unix windows.sh
dos2unix: converting file windows.sh to Unix format ...

# 转换为 dos 格式
$ unix2dos linux.sh
unix2dos: converting file linux.sh to DOS format ...
```

# 方法三：删除掉回车（\r）符号

```
# tr 删除 \r 回车符号，^M 终端输入为Ctrl+V和Ctrl+M
$ cat windows.sh | tr -d "^M" > windows2unix.sh

# sed 删除 \r 回车符号，^M 终端输入为Ctrl+V和Ctrl+M
$ sed -i "s/^M//g" windows.sh
```

# 方法四：文本编辑器工具转换换行符合（如：atom、notepad++ 等）

下图为 atom 编辑器的修改换行方式：
![-w761](/assets/images/linux/16214767016102.jpg)


> 微信公众号：daodaotest
