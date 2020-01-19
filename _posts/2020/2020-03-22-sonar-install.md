---
layout: post
title: "代码质量管理 SonarQube 系列之 安装"
date: "2020-03-22 22:00"
category: SonarQube
tags: SonarQube PostgreSQL
author: jiangliheng
---
* content
{:toc}

SonarQube 是一个开源的代码质量管理系统。



# 简介
SonarQube 是一个开源的代码质量管理系统。

**功能介绍：**
- 15种语言的静态代码分析
*Java、JavaScript、C#、TypeScript、Kotlin、Ruby、Go、Scala、Flex、Python、PHP、HTML、CSS、XML和VB.NET*
- 检测代码 bugs 和 漏洞
- 检查安全热点
- 跟踪代码坏味道，并修复技术债务
- 代码质量度量及历史变更记录
- CI/CD 集成
- 可扩展，社区有超过 60 多个插件

# 支持平台
## Java

SonarQube 仅支持 JVM 11，SonarQube scanners 支持 JVM 8 或 11。

Java | Server | Scanners
:---| :---- | :---
Oracle JRE | 11	 | 11
 | 不支持 8 | 8
OpenJDK|	 11	| 11
 | 不支持 8	 | 8

## Database

**注意：SonarQube 7.9+ 已经不再支持 MySQL。**

Database | version
:--- | :---
PostgreSQL | 12
 | 11
 | 10
 | 9.3-9.6
 | 字符集必须设置为 UTF-8
Microsoft SQL Server | 2017 (MSSQL Server 14.0)
 | 2016 (MSSQL Server 13.0)
 | 2014 (MSSQL Server 12.0)
Oracle | 19C
 | 18C
 | 12C
 | 11G
 | XE Editions
 | 字符集必须设置为 UTF-8 系列
 | 不支持驱动包 ojdbc14.jar
 | 建议使用最新的 Oracle JDBC 驱动程序
 | 仅支持 thin 模式，不支持 OCI

## Web Browser

Browser	| Version
:--- | :---
Microsoft Internet Explorer	| IE 11
Microsoft Edge	 | Latest
Mozilla Firefox	| Latest
Google Chrome	 |Latest
Opera	 | Not tested
Safari | Latest

# 验证环境

 操作系统：macOS Catalina 版本 10.15.2

 SonarQube：8.2.0

 Oracle JDK：11

 postgreSQL：12.2

# 操作系统参数设置

SonarQube 使用 Elasticsearch 做全文搜索，所以需要设置如下：

```bash
# 实时设置
$ sysctl vm.max_map_count
$ sysctl fs.file-max
$ ulimit -n
$ ulimit -u

# 永久生效
$ echo "sonarqube   -   nofile   65536
sonarqube   -   nproc    4096" > /etc/security/limits.d/99-sonarqube.conf
```

# zip 方式安装
## 安装 postgreSQL
### macOS

```bash
# 下载
# 官方下载地址：https://www.enterprisedb.com/downloads/postgres-postgresql-downloads
$ wget https://get.enterprisedb.com/postgresql/postgresql-12.2-1-osx.dmg

# 双击安装即可

# 验证
$ ps -ef | grep postgres

# 启动
$ sudo -i -u postgres
$ ./bin/pg_ctl -D /Library/PostgreSQL/12/data start

# 停止
$ ./bin/pg_ctl -D /Library/PostgreSQL/12/data stop
```

### linux（Red Hat family）

```bash
# 安装 yum 源
$ yum install https://download.postgresql.org/pub/repos/yum/reporpms/EL-7-x86_64/pgdg-redhat-repo-latest.noarch.rpm

# 安装客户端
$ yum install -y postgresql12

# 安装服务器端
$ yum install -y postgresql12-server

# 初始化数据库，并设置开机启动
$ /usr/pgsql-12/bin/postgresql-12-setup initdb
$ systemctl enable postgresql-12
$ systemctl start postgresql-12

# 验证
$ systemctl status postgresql-12
```

