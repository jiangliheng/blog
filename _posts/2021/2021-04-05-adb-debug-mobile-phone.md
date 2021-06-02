---
layout: post
title: "ADB 调试手机的三种方式（USB、WLAN、WIFI）"
date: "2021-04-05 18:30"
category: APP自动化
tags: ADB
author: jiangliheng
---
* content
{:toc}



# 背景

由于 MAC 的 USB 口全被占用着，采用无线连接更方便，记录一下，以防忘记~

# USB 调试

1. 手机开启开发者选项和 USB 调试模式（通常开启方式：在```关于手机```连续点击7次```版本号```）；
2. USB 连接线和各种接口正常（个别三方数据线仅支持充电，不支持调试）；
3. 驱动状态正常（Linux 和 Mac 通常没有问题，Windows 可能需要安装）；
4. ```adb devices -l```命令查看。

# WLAN 调试（Android 10 及更低版本，需要借助 USB）

Android 10以及更低的版本，必须通过 USB 连接后，才可实现同一 WLAN 下无线调试。

1. 手机和电脑需连接在同一 WiFi 下；
2. 手机开启开发者选项和 USB 调试模式，并通过 USB 连接电脑（即```adb devices -l```可以查看到手机）；
3. 设置手机的监听```adb tcpip 5555```;
4. 拔掉 USB 线，找到手机的 IP 地址;
5. 通过 IP 连接到手机```adb connect ip```（端口默认：5555）;
6. ```adb devices -l```命令查看。

**断开与重连**
```
# 断开连接
$ adb disconnect

# 重连
$ adb connect

# 如果统一 WLAN 下，adb connect 失败后，可重置 adb，再次连接
$ adb kill-server
$ adb connect
```

# Wi-Fi 调试（Android 11 及更高版本，无需借助 USB）

从 Android 11 开始支持 ADB 以无线方式连接手机调试，可以彻底摆脱 USB 线。

1. 手机和电脑需连接在同一 WiFi 下；
2. 保证 SDK 为最新版本（```adb --version``` ≥ 30.0.0）；
3. 手机启用开发者选项和无线调试模式（会提示确认）；
4. 允许无线调试后，选择使用配对码配对。记下显示的配对码、IP 地址和端口号；
5. 运行```adb pair ip:port```，使用第 4 步中的 IP 地址和端口号；
6. 根据提示，输入第 3 步中的配对码，系统会显示一条消息，表明您的设备已成功配对；
7. （仅适用于 Linux 或 Windows）运行 ```adb connect ip:port```。

> 微信公众号：daodaotest
