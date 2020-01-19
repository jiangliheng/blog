---
layout: post
title: "Docker 安装 Mysql 5.7"
date: "2020-06-21 13:00"
category: Mysql
tags: Docker Mysql
author: jiangliheng
---
* content
{:toc}



# 背景

阿里云基础版 RDS 最近因为大数据量查询经常宕机（阿里云工单回复是 OOM，让升级高可用版本~），导致日常办公软件（Crowd，Jira，Confluence等）无法使用，所以在 ECS 搭建本地 Mysql。

# 验证环境

- Centos 7.7
- Docker 1.13.1

# 拉取镜像

```bash
# 搜索 Mysql 镜像
$ docker search mysql

# 下载 Mysql 5.7 镜像
$ docker pull mysql:5.7

# 查看下载镜像
$ docker images
```

# 运行容器

```bash
# 运行 Mysql 容器
$ docker run -d 3306:3306 --name mysql -e MYSQL_ROOT_PASSWORD=daodaotest mysql:5.7

# 运行 Mysql 容器，映射目录，设置必须 Mysql 参数
$ docker run -d -p 3306:3306 --name mysql \
-v /home/mysql/mysql/conf:/etc/mysql \
-v /home/mysql/mysql/logs:/var/log/mysql \
-v /home/mysql/mysql/data:/var/lib/mysql \
-e MYSQL_ROOT_PASSWORD=daodaotest \
mysql:5.7 \
--lower_case_table_names=1 \
--max-allowed-packet=1073741824 \
--character_set_server=utf8 \
--innodb_log_file_size=256m
```

Docker 参数说明：
- ```-d```：后台运行容器，并返回容器 ID
- ```-p```：指定端口映射，格式为：主机(宿主)端口:容器端口
- ```--name```：容器名称，此处为```mysql```
- ```-v```：宿主机和容器的目录映射关系，“:” 前为宿主机目录
- ```-e```：配置信息，此处配置 Mysql 的 root 密码

Mysql 参数说明（业务需要设置）：
- ```lower_case_table_names=1```：设置表名参数名等忽略大小写，解决 Crowd 无法识别大写表名问题
- ```max-allowed-packet=1073741824```：设置最大插入和更新数据限制为 1G（1024 * 1024 * 1024 = 1073741824），单位：字节，解决 Confluence 数据迁移时大数据插入问问
- ```character_set_server=utf8```：设置 utf8字符集，解决 Confluence 添加修改中文乱码问题
- ```innodb_log_file_size=256m```：设置日志文件大小，Confluence 健康检查推荐大小

```
# 查看容器运行情况
$ docker ps

# 查看 log
$ docker logs -f mysql
```

# 访问 Mysql 服务

## 容器内访问

```bash
# 进入容器
$ docker exec -it mysql bash

# 容器内，访问 Mysql 服务
$ mysql -uroot -pdaodaotest
```

## 宿主机访问

```bash
# 仅安装 Mysql 客户端
$ rpm -ivh https://repo.mysql.com//mysql57-community-release-el7-11.noarch.rpm
$ yum install mysql-community-client.x86_64

# 宿主机访问
$ mysql -h 127.0.0.1 -uroot -pdaodaotest

# 设置 root 用户允许远程访问
> GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' IDENTIFIED BY 'daodaotest' WITH GRANT OPTION;
> FLUSH PRIVILEGES;
```

## 局域网访问

```bash
# 前提需要设置 root 远程访问权限
$ mysql -h 192.168.x.x -uroot -pdaodaotest
```

> 微信公众号：daodaotest
