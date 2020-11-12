---
layout: post
title: "跨平台开源密码管理器 KeePassXC"
date: "2020-04-06 00:30"
category: Soft
tags: Linux KeePassXC
author: jiangliheng
---
* content
{:toc}


KeePassXC 是一个开源的跨平台密码管理器。基于 KeePass 二次开发。
KeePassXC 可以安全地在本地存储您的密码，配合浏览器插件```KeePassXC-Browser```可辅助登录。



# 简介

KeePassXC 是一个开源的跨平台密码管理器。基于 KeePass 二次开发。

KeePassXC 可以安全地在本地存储您的密码，配合浏览器插件```KeePassXC-Browser```可辅助登录。

## 强加密
> The complete database is always encrypted with the industry-standard AES (alias Rijndael) encryption algorithm using a 256 bit key. KeePassXC uses a database format that is compatible with KeePass Password Safe. Your wallet works offline and requires no Internet connection.

使用 AES（别名 Rijndael）加密算法（256 位密钥）对数据库进行加密。可以离线使用，不需要互联网连接。

## 跨平台
> KeePassXC is a community fork of KeePassX, the cross-platform port of KeePass for Windows. Every feature works cross-platform and was thoroughly tested on multiple systems to provide users with the same look and feel on every supported operating system. This includes the beloved Auto-Type feature.

KeePassXC 支持跨平台（Linux、macOS、Windows）运行，并且在多平台上进行了全面测试。

## 开源
> The full source code is published under the terms of the GNU General Public License.
We see open source as a vital prerequisite for any security-critical software product. For that reason, KeePassXC is and always will be free as in freedom (and in beer). Contributions by everyone are welcome!

KeePassXC 完全开源。开源才是安全的前提。

# 安装并使用（macOS）

以下操作都是在 macOS 上操作的，其他平台操作类似。

下载地址：https://keepassxc.org/download/ ，下载相应平台安装包，安装即可。

![-w800](/assets/images/soft/keepassxc/15860980727848.jpg)

创建一个数据库，设置主密钥，存储到本地目录。

![-w692](/assets/images/soft/keepassxc/15860981034771.jpg)

![-w692](/assets/images/soft/keepassxc/15860981126145.jpg)

![-w1057](/assets/images/soft/keepassxc/15860981307105.jpg)

添加群组和项目。

![-w800](/assets/images/soft/keepassxc/15860983388217.jpg)

## KeePassXC 快捷登录

在 KeePassXC 软件中，选择项目，快捷键（shift+command+u）可以快速打开URL 链接，实现快捷登录。

![-w226](/assets/images/soft/keepassxc/15860999311030.jpg)

# 设置与浏览器集成

设置浏览器集成。

![-w800](/assets/images/soft/keepassxc/15860986730585.jpg)

在 Chrome 中，搜索安装 KeePassXC-Browser 插件。安装完成后，单击右上角的 KeePassXC 图标。会看到下面的对话框。

![-w873](/assets/images/soft/keepassxc/15860990010303.jpg)

单击蓝色的 “连接” 按钮，连接到 KeePassXC，起一个唯一的名称即可。

![-w278](/assets/images/soft/keepassxc/15860990542283.jpg)

首次登录，需要访问确认，选择“记住此选项”，点击“允许”即可。

![-w1392](/assets/images/soft/keepassxc/15860993465463.jpg)

允许后，密码就会自动填充了。


登录网站时，也会提示保存和更新密码。

![-w1397](/assets/images/soft/keepassxc/15861002270215.jpg)

> 微信公众号：daodaotest
