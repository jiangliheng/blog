---
layout: post
title: "Linux/UNIX 下 “command not found” 原因分析及解决"
date: "2020-03-11 20:00"
category: Linux
tags: Linux
author: jiangliheng
---
* content
{:toc}

在使用 Linux/UNIX 时，会经常遇到 “**command not found**” 的错误，就如提示的信息，Linux /UNIX 没有找到该命令。原因无外乎你命令拼写错误或 Linux/UNIX 系统就没有安装该命令。



# 分析过程
## 确认命令没有拼写错误
Linux/UNIX 中的所有命令都是大小写敏感的。

## 搜索路径中检查
**查找命令路径**
```bash
$ which xxxx
/usr/bin/which: no xxxx in (/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/root/bin)
```

**显示当前的搜索路径**
```bash
$ echo $PATH
/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/root/bin
```
检查执行命令的目录是否存在。目录存在但不正确，则修正即可；目录不存在，则通过如下命令添加。

**把目录添加到 $PATH 下面**
```bash
$ export PATH=$PATH:/xxxx/bin
```

注意：永久生效，需要添加到全局环境变量文件（/etc/profile）或用户环境变量文件（~/.bash_profile）中。

```bash
# 添加到系统环境变量文件，并实时生效
$ echo "export PATH=$PATH:/xxxx/bin" >> /etc/profile && source /etc/profile

# 添加到用户环境变量文件，并实时生效
$ echo "export PATH=$PATH:/xxxx/bin" >> ~/.bash_profile && source ~/.bash_profile
```

**验证命令路径**
```bash
$ which cd
/usr/bin/cd
```

# 常见场景
### Centos 最小化安装，导致 ifconfig，netstat 命令找不到
```bash
# 查找 ifconfig 命令路径
$ which ifconfig
/usr/bin/which: no ifconfig in (/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/root/bin)

# yum 搜索 ifconfig 命令
$ yum search all ifconfig
============================================================================= 匹配：ifconfig ==============================================================================
python36-ifcfg.noarch : Python cross-platform network interface discovery (ifconfig/ipconfig/ip)
moreutils.x86_64 : Additional unix utilities
net-tools.x86_64 : Basic networking tools
python2-psutil.x86_64 : A process and system utilities module for Python
python34-psutil.x86_64 : A process and system utilities module for Python
python36-psutil.x86_64 : A process and system utilities module for Python

# yum 安装 net-tools.x86_64 包
$ yum install -y net-tools

# 验证命令路径
$ which ifconfig
/usr/sbin/ifconfig
```

> 微信公众号：daodaotest
