---
layout: post
title: "Linux 下使用 Monit 实现服务挂掉自动拉起"
date: "2020-05-11 20:00"
category: Linux
tags: Linux Monit
author: jiangliheng
---
* content
{:toc}




# 背景

由于应用稳定性或者服务器资源限制等问题，应用就会出现自动挂掉的情况，此时就需要自动拉起应用。

生产环境，为了防止因为意外宕机造成服务长时间中断，一般都会设置服务进程监控拉起机制。

# 简介

> Monit - utility for monitoring services on a Unix system

Monit 是 Unix 系统上的服务监控工具。可以用来监控和管理进程、程序、文件、目录和设备等。

**优点**
* 安装配置简单，超轻量
* 可以监控前后台进程（Supervisor 无法监控后台启动进程）
* 除了监控进程还可以监控文件，还可以监控系统资源（CPU，内存，磁盘）使用率
* 可以设置进程依赖，控制启动顺序

**缺点**
* Monit 采用间隔轮询的方式检测，决定了它达不到 Supervisor 一样的实时感知。

# 安装

```bash
# 安装 epel 源
$ yum -y install epel-release

# 安装 monit
$ yum -y install monit

# 验证
$ monit -V
This is Monit version 5.26.0
Built with ssl, with ipv6, with compression, with pam and with large files
Copyright (C) 2001-2019 Tildeslash Ltd. All Rights Reserved.

# 启动服务
$ systemctl start monit

# 启动 monit 守护进程
$ monit
```

# 命令

> 官方手册：https://mmonit.com/monit/documentation/monit.html

命令格式: ```monit [options]+ [command]```

```bash
# 查看帮助信息
$ monit -h
```

## 命令选项

选项| 说明
:----- | :-----
```-c file``` | 启动指定配置文件
```-d n``` | 每间隔 n 秒，以守护进程的方式运行 monit
```-g name``` | 设置监控组名
```-l logfile``` | 指定日志文件
```-p pidfile``` | 指定守护模式的 PID（锁）文件
```-s statefile``` | 将状态信息写入文件
```-I``` | 不要在后台模式下运行（需要从init运行）
```--id``` | 打印 monit id 信息
```--resetid``` | 重置 monit id 信息，谨慎操作
```-B``` | 批处理命令行模式（不能输出表格或颜色）
```-t``` | 对配置文件进行语法检查
```-v``` | 详细模式，诊断输出
```-vv``` | 非常详细模式，与 -v 类似，并在错误时记录堆栈日志
```-H [filename]``` | 如果省略文件名，则输出文件或标准输入的 MD5 和 SHA1 哈希值；Monit之后将退出
```-V``` | 打印版本信息
```-h``` | 打印帮助信息

## 常用命令

命令| 说明
:----- | :-----
```monit start all``` | 启动所有服务
```monit start <name>``` | 启动指定 <name> 服务
```monit stop all``` | 停止所有服务
```monit stop <name>``` | 停止指定 <name> 服务
```monit restart all``` | 重启所有服务
```monit restart <name>``` | 重启指定 <name> 服务
```monit monitor all``` | 启用监控所有服务
```monit monitor <name>``` | 启用监控指定  <name> 服务
```monit unmonitor all``` | 禁用监控所有服务
```monit unmonitor <name>``` | 禁用监控指定 <name> 服务
```monit reload``` | 重新加载配置
```monit status``` | 打印所有服务状态
```monit status [name]``` | 打印指定 name 服务状态
```monit summary``` | 打印所有服务简要信息
```monit summary [name]``` | 打印指定 name 服务简要信息
```monit report [up|down|..]``` | 打印服务状态的个数
```monit quit``` | 退出 monit 守护进程
```monit validate``` | 检查所有服务，如果不运行则启动
```monit procmatch <pattern>``` | 测试进程匹配检查，支持正则表达式

# 配置

yum 安装后的默认配置文件如下：
全局参数配置文件 ： /etc/monitrc
服务监控配置文件目录：/etc/monit.d
日志文件： /var/log/monit.log

