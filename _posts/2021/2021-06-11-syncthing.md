---
layout: post
title: "Syncthing 开源文件同步工具 "
date: "2021-06-10 20:00"
category: Soft
tags: Soft Syncthing
author: jiangliheng
---
* content
{:toc}



# 简介

> Syncthing is a continuous file synchronization program. It synchronizes files between two or more computers in real time, safely protected from prying eyes. Your data is your data alone and you deserve to choose where it is stored, whether it is shared with some third party, and how it's transmitted over the internet.

**优点**
1. 开源
2. 安全（隐私、加密传输）
3. 多平台支持（Mac、Linux、Windows、Android、第三方IOS）
4. 易于使用（GUI、WEB）

# 安装

> https://syncthing.net/downloads/

## Docker

> https://github.com/syncthing/syncthing/blob/main/README-Docker.md

```bash
# 下载镜像
$ docker pull syncthing/syncthing

# 启动服务，访问地址：http://127.0.0.1:8384
$ docker run --name sysncthing -d -p 8384:8384 -p 22000:22000/tcp -p 22000:22000/udp \
    -v /var/syncthing:/var/syncthing \
    --hostname=daodaotest \
    syncthing/syncthing:latest

# 查看启动日志  
$ docker logs -f sysncthing
```

## Linux

```bash
# 下载安装包
$ wget https://github.com/syncthing/syncthing/releases/download/v1.17.0/syncthing-linux-amd64-v1.17.0.tar.gz

#  解压
$ tar -zxvf syncthing-linux-amd64-v1.17.0.tar.gz

# 启动指定访问地址，默认访问地址：http://127.0.0.1:8384
$ cd syncthing-linux-amd64-v1.17.0
$ ./syncthing -gui-address=0.0.0.0:8384
```

## Android

```bash
# 下载安装包
$ wget https://github.com/syncthing/syncthing-android/releases/download/1.17.0/app-release.apk

# usb 调试模式连接电脑，adb 安装 apk
$ adb install app-release.apk

# 卸载
$ adb uninstall com.nutomic.syncthingandroid
```

## MacOS

> 下载地址：https://github.com/syncthing/syncthing-macos/releases/latest

```bash
# 下载，双击安装即可
$ wget https://github.com/syncthing/syncthing-macos/releases/download/v1.17.0-1/Syncthing-1.17.0-1.dmg
```

其他平台安装方式参加官网

# 使用

**使用步骤**
1. 启动 syncthing 服务
2. 获取自己的 ID（启动日志 或 GUI）
3. 添加远程设备（需要在远程设备同意）
4. 同步文件夹设置（共享设备设置、文件夹同步类型设置、扫描周期等）
5. 同步文件

## 同步 Android 手机照片到 Mac

**前提**
1. 手机和 Mac 都安装成功 Syncthing
2. 处于同一个WIFI 下（不在一个局域网也可以，传输速度较慢）


手机添加 Mac 为远程设备
![](/assets/images/soft/Syncthing/16233835069000.jpg)

![](/assets/images/soft/Syncthing/16233835875393.jpg)

手机通过扫描二维码添加设备
![-w540](/assets/images/soft/Syncthing/16233836212987.jpg)

Mac 同意远程设备添加
![](/assets/images/soft/Syncthing/16233836992582.jpg)

![](/assets/images/soft/Syncthing/16233837710960.jpg)

手机共享文件夹给 Mac（摄像机文件夹为 app 安装时默认创建）
![](/assets/images/soft/Syncthing/16233840166858.jpg)


Mac 同意共享文件夹
![](/assets/images/soft/Syncthing/16233838894057.jpg)

Mac 同意添加完成后，手机照片就会开始自动同步到 Mac
![](/assets/images/soft/Syncthing/16233839300181.jpg)

设置 Mac 文件夹类型为“仅接收”，版本控制、忽略模式、扫描间隔等也可修改
![](/assets/images/soft/Syncthing/16233841280107.jpg)

![](/assets/images/soft/Syncthing/16233844987346.jpg)


> 微信公众号：daodaotest
