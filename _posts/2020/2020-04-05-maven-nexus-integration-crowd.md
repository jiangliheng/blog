---
layout: post
title: "Nexus3 集成 crowd 插件"
date: "2020-04-05 20:00"
category: Nexus
tags: Maven Nexus Crowd
author: jiangliheng
---
* content
{:toc}

公司使用的软件开发和协作工具为 Atlassian 系列软件，所以统一使用 crowd 来实现统一登录（SSO）。



# crowd 配置

具体操作细节见我之前写的 [Atlassian 系列软件安装（Crowd+JIRA+Confluence+Bitbucket+Bamboo）](/2020/03/28/atlassian-install)

## 添加 nexus 用户组

此处的 nx-admin 组为 nexus 默认的管理组。

![-w854](/assets/images/maven/15860637254413.jpg)

## 添加 nexus 应用

![-w1049](/assets/images/maven/15860636815453.jpg)

**注意**：“groups” 和 “Remote addresses” 的设置。

# 安装 nexus3-crowd-plugin 插件

> Available in Nexus Repository Manager Pro only

官方 Nexus Pro 直接集成了 Atlassian Crowd 支持，但社区版不支持，需要自己集成开源插件。

插件地址：https://github.com/pingunaut/nexus3-crowd-plugin

## 插件安装问题

在插件部署过程中，我碰见如下问题：

```
2020-04-05 16:33:00,792+0800 ERROR [FelixDispatchQueue]  *SYSTEM nexus3-crowd-plugin - FrameworkEvent ERROR - nexus3-crowd-plugin
org.osgi.framework.BundleException: Unable to resolve nexus3-crowd-plugin [49](R 49.0): missing requirement [nexus3-crowd-plugin [49](R 49.0)] osgi.wiring.package; (&(osgi.wiring.package=org.sonatype.nexus.security.authz)(version>=3.20.0)) Unresolved requirements: [[nexus3-crowd-plugin [49](R 49.0)] osgi.wiring.package; (&(osgi.wiring.package=org.sonatype.nexus.security.authz)(version>=3.20.0))]
	at org.apache.felix.framework.Felix.resolveBundleRevision(Felix.java:4132)
	at org.apache.felix.framework.Felix.startBundle(Felix.java:2117)
	at org.apache.felix.framework.Felix.setActiveStartLevel(Felix.java:1371)
	at org.apache.felix.framework.FrameworkStartLevelImpl.run(FrameworkStartLevelImpl.java:308)
	at java.lang.Thread.run(Thread.java:748)
```

**解决方法**
```bash
# 下载插件代码到本地
$ git clone https://github.com/pingunaut/nexus3-crowd-plugin.git

# 修改 pom.xml 内容
$ vi pom.xml
......
<parent>
    <groupId>org.sonatype.nexus.plugins</groupId>
    <artifactId>nexus-plugins</artifactId>
    <!-- 修改 parent 的版本与 nexus3 的版本一致 -->
    <version>3.12.1-01</version>
</parent>
......
<dependency>
    <groupId>org.sonatype.nexus</groupId>
    <artifactId>nexus-plugin-api</artifactId>
    <!-- 注释掉下面这行，把依赖包打入到插件 jar 中 -->
    <!--<scope>provided</scope>-->
</dependency>
......

# 重新编译打包
$ mvn clean package
```

## 前提条件

> - JDK 8 is installed
> - Sonatype Nexus OSS 3.x is installed

## 下载 nexus3-crowd-plugin 插件

```bash
$ cd /home/nexus/nexus3/nexus-3.12.1-01/system
$ wget https://github.com/pingunaut/nexus3-crowd-plugin/releases/download/nexus3-crowd-plugin-3.4.2/nexus3-crowd-plugin-3.4.2.jar
```

## 将绑定添加到 startup.properties

```bash
$ echo "reference\:file\:nexus3-crowd-plugin-3.4.2.jar = 200" >> /home/nexus/nexus3/nexus-3.12.1-01/etc/karaf/startup.properties
```

## 配置 crowd.properties

```bash
$ echo "# 配置 crowd 的地址
crowd.server.url=http://localhost:8095/crowd/
# 配置 crowd 里该 Application 的名称
application.name=nexus
# 配置 crowd 里该 Application 的密码
application.password=nexus
cache.authentication=false

# optional:
timeout.connect=15000
timeout.socket=15000
timeout.connectionrequest=15000" > /home/nexus/nexus3/nexus-3.12.1-01/etc/crowd.properties

# 重启验证
$ sh /home/nexus/nexus3/nexus-3.12.1-01/bin/nexus restart
```

# nexus 设置

## 激活插件

![-w965](/assets/images/maven/15860805439639.jpg)

## 使用 crowd 用户登录

查看 crowd 的用户信息。
![-w1381](/assets/images/maven/15860877382264.jpg)


查看 crowd 的用户详细：
![-w1045](/assets/images/maven/15860878141681.jpg)

> 微信公众号：daodaotest
