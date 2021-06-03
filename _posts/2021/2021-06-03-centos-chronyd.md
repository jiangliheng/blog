---
layout: post
title: "Centos7 使用 chronyd 进行时钟同步"
date: "2021-06-03 19:00"
category: Linux
tags: Linux chrony timedatectl ntpdate
author: jiangliheng
---
* content
{:toc}



# 背景

最近要做阿里云迁移 IDC 机房，整理下 Linux 运维基线，简单记录，以备后用~

# 安装

```bash
# 默认已经安装
$ yum install -y chrony
```

# 配置文件

```bash
$ cat /etc/chrony.conf
# Use public servers from the pool.ntp.org project.
# Please consider joining the pool (http://www.pool.ntp.org/join.html).
# 国家服务器
server 0.cn.pool.ntp.org
server 1.cn.pool.ntp.org
server 2.cn.pool.ntp.org
server 3.cn.pool.ntp.org
# 阿里
server ntp.aliyun.com
# 腾讯
server time1.cloud.tencent.com
server time2.cloud.tencent.com
server time3.cloud.tencent.com
server time4.cloud.tencent.com
server time5.cloud.tencent.com
# 苹果
server time.asia.apple.com
# 微软
server time.windows.com
# 其他
server cn.ntp.org.cn

# Record the rate at which the system clock gains/losses time.
driftfile /var/lib/chrony/drift

# Allow the system clock to be stepped in the first three updates
# if its offset is larger than 1 second.
makestep 1.0 3

# Enable kernel synchronization of the real-time clock (RTC).
rtcsync

# Enable hardware timestamping on all interfaces that support it.
#hwtimestamp *

# Increase the minimum number of selectable sources required to adjust
# the system clock.
#minsources 2

# Allow NTP client access from local network.
#allow 192.168.0.0/16

# Serve time even if not synchronized to a time source.
#local stratum 10

# Specify file containing keys for NTP authentication.
#keyfile /etc/chrony.keys

# Specify directory for log files.
logdir /var/log/chrony

# Select which information is logged.
#log measurements statistics tracking
```

# 启动服务及时区设置

```bash
# 启动服务
$ systemctl start chronyd

# 开机启动
$ systemctl enable chronyd

# 查看当前状态
$ systemctl status chronyd

# 查看亚洲时区
$ timedatectl list-timezones | grep Asia

# 设置时区
$ timedatectl set-timezone Asia/Shanghai
```

# 验证服务

```bash
# 查看现有的时间服务器
$ chronyc sources -v

# 查看时间服务器状态
$ chronyc sourcestats -v

# 显示时钟同步相关参数
$ chronyc tracking

# 查看当前时区及时间
$ timedatectl
```

# 手动同步时间

```bash
# 使用 ntpdate 同步时间
$ ntpdate ntp.aliyun.com

# chronyd 未启动时，如下命令同步时间
$ chronyd -q 'server pool.ntp.org iburst'

# chronyd 启动时，使用如下命令同步时间
$ chronyc -a 'burst 4/4' && sleep 10 && chronyc -a makestep
```

# 手动设置时间

```bash
# date 设置时间
$ date -s '2021-06-03 19:00:00'

# 关闭 ntp 同步后，才可以使用 timedatectl 进行时间设置
$ timedatectl set-ntp false

# 设置日期和时间
$ timedatectl set-time '2021-06-03 19:00:00'

# 设置日期
$ timedatectl set-time '2021-06-03'

# 设置时间
$ timedatectl set-time '19:00:00'

# 设置完成后，再开启
$ timedatectl set-ntp true
```

# 参考
- https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/7/html/system_administrators_guide/ch-configuring_ntp_using_the_chrony_suite
- https://chrony.tuxfamily.org/comparison.html


> 微信公众号：daodaotest