## 创建 sonar 数据库

如下为 macOS 操作步骤，Linux 操作步骤一样。

```bash
$ sudo -i -u postgres
$ psql
Password for user postgres:
psql (12.2)
Type "help" for help.

# 创建数据库
postgres=# CREATE DATABASE sonar;
CREATE DATABASE

# 创建 sonar 用户
postgres=# CREATE USER sonar WITH ENCRYPTED PASSWORD 'sonar';
CREATE ROLE

# 设置权限
postgres=# GRANT ALL PRIVILEGES ON DATABASE sonar TO sonar;
GRANT

# 修改 sonar 数据库所属者为 sonar
postgres=# ALTER DATABASE sonar OWNER TO sonar;
ALTER DATABASE

# 查看数据库
postgres=# \l sonar
                            List of databases
 Name  | Owner | Encoding |   Collate   |    Ctype    | Access privileges
-------+-------+----------+-------------+-------------+-------------------
 sonar | sonar | UTF8     | zh_CN.UTF-8 | zh_CN.UTF-8 | =Tc/sonar        +
       |       |          |             |             | sonar=CTc/sonar
(1 row)

# 查看用户
postgres=# \du sonar
           List of roles
 Role name | Attributes | Member of
-----------+------------+-----------
 sonar     |            | {}

# 退出
postgres=# \q
```

## 安装 SonarQube

```bash
# 下载
$ wget https://binaries.sonarsource.com/Distribution/sonarqube/sonarqube-8.2.0.32929.zip

# 解压
$ unzip sonarqube-8.2.0.32929.zip

# 修改配置文件，设置数据库
$ vi conf/sonar.properties
sonar.jdbc.username=sonar
sonar.jdbc.password=sonar
sonar.jdbc.url=jdbc:postgresql://localhost/sonar

# 启动
$ ./macosx-universal-64/sonar.sh start
```

# docker 安装

## 下载镜像

```bash
# 下载
$ docker pull postgres
$ docker pull sonarqube

# 查看镜像
$ docker images
```

## 创建网络

```bash
$ docker network create sonar-network

# 查看网络
$ docker inspect sonar-network
```

## 启动 postgres

```bash
# 启动
$ docker run --name sonar-postgres -d  \
    -e POSTGRES_USER=sonar \
    -e POSTGRES_PASSWORD=sonar \
    -p 5432:5432 \
    --net sonar-network \
    postgres

# 查看容器
$ docker ps

# 查看启动 log
$ docker logs -f postgres
```

## 启动 sonarqube

```bash
# 启动
$ docker run --name sonarqube -d \
    -p 9000:9000 \
    -e SONARQUBE_JDBC_USERNAME=sonar \
    -e SONARQUBE_JDBC_PASSWORD=sonar \
    -e SONARQUBE_JDBC_URL=jdbc:postgresql://sonar-postgres:5432/sonar \
    --net sonar-network \
    sonarqube

# 查看容器
$ docker ps

# 查看启动 log
$ docker logs -f sonarqube
```

# 访问 SonarQube
启动成功后，通过 http://localhost:9000 进行访问。
默认用户名/密码：admin / admin。

![-w1376](/assets/images/sonar/install/15849341957964.jpg)

# 下载中文插件

在线安装中文插件，重启。

![-w1378](/assets/images/sonar/install/15849343193424.jpg)


**注意：** 离线安装只需要下载 jar 放到 extensions/plugins 目录下，重启即可。

![-w1372](/assets/images/sonar/install/15849350801013.jpg)

# 其他常用插件
- Crowd
- Bitbucket Authentication for SonarQube
- Findbugs
- Checkstyle
- PMD
- MyBatis Plugin for SonarQube
- ShellCheck Analyzer
- YAML Analyzer

> 微信公众号：daodaotest
