---
layout: post
title: "Linux 日常操作"
date: "2020-08-27 22:00"
category: Linux
tags: Linux
author: jiangliheng
---
* content
{:toc}



# 背景

质量团队 Linux 日常操作培训，提升团队整体 Linux 水平。

注: 以下所有命令都是在 ```Centos``` 操作系统来进行演示。

# 帮助命令及工具

详见： [Linux 帮助命令及工具](/2020/08/26/linux-man/)

# 用户和用户组

命令|说明
:----|:----
```useradd```|创建一个新用户或更新默认新用户信息
```usermod```|修改一个用户账户
```userdel```|删除用户账户和相关文件
```passwd```|修改用户密码
```chage```|更改用户密码过期信息
```id```|显示真实和有效的 UID 和 GID
```su```|切换用户
```sudo```|以系统管理者身份执行指令
```w```|显示已经登录的用户以及他们在做什么
```who```|显示已经登录的用户
```who am i```|显示实际用户
```whoami```|显示有效用户
```groupadd```|创建一个新组
```groupmod```|修改组信息
```groupdel```|删除一个组
```gpasswd```|将用户加入或删除组
```newgrp```|切换用户组
```newusers```|批量更新和创建新用户
```chpasswd```|批量更新密码

## 用户和组常用命令

```bash
# 创建组
$ groupadd daodaotest2

# 修改组名
$ groupmod -n daodaotest daodaotest2

# 创建用户
$ useradd daodaotest
# 指定参数创建用户，-u uid；-g 用户组名；-G 附加组；-d 主目录；-c 用户描述；-s shell
$ useradd -u 550 -g daodaotest -G root -d /home/daodaotest -c "test user" -s /bin/bash daodaotest

# 修改用户信息
$ usermod -c "update test user" daodaotest

# 查看用户和组信息
$ id daodaotest
uid=550(daodaotest) gid=1009(daodaotest) 组=1009(daodaotest),0(root)

# 设置用户密码
$ passwd daodaotest

# 查看用户密码
$ passwd -S daodaotest
或
$ chage -l daodaotest

# 仅切换用户
$ su daodaotest

# 切换用户，并同时切换环境变量
$ su - daodaotest

# 以 root 身份安装软件
$ sudo yum install jq -y

# 查看当前有效用户
$ whoami

# 查看当前实际用户
$ who am i

# 退出
$ exit

# 删除用户，强制删除并删除与用户的相关文件（home、邮件等）
$ userdel -rf daodaotest

# 删除组
$ groupdel daodaotest
```

## 用户和组相关文件

```bash
# Linux 用户文件:
$ cat /etc/passwd
# 用户名:口令:用户标识号:组标识号:注释性描述:主目录:默认 Shell
pe:x:1001:1001::/home/pe:/bin/bash

# Linux 用户影子文件
$ cat /etc/shadow
# 用户名:加密密码:最后一次修改时间:最小修改时间间隔:密码有效期:密码需要变更前的警告天数:密码过期后的宽限时间:账号失效时间:保留字段
pe:$6$rounds=656000$qX8tIa/2E34tao1V$6xKEJc9pGhY/lqLFKPuzqt0Kd8nROPy3RdGS4a7HyCN.upgPfYZ.eF453P6.Y4u0GifVui8KdMPW4NdjhW1cn0:18239:0:99999:7:::

# Linux 用户组文件
$ cat /etc/group
# 组名:口令:组标识号:组用户列表
pe:x:1001:peftp,ruiftp,hx

# Linux 组影子文件
$ cat /etc/gshadow
# 组名:加密密码:组管理员:组附加用户列表
pe:!::peftp,ruiftp,hx
```

# 文件和目录

**文件类型**

文件类型|说明
:----|:----
b|面向块的设备文件(block-oriented device file)
c|面向字符的设备文件(charcter-oriented device file)
d|目录(directroy)
f|文件(regular file)
l|符号链接文件(symbolic link)
p|管道文件(pipe)或命名管道文件(named pipe)


