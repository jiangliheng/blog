---
layout: post
title: "Linux 查询应用进程号、端口、文件（知道其中之一查询其他）"
date: "2020-05-12 22:00"
category: Linux
tags: Linux ps lsof netstat jps
author: jiangliheng
---
* content
{:toc}




# 背景

日常搭建环境、查问题、接手前人搭建的环境等日常操作都需要。

**常见的场景**
1. 查询应用程序的端口号（懒得查看配置文件），就可以通过查找进程号，再找端口号；
2. 知道应用程序的访问 url，在服务器通过端口号，反查进程号、文件等；
3. 查询某个文件是否被应用程序占用。

# 查看应用进程号

```bash
# 查看 jenkins 进程号
$ ps -ef | grep jenkins
或者
$ ps aux | grep jenkins
jenkins  23288  0.2  8.0 7958468 1294952 ?     Sl   3月27 161:08 java -jar jenkins.war --webroot=/home/jenkins/war --prefix=/jenkins

# java 应用可以通过 jps 命令查询
$ jps -mlv | grep jenkins
23288 jenkins.war --webroot=/home/jenkins/war --prefix=/jenkins
```

# 查询端口对应的进程号

```bash
$ lsof  -i:8080
COMMAND   PID    USER   FD   TYPE    DEVICE SIZE/OFF NODE NAME
java    23288 jenkins  155u  IPv4 224369961      0t0  TCP *:webcache (LISTEN)

或者

$ netstat -lnp | grep 8080
tcp        0      0 0.0.0.0:8080            0.0.0.0:*               LISTEN      23288/java
```

# 查看应用进程占用的文件信息

```bash
$ lsof -p <PID>
```

# 查看文件被那个进程占用

```bash
$ lsof jenkins.log
COMMAND   PID    USER   FD   TYPE DEVICE SIZE/OFF    NODE NAME
java    23288 jenkins    1w   REG  253,1  9797873 1455418 jenkins.log
java    23288 jenkins    2w   REG  253,1  9797873 1455418 jenkins.log
```

> 微信公众号：daodaotest