```bash
# 配置文件
$ grep -v "^#" /etc/monitrc
# 每 5 秒检查被监控服务的状态
set daemon  5              # check services at 30 seconds intervals
set log syslog

# 启用内置的 web 服务器
set httpd port 2812 and
    use address 10.0.0.2  # only accept connection from localhost (drop if you use M/Monit)
    # 允许 localhost 连接
    allow localhost        # allow localhost to connect to the server and
    # 解决本地命令报错问题： Error receiving data -- Connection reset by peer
    allow 10.0.0.2
    # 运行外网 IP 访问
    allow x.x.x.x
    # web登录的用户名和密码
    allow admin:monit      # require user 'admin' with password 'monit'
    #with ssl {            # enable SSL/TLS and set path to server certificate
    #    pemfile: /etc/ssl/certs/monit.pem
    #}

# 监控服务配置文件目录
include /etc/monit.d/*
```

# 监控服务

```bash
# 查看 nexus 监控文件
$ cat /etc/monit.d/nexus
check process nexus
        matching "org.sonatype.nexus.karaf.NexusMain"
        start program = "/root/nexus3/nexus-3.12.1-01/bin/nexus start"
        stop program = "/root/nexus3/nexus-3.12.1-01/bin/nexus stop"
        if failed port 18081 then restart

# 查看 nexus 监控状态
$ monit status nexus
Monit 5.26.0 uptime: 3h 48m

Process 'nexus'
  status                       OK
  monitoring status            Monitored
  monitoring mode              active
  on reboot                    start
  pid                          15191
  parent pid                   1
  uid                          0
  effective uid                0
  gid                          0
  uptime                       1m
  threads                      96
  children                     0
  cpu                          0.2%
  cpu total                    0.2%
  memory                       14.3% [1.1 GB]
  memory total                 14.3% [1.1 GB]
  security attribute           -
  disk read                    0 B/s [1.6 MB total]
  disk write                   0 B/s [232.5 MB total]
  port response time           1.756 ms to localhost:18081 type TCP/IP protocol DEFAULT
  data collected               Wed, 13 May 2020 14:36:27

# 验证 nexus 停机自动拉起
$  kill -9 15191

# 间隔时间内还未拉起
$ monit status nexus
Monit 5.26.0 uptime: 3h 48m

Process 'nexus'
  status                       Does not exist
  monitoring status            Monitored
  monitoring mode              active
  on reboot                    start
  data collected               Wed, 13 May 2020 14:36:42

# 查看自动拉起后的 nexus 监控状态
$ monit status nexus
Monit 5.26.0 uptime: 3h 48m

Process 'nexus'
  status                       OK
  monitoring status            Monitored
  monitoring mode              active
  on reboot                    start
  pid                          15830
  parent pid                   1
  uid                          0
  effective uid                0
  gid                          0
  uptime                       0m
  threads                      52
  children                     0
  cpu                          64.0%
  cpu total                    64.0%
  memory                       4.5% [349.2 MB]
  memory total                 4.5% [349.2 MB]
  security attribute           -
  disk read                    0 B/s [84 kB total]
  disk write                   0 B/s [36.9 MB total]
  port response time           -
  data collected               Wed, 13 May 2020 14:36:45

# 查看过程日志
$ tailf -20 /var/log/monit.log
......
[CST May 13 14:35:09] error    : 'nexus' process is not running
[CST May 13 14:35:09] info     : 'nexus' trying to restart
[CST May 13 14:35:09] info     : 'nexus' start: '/root/nexus3/nexus-3.12.1-01/bin/nexus start'
[CST May 13 14:35:17] info     : Reinitializing monit daemon
[CST May 13 14:35:17] info     : Reinitializing Monit -- control file '/etc/monitrc'
[CST May 13 14:35:17] info     : 'VM_0_2_centos' Monit reloaded
[CST May 13 14:36:42] error    : 'nexus' process is not running
[CST May 13 14:36:42] info     : 'nexus' trying to restart
[CST May 13 14:36:42] info     : 'nexus' start: '/root/nexus3/nexus-3.12.1-01/bin/nexus start'
[CST May 13 14:36:45] info     : 'nexus' process is running with pid 15830
```

# web 控制台

web 控制台地址：http://10.0.0.2:2812/

主页面：
![-w1395](/assets/images/monit/15893383990540.jpg)

监控运行信息：
![-w1394](/assets/images/monit/15893390832925.jpg)

系统监控信息：
![-w1395](/assets/images/monit/15893384128051.jpg)

进程监控信息：
![-w1397](/assets/images/monit/15893390410457.jpg)

> 微信公众号：daodaotest