**命令列表**

命令|说明
:----|:----
```ls```|列目录内容
```cd```|改变工作目录
```pwd```|返回当前的工作目录
```mkdir```|建立目录
```rmdir```|删除目录
```touch```|创建新文件、修改文件权限和时间属性
```file```|查看文件类型
```cp```|复制文件和目录
```md5sum```|获取 md5 值
```diff```|比较文件
```mv```|移动 (改名) 文件
```rm```|移除文件或者目录
```tree```|树状图列出目录
```cat```|从第一行开始查看文本内容
```tac```|与 cat 相反，从最后一行开始查看文本内容
```more```|按页显示文本内容
```less```|与 more 类似，功能跟抢到，提供翻页、跳转、查找等命令
```nl```|添加行号显示出内容
```head```|查看文件的前面 N 行，默认为 10 行
```tail```|查看文件的后面 N 行，默认为 10 行
```wc```|统计行
```cut```|截取文本
```split```|切割文件

## 文件和目录常用命令

```bash
# 长数据格式列出所有目录，并按时间排序
$ ls -lat

# 长数据格式列出所有目录，并按时间反序排序
$ ls -lart

# 长数据格式列出所有目录，并按大小反序排序
$ ls -larS

# 进入 home 目录
$ cd ~
或
$ cd

# 进入上一次工作目录
$ cd -

# 进入上层目录
$ cd ..

# 显示当前目录
$ pwd

# 查看软链接的实际路径
$ pwd -P

# 递归创建目录
$ mkdir -p daodaotest/test

# 递归删除目录
$ rmdir -p daodaotest/test

# 创建文本
$ touch 1.txt

# 查看文件类型
$ file 1.txt

# 复制文件
$ cp 1.txt 2.txt

# 查看文件 md5
$ md5sum 1.txt 2.txt

# 比较文本
$ diff 1.txt 2.txt

# 递归复制目录
$ cp -r daodaotest daodaotest2

# 修改文件名称
$ mv daodaotest2 daodaotest22

# 移动文件或目录
$ mv 2.txt daodaotest22

# 删除文件
$ rm 2.txt

# 强制递归删除
$ rm -rf daodaotest

# 显示树状目录和文件
$ tree .

# 仅显示树状目录
$ tree -d .

# 显示指定层级目录和问题
$ tree -L 2 .

# 查看文本内容
$ cat /etc/passwd
$ more /etc/passwd
$ less /etc/passwd
$ nl /etc/passwd
# 与 cat 相反，从最后一行开始查看文本内容
$ tac /etc/passwd

# 统计行数
$ ls -l | wc -l
$ cat /etc/passwd | wc -l

# 查看前几行
$ head -5 /etc/passwd

# 动态查看文本内容
$ tail -f /var/log/messages
```

## 显示部分行内容

详见：[Linux 打印文本部分行内容（前几行，指定行，中间几行，跨行，奇偶行，后几行，最后一行，匹配行）](/2020/07/10/linux-print-text-part-line-content/)

# 查找

命令|说明
:----|:----
```which```|在 ```PATH``` 路径中查找命令位置
```whereis```|程序名查找，只搜索二进制（-b），man 文件（-m），源代码文件（-s），默认返回所有
```locate```|从数据库中 ```/var/lib/mlocate/mlocate.db``` 查找文件，默认
```find```|从磁盘中查找文件
```grep```|查找文件中符合条件的字符串

## 查找常用命令

