---
layout: post
title: "Centos 网卡命名规范及信息查看（物理网卡，虚拟网卡）"
date: "2020-09-03 20:00"
category: Linux
tags: Linux Centos biosdevname net.ifnames
author: jiangliheng
---
* content
{:toc}



# 背景

之前写的脚本中获取 IP 地址时，未考虑虚拟网卡的情况（docker 创建的虚拟网卡），导致脚本失败，故总结下网卡相关知识。

# 一致网络设备命名规范

Centos 6及之前的版本网卡命名格式：```eth[0123…]```。

Centos 7为了方便定位和区分网络设备，采用```一致网络设备命名（CONSISTENT NETWORK DEVICE NAMING）```规范，支持 ```biosdevname``` 和 ```net.ifnames``` 两种命名规范。

## biosdevname

**biosdevname 命名规范**

设备|旧名称|新名称|示例
:----|:----|:----|:----
内嵌网络接口（LOM）|eth[0123…]|	em[1234…][a]|em1
PCI 卡网络接口|eth[0123…]|p<slot>p<ethernet port>[b]|p3p4
虚拟功能|eth[0123…]|p<slot>p<ethernet port>_<virtual interface>[c]|p3p4_1

注: 新枚举从 1 开始。


## net.ifnames

net.ifnames 命名规范为：设备类型 + 设备位置 + 数字

**设备类型**
- ```en``` 代表以太网
- ```wl``` 代表无线局域网（WLAN）
- ```ww``` 代表无线广域网（WWAN）

**设备命名**
格式|描述
:----|:----
o<index>|	板载设备索引号
s<slot>[f<function>][d<dev_id>]|热插拔插槽索引号
x<MAC>|MAC 地址
p<bus>s<slot>[f<function>][d<dev_id>]|PCI 地理位置
p<bus>s<slot>[f<function>][u<port>][..][c<config>][i<interface>]|USB 端口链

**示例**
- ```eno1``` 板载1号网卡
- ```enp0s2``` PCI扩展卡的2号端口
- ```ens33``` 热插拔插槽3号PCI-E插槽的3号端口
- ```wlp3s0``` 第3号PCI扩展卡的0号端口

## 系统默认命名规则

默认情况下，```systemd``` 会使用以下策略，采用支持的命名方案为接口命名：

- **方案 1：**如果固件或 BIOS 信息适用且可用，则使用整合了为板载设备提供索引号的固件或 BIOS 的名称（例如：```eno1```），否则请使用方案 2。
- **方案 2：**如果固件或 BIOS 信息适用且可用，则使用整合了为 PCI 快速热插拔插槽提供索引号的固件或 BIOS 名称（例如 ```ens1```），否则请使用方案 3。
- **方案 3：**如果硬件连接器物理位置信息可用，则使用整合了该信息的名称（例如：enp2s0），否则请使用方案 5。
- **方案 4：**默认不使用整合接口 MAC 地址的名称（例如：```enx78e7d1ea46da```），但用户可选择使用此方案。
- **方案 5：**传统的不可预测的内核命名方案，在其他方法均失败后使用（例如： ```eth0```）。
这个策略（如上所述）是默认策略。如果该系统已启用 biosdevname，则会使用该方案。

注：启用 biosdevname 需要添加 ```biosdevname=1``` 作为命令行参数（Dell 系统除外），此时只要安装 biosdevname，就会默认使用该方案。如果用户已添加 udev 规则，该规则会更高内核设备名称，则会优先使用这些规则。

# 查看网卡、获取 IP

```bash
# 全部网卡
$ ls /sys/class/net/
或
$ ifconfig -a
或
$ ip a

# 虚拟网卡
$  ls /sys/devices/virtual/net/

# 物理网卡
$ ls /sys/class/net/ | grep -v "$(ls /sys/devices/virtual/net/)"

# 获取本机所有 IP
$ ifconfig -a |grep inet|grep -v 127.0.0.1|grep -v inet6|awk '{print $2}'|tr -d "addr:"

# 获取物理网卡的 IP
$ ifconfig $(ls /sys/class/net/ | grep -v "$(ls /sys/devices/virtual/net/)") |grep inet|grep -v 127.0.0.1|grep -v inet6|awk '{print $2}'|tr -d "addr:"
```

# 参考资料
- https://access.redhat.com/documentation/zh-cn/red_hat_enterprise_linux/7/html/networking_guide/ch-consistent_network_device_naming

> 微信公众号：daodaotest
