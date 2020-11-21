---
layout: post
title: "Nginx 使用 logrotate 进行日志滚动"
date: "2020-11-21 19:00"
category: Nginx
tags: Nginx logrotate
author: jiangliheng
---
* content
{:toc}



# Nginx 日志滚动（官方）

向 Nginx 主进程发送 ```USR1``` 信号。

```USR1``` 信号量被 Nginx 自定义了，为重新打开日志；当 kill 命令发送 ```USR1```时，nginx 会重新打开日志文件，并重新创建进程。

```bash
# nginx 官方提供的日志滚动方式
$ mv access.log access.log.0
$ kill -USR1 `cat master.nginx.pid`
$ sleep 1
$ gzip access.log.0    # do something with access.log.0
```

# logrotate 管理 Nginx 日志

> ```logrotate``` is designed to ease administration of systems that generate large numbers of log files. It allows automatic rotation, compression, removal, and mailing of log files. Each log file may be handled daily, weekly, monthly, or when it grows too large.

```logrotate``` 是一个日志文件管理工具。用于分割日志，删除旧的日志，并创建新的日志文件，起到日志滚动的作用。

```logrotate``` 是基于 linux 的 CRON 来运行的，其脚本是 ```/etc/cron.daily/logrotate```。



## 安装 logrotate

Linux 一般会默认安装```logrotate```，它默认的配置文件在：
```bash
# 配置文件
$ cat /etc/logrotate.conf | grep -v '^#' | grep -v '^$'
weekly
rotate 4
create
dateext
include /etc/logrotate.d
/var/log/wtmp {
    monthly
    create 0664 root utmp
	minsize 1M
    rotate 1
}
/var/log/btmp {
    missingok
    monthly
    create 0600 root utmp
    rotate 1
}

# logrotate 配置文件目录，在 /etc/logrotate.conf 中会引用该目录下的所有文件
$ ls -lt /etc/logrotate.d/
```

安装 logrotate:
```bash
$ yum install logrotate
```

## 配置 logrotate

```bash
# nginx logratate 配置文件
$ vi /etc/logrotate.d/nginx
/usr/local/nginx/logs/*.log {
    # 指定转储周期为每天
    daily
    # 使用当期日期作为命名格式
    dateext
    # 如果日志丢失，不报错继续滚动下一个日志
    missingok
    # 保留 31 个备份
    rotate 31
    # 不压缩
    nocompress
    # 整个日志组运行一次的脚本
    sharedscripts
    # 转储以后需要执行的命令
    postrotate
        # 重新打开日志文件
        [ ! -f /usr/local/nginx/nginx.pid ] || kill -USR1 `cat /usr/local/nginx/nginx.pid`
    endscript
}
```

配置文件参数说明：

参数名称|说明
:----|:----
daily                         |指定转储周期为每天
weekly                        |指定转储周期为每周
monthly                       |指定转储周期为每月
dateext                       |使用当期日期作为命名格式，如：access.log-20201121
dateformat .%s                |配合dateext使用，紧跟在下一行出现，定义文件切割后的文件名，必须配合dateext使用，只支持 %Y %m %d %s 这四个参数
compress                      |通过gzip压缩转储以后的日志
nocompress                    |不压缩
copytruncate                  |用于还在打开中的日志文件，把当前日志备份并截断
nocopytruncate                |备份日志文件但是不截断
create mode owner group       |转储文件，使用指定的文件模式创建新的日志文件
nocreate                      |不建立新的日志文件
delaycompress 和 compress      |一起使用时，转储的日志文件到下一次转储时才压缩
nodelaycompress               |覆盖 delaycompress 选项，转储同时压缩。
errors address                | 专储时的错误信息发送到指定的Email 地址
ifempty                       |即使是空文件也转储，这个是 logrotate 的缺省选项。
missingok                     |如果日志丢失，不报错继续滚动下一个日志
notifempty                    |如果是空文件的话，不转储
mail address                  |把转储的日志文件发送到指定的E-mail 地址
nomail                        |转储时不发送日志文件
olddir directory              |转储后的日志文件放入指定的目录，必须和当前日志文件在同一个文件系统
noolddir                      |转储后的日志文件和当前日志文件放在同一个目录下
sharedscripts                 |运行postrotate脚本，作用是在所有日志都轮转后统一执行一次脚本。如果没有配置这个，那么每个日志轮转后都会执行一次脚本
prerotate/endscript           |在转储以前需要执行的命令可以放入这个对，这两个关键字必须单独成行
postrotate/endscript          |在转储以后需要执行的命令可以放入这个对，这两个关键字必须单独成行
rotate count                  |指定日志文件删除之前转储的次数，0 指没有备份，5 指保留5 个备份
size log-size                  |当日志文件到达指定的大小时才转储，log-size能指定bytes(缺省)及KB (sizek)或MB(sizem)，当日志文件 >= log-size 的时候就转储

## logrotate 命令参数

```
logrotate [OPTION...] <configfile>
-d, --debug ：debug模式，测试配置文件是否有错误。
-f, --force ：强制转储文件。
-m, --mail=command ：压缩日志后，发送日志到指定邮箱。
-s, --state=statefile ：使用指定的状态文件。
-v, --verbose ：显示转储过程。
```

## 手动执行 logrotate

```bash
# '-d' 调试模式（不切分日志文件），并输出详细处理过程日志
$ logrotate -d -f /etc/logrotate.d/nginx

# '-f' 强制切分日志，'-v' 输出详细信息
$ logrotate -vf /etc/logrotate.d/nginx
reading config file nginx
Allocating hash table for state file, size 15360 B

Handling 1 logs

rotating pattern: /usr/local/nginx/logs/*.log  forced from command line (100 rotations)
empty log files are rotated, old logs are removed
considering log /usr/local/nginx/logs/access.log
  log needs rotating
considering log /usr/local/nginx/logs/error.log
  log needs rotating
rotating log /usr/local/nginx/logs/access.log, log->rotateCount is 100
dateext suffix '-20201121'
glob pattern '-[0-9][0-9][0-9][0-9][0-9][0-9][0-9][0-9]'
glob finding old rotated logs failed
rotating log /usr/local/nginx/logs/error.log, log->rotateCount is 100
dateext suffix '-20201121'
glob pattern '-[0-9][0-9][0-9][0-9][0-9][0-9][0-9][0-9]'
glob finding old rotated logs failed
renaming /usr/local/nginx/logs/access.log to /usr/local/nginx/logs/access.log-20201121
renaming /usr/local/nginx/logs/error.log to /usr/local/nginx/logs/error.log-20201121
running postrotate script

# 切分后的日志文件
$ ls -lt /usr/local/nginx/logs
总用量 0
-rw-r--r-- 1 nginx root 0 11月 21 18:58 access.log
-rw-r--r-- 1 nginx root 0 11月 21 18:57 access.log-20201121
-rw-r--r-- 1 nginx root 0 11月 21 18:58 error.log
-rw-r--r-- 1 nginx root 0 11月 21 18:57 error.log-20201121
-rw-r--r-- 1 nginx root 0 11月 21 18:58 images.log
-rw-r--r-- 1 nginx root 0 11月 21 18:57 images.log-20201121
```

> 微信公众号：daodaotest
