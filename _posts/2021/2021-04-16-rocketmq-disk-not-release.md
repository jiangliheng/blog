---
layout: post
title: "RocketMQ 部署不当导致磁盘空间不释放"
date: "2021-04-16 18:40"
category: RocketMQ
tags: RocketMQ Logback
author: jiangliheng
---
* content
{:toc}



# 背景

生产环境采用 ```RocketMQ``` 三主三从集群搭建，6 个实例部署在 3 台 Linux 服务器上（节省资源），每台服务器部署一主一从，生产上运行一段时间后，发现磁盘空间报警，发现```df```与```du```显示的空间不一致（相差几十G）。

# 问题原因

```RocketMQ```在同一台服务器上，启动一主一从 2 个实例，由于 2 个主从```RocketMQ```实例采用同样的 ```Logback``` 配置文件，写入的日志名称及滚动策略是一样的。

主从 2 个实例```Logback```在 Linux 下共享日志滚动时，会导致日志文件滚动后，但是其中一个 实例进程未释放日志文件的磁盘空间。

PS：我自己写了个```Logback```的 Demo，启动多个实例会重现该问题。

```Logback```源码分析见参考文章。

**检查方法**
```
# 查看 rocketmq 未释放文件
$ lsof | grep rocketmq | grep deleted

# 查看 rocketmq 未释放文件的磁盘总大小，$7 是lsof 的 size 字段，单位 Byte
$ lsof | grep rocketmq | grep deleted |awk 'BEGIN{sum=0}{sum+=$7}END{print sum/1024/1024 "MB"}'
```

# 解决方案

**临时释放磁盘空间**
```
# 主从同时停止写操作
# -v参数：4表示为只读，6为可读写，配置禁止写入数据操作时，主备节点都需要设置禁止写入数据的操作。
$ sh /home/mq/rocketmq-all-4.6.0-bin-release/bin/mqadmin updateBrokerConfig -b mq1:10911 -k brokerPermission -v 4
$ sh /home/mq/rocketmq-all-4.6.0-bin-release/bin/mqadmin updateBrokerConfig -b mq1:10912 -k brokerPermission -v 4

# rocketmq-console 上监控实例生产和消费情况，当消费变为0时，重启服务

# 启动成功后，恢复主从写操作
$ sh /home/mq/rocketmq-all-4.6.0-bin-release/bin/mqadmin updateBrokerConfig -b mq1:10911 -k brokerPermission -v 6
$ sh /home/mq/rocketmq-all-4.6.0-bin-release/bin/mqadmin updateBrokerConfig -b mq1:10912 -k brokerPermission -v 6
```

**彻底解决**
- 新增 3 台服务器，迁移```RocketMQ```的 3 个从节点到新服务器即可；
- 不增加服务器情况下，重新复制一份```RocketMQ```安装目录，修改 ```Logback``` 中的日志路径；
- 分析```Logback```源码，修改日志滚动实现，见参考文章。

# 参考文章

> 多项目写入同一Logback日志文件导致的滚动混乱问题（修改Logback源码）：https://blog.csdn.net/Abysscarry/article/details/102847754

> 微信公众号：daodaotest