```bash
# 操作 grep 命令
$ which grep
alias grep='grep --color=auto'
	/usr/bin/grep
$ whereis grep
grep: /usr/bin/grep /usr/share/man/man1/grep.1.gz /usr/share/man/man1p/grep.1p.gz
$ locate /bin/grep
/usr/bin/grep

# 查找所有 .sh 结尾的文件
$ find . -name "*.sh"

# 查找当前路径下的所有目录
$ find . -type d

# 查找当前路径下的所有文件
$ find . -type f

# 查找当前路径下的所有文件，并列出来
$ find . -type f -exec ls -l {} \

# 查找7天前的以 .log 结果的文件，确认之后删除
$ find . -name "*.log" -mtime +7 -ok rm {} \;

# 查找关键字
$ grep root /etc/passwd

# 正则表达式查找
$ grep "/.*sh" /etc/passwd

# 递归（-r） 查找目录下的所有文件
$ grep -r LANG /etc

# 递归（-r） 查找目录下的所有文件，排除指定目录和文件
$  grep -r --exclude-dir={yum,ssh,profile.d,rc.d,ansible} --exclude=*.conf LANG /etc

# 查找关键字，并打印前（-B），后（-A），前后（-C）各多少行
$ grep root -C 1 /etc/passwd

# 不区分大小写（-i）查找关键字，并打印行号（-n）
$ grep -i ROOT -n /etc/passwd

# 反向选择，输出没有匹配的行
$ grep -v root /etc/passwd
```

# 权限

**权限码**

权限|二进制|八进制|描述
:----|:----|:----|:----
---|000|0|无权限
--x|001|1|只有执行权限
-w-|010|2|只有写入权限
-wx|011|3|写和执行权限
r--|100|4|读权限
r-x|101|5|读取和执行的权限
rw-|110|6|读取的写入的权限
rwx|111|7|所有权限


**常见权限表**

权限|说明
:----|:----
-rw------- (600) |只有拥有者有读写权限
-rw-r--r-- (644) |只有拥有者有读写权限；而属组用户和其他用户只有读权限
-rwx------ (700) |只有拥有者有读、写、执行权限
-rwxr-xr-x (755) |拥有者有读、写、执行权限；而属组用户和其他用户只有读、执行权限
-rwx--x--x (711) |拥有者有读、写、执行权限；而属组用户和其他用户只有执行权限
-rw-rw-rw- (666) |所有用户都有文件读、写权限
-rwxrwxrwx (777) |所有用户都有读、写、执行权限

**命令列表**

命令|说明
:----|:----
```chmod```|修改文件权限
```chown```|更改文件属主，也可以同时更改文件属组
```chgrp```|更改文件属组
```umask```|权限掩码

## umask

详见：[linux知识点系列之 umask](/2020/03/08/linux-umask/)

## 权限常用命令

```bash
# 修改文件权限
$ chmod 755 test.txt
$ chmod +rw test.txt

# 修改文件权限，递归（-R）修改
$ chmod -R 755 /tmp/daodaotest

# 修改文件属主用户和属组
$ chown jlh.jlh test.txt

# 修改文件属组用户
$ chgrp jlh test.txt
```

# 进程

命令|说明
:----|:----
```top```|显示当前系统正在执行的进程的相关信息
```ps```|查看进程瞬时状态
```kill```|杀死进程
```killall```|杀死指定名称的所有进程

```bash
# 显示当前进程情况，输入 h 查看帮助
$ top

# 查看 java 进程
$ ps -ef | grep java

# 强制 kill 掉进程
$ kill -9  <pid>

# kill 所有进程
$ killall -9 php-fpm
```

# Linux 查询应用进程号、端口、文件（知道其中之一查询其他）

详见: [Linux 查询应用进程号、端口、文件（知道其中之一查询其他）](/2020/05/12/linux-find-process-port-file/)

# 压缩解压

Linux 常见的压缩包格式：tar、gz、tar.gz、bz2、tar.bz2、zip

压缩率一般来说：
```
tar.bz2 > tar.gz > zip > tar
```

命令|说明
:----|:----
```tar```|tar、tar.gz 压缩解压
```zip```|zip 压缩
```unzip```|zip 解压
```gzip```|gz 压缩
```gunzip```|gz 解压

