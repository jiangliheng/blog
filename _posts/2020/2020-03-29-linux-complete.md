---
layout: post
title: "Linux 提高操作效率之 tab 命令补全"
date: "2020-03-29 12:00"
category: Linux
tags: Linux bash-completion complete
author: jiangliheng
---
* content
{:toc}

最近在使用阿里云 ECS 时，发现 Centos 无法进行 tab 补全，特别影响操作效率，本文简单记录下 Linux 下的 tab 命令补全功能，希望对 Linux 初学者有所帮助。



# 安装
Linux 最小化安装时，是没有 tab 命令补全的，需要自己手动安装。

```
# 安装
$ yum -y install bash-completion

# 重新登录生效
```

# 命令补全
默认情况下，在 Linux 中提供下列补全功能：
- 变量补全
- 用户名补全
- 可执行命令补全
- 文件名和目录补全
- 主机名补全

## 变量补全

```bash
# echo 在 $ 符号后按两次 tab 将显示所有可用的变量
$ echo $[tab] [tab]
```

## 用户名补全

```bash
# su 在 “- ” 符号后，按两次 tab 将显示所有用户名
$ su - [tab] [tab]

# 同上，按两次 tab 将显示所有用户名
$ cd ~[tab] [tab]
```

**注意**：用户名是从 /etc/passwd 文件中获取的。

## 可执行命令补全

在执行命令时，如果找到单个匹配项的可执行文件，则一个 tab 就会将可执行命令自动补全。

```bash
$ ls -lt
总用量 5736
-rwxr-xr-x 1 nginx nginx 5872560 3月  24 15:33 nginx

# ./n 之后按一次 tab 将补全可执行命令：./nginx
$ ./n[tab]
```

当找到多个匹配项时，则两个 tab 将会显示可用命令。

```bash
$ ./yum[tab] [tab]
yum                 yum-builddep        yum-config-manager  yum-debug-dump      yum-debug-restore   yumdownloader       yum-groups-manager
```

## 文件名和目录补全

与可执行命令补全类似，找到单个匹配项时，一个 tab 自动补全，两个 tab 列出所有匹配项。

```bash
$ ls -lt
总用量 80
-rw-r--r-- 1 nginx nginx 6542 3月  26 21:06 nginx.conf
drwxr-xr-x 2 root  root  4096 3月  26 20:59 site-enable
drwxr-xr-x 2 nginx nginx 4096 3月  24 15:33 ssl
-rw-r--r-- 1 nginx nginx 2656 3月  24 15:33 nginx.conf.default
-rw-r--r-- 1 nginx nginx  636 3月  24 15:33 scgi_params.default
-rw-r--r-- 1 nginx nginx  636 3月  24 15:33 scgi_params
-rw-r--r-- 1 nginx nginx  664 3月  24 15:33 uwsgi_params.default
-rw-r--r-- 1 nginx nginx  664 3月  24 15:33 uwsgi_params
-rw-r--r-- 1 nginx nginx 1077 3月  24 15:33 fastcgi.conf.default
-rw-r--r-- 1 nginx nginx 1077 3月  24 15:33 fastcgi.conf
-rw-r--r-- 1 nginx nginx 1007 3月  24 15:33 fastcgi_params.default
-rw-r--r-- 1 nginx nginx 1007 3月  24 15:33 fastcgi_params
-rw-r--r-- 1 nginx nginx 5231 3月  24 15:33 mime.types.default
-rw-r--r-- 1 nginx nginx 5231 3月  24 15:33 mime.types
-rw-r--r-- 1 nginx nginx 3610 3月  24 15:33 win-utf
-rw-r--r-- 1 nginx nginx 2837 3月  24 15:33 koi-utf
-rw-r--r-- 1 nginx nginx 2223 3月  24 15:33 koi-win

# 在cat n 之后按一次 tab 键，会自动补全 cat nginx.conf
$ cat n[tab]

# “cd ” 之后按一次 tab 键，会
$ cd [tab]
$ cd s[tab]
site-enable/ ssl/

# 当有很多文件要显示时，会显示以下警告消息
$ ls -l /etc/[tab] [tab]
Display all 194 possibilities? (y or n)
```

## 主机名补全

```bash
# ssh 在 @ 符号后，按两次 tab 键，获取要连接的主机名
$ ssh root@ [tab] [tab]

# 同上，按两次 tab 键，获取要连接的主机名
$ scp nginx.conf nginx@ [tab] [tab]
```

**注意**：主机名是从 /etc/hosts 文件中获取的。

# 查看已有的命令行补全

```bash
# 查看已有的命令行补全
$ complete | more
complete -F _minimal
complete -F _filedir_xspec oodraw
complete -F _filedir_xspec elinks
complete -F _filedir_xspec freeamp
complete -F _longopt split
complete -F _longopt sed
complete -F _longopt ld
complete -F _longopt grep
complete -j -P '"%' -S '"' jobs
complete -d pushd
complete -F _minimal sh
complete -F _filedir_xspec playmidi
complete -F _longopt mv
complete -F _known_hosts rlogin
complete -F _service service
complete -b help
complete -A stopped -P '"%' -S '"' bg
complete -F _filedir_xspec cdiff
complete -F _filedir_xspec bibtex
complete -F _filedir_xspec rgview
complete -F _filedir_xspec realplay
complete -F _filedir_xspec xine
complete -F _filedir_xspec xpdf
complete -F _longopt strip
complete -F _longopt pr
complete -F _longopt grub
complete -F _longopt gperf
complete -F _known_hosts ftp
complete -o filenames -F _yu_debug_dump yum-debug-dump.py
complete -o filenames -F _yu_builddep yum-builddep
complete -o filenames -F _yu_repoclosure repoclosure
complete -o filenames -F _yu_repo_rss repo-rss
complete -F _filedir_xspec oowriter
complete -F _filedir_xspec chromium-browser
complete -F _filedir_xspec gqmpeg
complete -F _filedir_xspec tex
complete -F _filedir_xspec zathura
complete -F _filedir_xspec lzegrep
complete -F _longopt m4
complete -F _command time
--More--

# complete 命令详情
$ man complete
```

另外，complete 可以让自己写的程序也支持自动补全功能，目前我没有此需求，需要时再研究。

> 微信公众号：daodaotest
