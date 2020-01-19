---
layout: post
title: "Linux 知识点系列之  umask"
date: "2020-03-08 19:42"
category: Linux
tags: Linux umask
author: jiangliheng
---
* content
{:toc}

umask（user's mask）用来设置文件**权限掩码**。**权限掩码**是由3个八进制的数字所组成，将现有的存取权限减掉权限掩码后，即可产生建立文件时预设的权限。
UNIX最初实现时不包含umask命令。1978年左右，在UNIX第七版中引入，用于解决权限掩码问题。



# 介绍
umask（user's mask）用来设置文件**权限掩码**。**权限掩码**是由3个八进制的数字所组成，将现有的存取权限减掉权限掩码后，即可产生建立文件时预设的权限。

UNIX最初实现时不包含umask命令。1978年左右，在UNIX第七版中引入，用于解决权限掩码问题。

# Shell 命令
在 Shell 中，使用 umask命令来设置权限掩码。

```bash
umask [-S] [maskExpression]   # 中括号内的参数是可选的。
```

参数说明：
- -S 　以符号的形式来表示权限掩码。

## 显示当前掩码
```bash
$ umask           # 以数字形式显示掩码（八进制）
022
$ umask -S        # 以符号形式显示掩码
u=rwx,g=rx,o=rx
```

## 使用数字设置掩码
```bash
$ umask 007    # 设置权限掩码为 007
$ umask        # 以数字形式显示掩码（八进制）
0007           #   0 - 特殊权限 (setuid | setgid | sticky )
               #   0 - (u)用户权限掩码
               #   0 - (g)组权限掩码
               #   7 - (o)其他用户权限掩码
$ umask -S     # 以符号形式显示掩码
u=rwx,g=rwx,o=
```

### 八进制掩码表

八进制掩码 | 创建时的掩码权限 | 文件权限 | 目录权限
:--- | :--- | :--- | :---
0 | 可以设置任何权限（读、写、执行） | 6 | 7
1 | 禁止设置执行权限（读、写） | 6 | 6
2 | 禁止设置写权限（读、执行） | 4 | 5
3 | 禁止设置执行和写权限（只读） | 4 | 4
4 | 禁止设置读权限（写、执行） | 2 | 3
5 | 禁止设置读和执行权限（写） | 2 | 2
6 | 禁止设置读和写权限（执行） | 0 | 1
7 | 禁止设置所有权限（无权限） | 0 | 0

## 使用符号设置掩码
当umask使用符号设置掩码时，它将使用以下语法进行修改：
*[用户标识] 操作符 权限符号*

**用户标识表**

用户缩写符号 | 用户类 | 描述
:--- | :--- | :---
u | user | 所有者
g | group | 所属组下的所有用户
o | others | 不是所有者且不包含在所属组下的其他用户
a | all | 以上三个的所有用户，与*ugo*一样

**操作符表**

操作符 | 作用
:--- | :---
+ | 指定的权限启用，未指定的权限不变
- | 指定的权限被禁止启用，未指定的权限不变
= | 指定的权限启用，未指定的权限被禁止

**权限符号表**

权限缩写符号 | 名称 | 描述
:--- | :--- | :---
r | read | 读取文件或列出目录的内容
w | write | 写入文件或目录
x | execute | 执行文件或递归目录树
X | special execute | 参加链接权限相关知识
s | setuid/gid | 参见文件权限相关知识
t | sticky | 参见文件权限相关知识

示例：
```bash
umask u-w                 # 禁止为用户设置写权限，同时保持其余标志不变。
umask u-w,g=r,o+r         # u-w 禁止为用户设置写权限，同时保持其余标志不变；
                          # g=r 允许对组启用读权限，同时禁止对组的写入和执行权限；
                          # o+r 允许对其他人启用读权限，同时保持其他标志不变。
```

# 常用 umask
常用的umask及所对应的目录和文件权限。

umask |	文件权限 |	目录权限
:--- | :--- | :---
022 | 644	| 755
027	| 640	| 750
002	| 664	| 775
006	| 660	| 771
007	| 660	| 770

# 使用场景
## 系统 umask
在系统变量文件（/etc/profile）中设置。

```bash
# 查看默认 umask
$ grep -C 1 umask /etc/profile
if [ $UID -gt 199 ] && [ "`id -gn`" = "`id -un`" ]; then
    umask 002
else
    umask 022
fi

# 设置系统 umask，在 /etc/profile 末尾添加 umask 022即可
$ echo "umask 022" >> /etc/profile

# 实时生效环境变量
$ source /etc/profile
```

## 用户 umask
在用户变量文件（~/.bash_profile）中设置。

```bash
# 设置系统 umask，在 /etc/profile 末尾添加 umask 022即可
$ echo "umask 022" >> ~/.bash_profile

# 实时生效环境变量
$ source ~/.bash_profile
```

## vsftpd中的umask使用
vsftpd中的umask参数：
- local_umask：本地用户的 umask
- anon_umask：虚拟用户的 umask

```bash
# 查看默认 umask
$ grep -C 1 umask /etc/vsftpd/vsftpd.conf
local_umask=027

# 设置 umask 为 0022
$ sed -i 's/local_umask=027/local_umask=022/g' /etc/vsftpd/vsftpd.conf

# 重启 vsftpd 生效
$ systemctl restart vsftpd
```

## 中间件 umask
以 tomcat 为例，说明设置中间件 umask，其他中间件类似。
```bash
# 查看默认 umask
$ grep -C 1 umask bin/catalina.sh
if [ -z "$UMASK" ]; then
UMASK="0027"
fi
umask $UMASK

# 设置 umask 为 0022
$ sed -i 's/UMASK="0027"/UMASK="0022"/g' bin/catalina.sh
```

# 参考资料
- https://en.wikipedia.org/wiki/Umask
- http://www.man7.org/linux/man-pages/man2/umask.2.html

> 微信公众号：daodaotest