## tar

tar 是最常用的解压缩命令。

参数说明：
```
-c 建立新的压缩文件
-r 添加文件到已经压缩的文件
-u 添加改变了和现有的文件到已经存在的压缩文件
-x 从压缩的文件中提取文件
-t 显示压缩文件的内容
-z 支持gzip解压文件
-j 支持bzip2解压文件
-v 显示操作过程
-k 保留源文件不覆盖
-C 切换到指定目录
-f 指定压缩文件

--delete            删除包中文件
--strip-components  去除目录
--add-file          向包中添加文件
```


```bash
# 归档 tar 包，不压缩
$ tar -cvf test.tar test1.log test2.log
$ tar -

# 仅查看包中文件，不解压
$ tar -tvf test.tar

# 归档并压缩为 tar.gz、tar.bz2
$ tar -zcvf test.tar.gz test1.log test2.log
$ tar -jcvf test.tar.bz2 test1.log test2.log

# 解压
$ tar -xvf test.tar
$ tar -zxvf test.tar.gz
$ tar -jxvf test.tar.bz2

# 解压到指定目录
$ tar -xvf test.tar -C dir
```

## zip & unzip

参数说明：
```bash
# zip
-d 从压缩文件内删除指定的文件。
-f 此参数的效果和指定"-u"参数类似，但不仅更新既有文件，如果某些文件原本不存在于压缩文件内，使用本参数会一并将其加入压缩文件中。
-j 只保存文件名称及其内容，而不存放任何目录名称。
-r 递归处理，将指定目录下的所有文件和子目录一并处理。
-u 更换较新的文件到压缩文件内。
-v 显示指令执行过程或显示版本信息。
-y 直接保存符号连接，而非该连接所指向的文件，本参数仅在UNIX之类的系统下有效。
- <压缩效率> 压缩效率是一个介于1-9的数值。

# unzip
-l 显示压缩文件内所包含的文件
-j 只保存文件名称及其内容，而不存放任何目录名称。
-o 以压缩文件内拥有最新更改时间的文件为准，将压缩文件的更改时间设成和该
-v 显示指令执行过程或显示版本信息。
-d 指定解压目录，目录不存在会创建
```

```bash
# 打包 test 目录下的文件
$ zip -r test.zip test/

# 打包 test 目录下文件，且压缩包不带 test 目录
$ zip -rj test.zip test/

# 指定压缩比率，数值（1-9）越大，压缩率越高，耗时越长
$ zip -r8 test.zip test/*

# 解压 zip 包
$ unzip test.zip -d dir

# 查看压缩包中的文件
$ unzip -l test.zip

# 查看更多信息，例如crc校验信息等
$ unzip -v test.zip

# 解压jar包
$ unzip -o java.jar -d dir
```

## gzip & unzip

参数说明：
```
-k 保留源文件
-d 解开压缩文件
-r 递归处理，将指定目录下的所有文件及子目录一并处理
-v 显示指令执行过程
```

```bash
# 压缩
$ gzip test1.log

# 解压
$ gunzip test1.log
```

# 磁盘

命令|说明
:----|:----
```df```|报告文件系统磁盘空间的使用情况
```du```|报告磁盘空间使用情况
```fdisk```|Linux分区表操作工具软件

## 磁盘常用命令

```bash
# 查看磁盘使用情况，易读方式
$ df -h

# 查看 inode 使用情况
$ df -i

# 查看磁盘占用空间，易读方式
$ du -h

# 查看本目录磁盘占用总大小
$ du -sh

# 查看指定层级目录的大小
$ du -h --max-depth=2 .
$ du -h -d 2 .

# 查看系统硬盘
$ fdisk -l
```

# json 解析命令 jq

详见： [linux 下强大的 JSON 解析命令 jq](/2020/03/14/linux-jq/)

> 微信公众号：daodaotest
