---
layout: post
title: "基于 Njmon + InfluxDB + Grafana 实现性能指标实时可视监控"
date: "2020-03-15 22:00"
category: Njmon
tags: Linux Njmon InfluxDB Grafana
author: jiangliheng
---
* content
{:toc}

可以使用 njmon 来向 InfluxDB 存储服务器性能统计数据，再通过 Grafana 实时读取展示，来实现性能测试过程中的实时可视化监控服务器性能指标的目的。



# 引言

最近逛 nmon 官网时，发现了一个新工具 njmon，功能与 nmon 类似，但输出为 JSON 格式，可以用于服务器性能统计。

可以使用 njmon 来向 InfluxDB 存储服务器性能统计数据，再通过 Grafana 实时读取展示，来实现性能测试过程中的实时可视化监控服务器性能指标的目的。

当然，传统的 nmon、InfluxDB+Grafana+Jmeter等都可以实现。

# 验证环境

CentOS Linux release 7.6.1810 (Core)

# 整体架构

![](/assets/images/njmon-influxdb-grafana/15842347064089.jpg)
> 原图链接：http://nmon.sourceforge.net/docs/nmon_outline_800.png

# InfluxDB

InfluxDB 是一个由 InfluxData 开发的开源时序型数据。它由 Go 写成，着力于高性能地查询与存储时序型数据。InfluxDB 被广泛应用于存储系统的监控数据，IoT 行业的实时数据等场景。

InfluxDB 的语法是类 SQL 的，增删改查与 mysql 相同。InfluxDB 中的 measurement 对应的关系型数据库中的 table 。默认端口是 8086。

## 安装 & 启动

> 官方教程：https://docs.influxdata.com/influxdb/v1.7/introduction/installation/

配置 InfluxDB 的 yum 源：

```bash
$ cat <<EOF | sudo tee /etc/yum.repos.d/influxdb.repo
[influxdb]
name = InfluxDB Repository - RHEL \$releasever
baseurl = https://repos.influxdata.com/rhel/\$releasever/\$basearch/stable
enabled = 1
gpgcheck = 1
gpgkey = https://repos.influxdata.com/influxdb.key
EOF
```

yum 安装

```bash
# CentOS 7-, RHEL 7-
$ sudo yum install -y influxdb
$ sudo service influxdb start

# CentOS 7+, RHEL 7+
$ sudo yum install -y influxdb
$ sudo systemctl start influxdb
```

## 创建 njmon 库

```bash
$ influx
> create database njmon
> show databases
name: databases
name
----
_internal
njmon
> exit
```

## 启用用户认证

添加用户，设置权限。

```bash
# 查看所有用户
> show users
user admin
---- -----

# 创建 admin 用户，设置密码为 admin
> create user "admin" with password 'admin' with all privileges

# 再次查看用户信息，发现 admin 为 true
> show users
user admin
---- -----
admin true
```

InfluxDB 默认是禁用认证策略的。

```bash
# 编辑配置文件，把 [http] 下的 auth-enabled 选项设置为 true
$ vi /etc/influxdb/influxdb.conf
[http]
  ...
  auth-enabled = true
  ...

# 重启服务，配置生效
$ systemctl restart influxdb.service
```

# njmon

> njmon = nmon + JSON format + real-time push to a stats database +  instant graphing of "all the stats you can eat"  (AIX and Linux)
>
This njmon is a major overhaul of nmon for the next 10 years:
- Load more stats
- JSON format is self documenting, flexible and the performance stats format for many new tools
- Direct real-time loading of the JSON into modern open source time aware databases
- New age browser based graphing tools allow dynamic data choice and graph style per VM, per server or across the estateAll this will be covered and more including many demo's.

与 nmon 类似，但输出为 JSON 格式，可以用于服务器性能统计。

## 安装 njmon

官方下载总目录：https://sourceforge.net/projects/nmon/files/

```bash
# 下载
$ wget http://sourceforge.net/projects/nmon/files/njmon_linux_binaries_v53.zip

# 解压
$ unzip njmon_linux_binaries_v53.zip

# 选择相应版本，放到 local 的 bin 下
$ mv njmon_linux_RHEL7_AMD64_v53 /usr/local/bin/njmon

# 验证
$ njmon -?
```

**njmon 统计的指标项**

```bash
$ njmon -c 1 -s 1 | jq keys
[
  "cpu_total",
  "cpuinfo",
  "cpus",
  "disks",
  "filesystems",
  "identity",
  "lscpu",
  "networks",
  "os_release",
  "proc_meminfo",
  "proc_version",
  "proc_vmstat",
  "stat_counters",
  "timestamp",
  "uptime"
]
```

关于 jq 的功能和使用，可以参见我之前写的文章 “linux 下强大的 JSON 解析命令 jq”。

