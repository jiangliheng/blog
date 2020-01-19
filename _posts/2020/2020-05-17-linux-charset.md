---
layout: post
title: "Linux 知识点系列之 字符集"
date: "2020-05-17 23:00"
category: Linux
tags: Linux locale iconv
author: jiangliheng
---
* content
{:toc}



# 操作系统字符集

```bash
# 查看操作系统支持的所有字符集
$ locale -a

# 查看操作系统支持的中文字符集
$ locale -a | grep zh

# 查看当前系统字符集
$ locale
或
$ echo $LANG
或
$  env |grep LANG
或
# Centos7 字符集配置文件，Centos6 为： cat /etc/sysconfig/i18n
$ cat /etc/locale.conf

# 临时设置字符集
$ LANG=zh_CN.UTF-8

# Centos7 设置字符集永久生效 ，Centos6 为：echo "LANG=zh_CN.UTF-8" > /etc/sysconfig/i18n
$ echo "LANG=zh_CN.UTF-8" > /etc/locale.conf
```

# 文件字符集

```bash
# 查看文件字符集
$ file testString.sh
testString.sh: Bourne-Again shell script, UTF-8 Unicode text executable
或
$ vi testString.sh
:set fileencoding

# 查看文件内容，中文正常输出
$ cat testString.sh |head -2
#!/bin/bash
# 字符串操作符实例

# 使用 iconv 转换文件字符集，iconv -f 原编码 -t 转换后的编码 inputfile -o outputfile
$ iconv -f utf-8 -t utf-16 testString.sh -o testString-utf16.sh

# 查看转换后的字符集
$ file testString-utf16.sh
testString-utf16.sh: Bourne-Again shell script, Little-endian UTF-16 Unicode text executable

# 查看转换为 utf-16 字符集后，中文为乱码
$ cat testString-utf16.sh |head -2
��#!/bin/bash
# W[&{2N�d\O&{�[�O
```

> 微信公众号：daodaotest