## 安装 njmon_tools

```bash
# 下载
$ wget http://sourceforge.net/projects/nmon/files/njmon_tools_v50.zip

# 解压
$ unzip njmon_tools_v50.zip
Archive:  njmon_tools_v50.zip
  inflating: line2pretty.py
  inflating: njmon2influx.py
  inflating: njmond.conf
  inflating: njmond.py
  inflating: njmonold2line.py
  inflating: pretty2line.py
```

## 采集数据到 InfluxDB

官方设置了多种采集方式，本教程基于 njmon2influx.py 采集方式。

**修改配置文件 njmond.conf**

```txt
{
    "njmon_port": 8181,
    "njmon_secret": "ignore",
    "data_inject": false,
    "data_json": true,
    "directory": "/home/njmon/data",
    "influx_host": "localhost",
    "influx_port": 8086,
    "influx_user": "admin",
    "influx_password": "admin",
    "influx_dbname": "njmon",
    "workers": 2,
    "debug": true,
    # for njmon2influx.py
    "batch": 100
}
```

**采集数据**

```bash
# 间隔 5 秒，一直采集数据
$ nohup njmon -s 5 | ./njmon2influx.py njmond.conf >/dev/null 2>&1 &

# 监控 log
$ tail -f /home/njmon/data/njmon2influx.log
```

## InfluxDB 查询数据

```bash
# 用户名密码登录
$ influx -username admin -password admin
Connected to http://localhost:8086 version 1.7.10
InfluxDB shell version: 1.7.10

# 查看库
> show databases
name: databases
name
----
_internal
njmon

# 使用 njmon
> use njmon
Using database njmon

# 查看 measurements，有数据表示已经采集到
> show measurements
name: measurements
name
----
cpu_total
cpuinfo
cpus
disks
filesystems
identity
lscpu
networks
os_release
proc_meminfo
proc_version
proc_vmstat
stat_counters
timestamp
uptime
```

# Grafana

Grafana 是一个跨平台的开源的度量分析和可视化工具，可以通过将采集的数据查询然后可视化的展示，并及时通知

## 安装
> 官方教程：https://grafana.com/grafana/download

```bash
$ wget https://dl.grafana.com/oss/release/grafana-6.6.2-1.x86_64.rpm
$ sudo yum install -y grafana-6.6.2-1.x86_64.rpm
```

## 启动

```bash
# 启动并验证
$ sudo systemctl daemon-reload
$ sudo systemctl start grafana-server
$ sudo systemctl status grafana-server

# 配置自启动
$ sudo systemctl enable grafana-server.service
```

默认密码：admin / admin，登录地址：http://xx.xx.xx.xx:3000。

## 安装插件

```
# 安装插件 grafana-clock-panel
# 插件链接：https://grafana.com/grafana/plugins/grafana-clock-panel
$ grafana-cli plugins install grafana-clock-panel

# 安装插件 grafana-piechart-panel
# 插件链接：https://grafana.com/grafana/plugins/grafana-piechart-panel
$ grafana-cli plugins install grafana-piechart-panel

# 重启生效
$ service grafana-server restart
```

## 配置
**添加数据源**

![-w1390](/assets/images/njmon-influxdb-grafana/15842445509775.jpg)

**选择 InfluxDB，并配置**

![-w762](/assets/images/njmon-influxdb-grafana/15842451937158.jpg)

**导入仪表盘模板**

njmon 模板链接：https://grafana.com/grafana/dashboards?search=njmon

选择 “njmon Single Host njmon for Linux v11” 模板：

![-w1301](/assets/images/njmon-influxdb-grafana/15842529878073.jpg)


复制 ID ，在 Grafana 中导入即可：

![-w1244](/assets/images/njmon-influxdb-grafana/15842530270859.jpg)

![-w1343](/assets/images/njmon-influxdb-grafana/15842532356769.jpg)


![-w1061](/assets/images/njmon-influxdb-grafana/15842532579169.jpg)


选择 InfluxDB。

![-w983](/assets/images/njmon-influxdb-grafana/15842533343060.jpg)

导入完成。

## 查看监控数据


![-w1321](/assets/images/njmon-influxdb-grafana/15842753341949.jpg)


![-w1385](/assets/images/njmon-influxdb-grafana/15842753686111.jpg)


![-w1377](/assets/images/njmon-influxdb-grafana/15842761781488.jpg)

# 参考资料
- http://nmon.sourceforge.net/pmwiki.php?n=site.njmon
- https://docs.influxdata.com/influxdb/v1.7/introduction/installation/
- https://www.readkong.com/page/njmon-is-nmon-but-saving-to-json-format-for-modern-4222619

> 微信公众号：daodaotest
