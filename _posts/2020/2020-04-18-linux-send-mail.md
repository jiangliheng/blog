---
layout: post
title: "Linux 下发送邮件"
date: "2020-04-18 20:30"
category: Linux
tags: Linux mail mailx
author: jiangliheng
---
* content
{:toc}

由于种种原因，需要由我这个兼职运维每天发送对账单文件给运营同学，故研究下 Linux 发送邮件，希望对大家有所帮助。



# 安装

```bash
# Centos，安装 mailx
$ yum install -y mailx

# 查看帮助
$ mail --h
```

# SSL 证书

配置 SSL 证书，否则会提示 “Error in certificate: Peer’s certificate issuer is not recognized.”。

```bash
# 生成证书
$ mkdir ~/.certs
$ echo -n | openssl s_client -connect smtp.mxhichina.com:465 | sed -ne '/-BEGIN CERTIFICATE-/,/-END CERTIFICATE-/p' > ~/.certs/ali.crt
$ certutil -A -n "GeoTrust SSL CA" -t "C,," -d ~/.certs -i ~/.certs/ali.crt
$ certutil -A -n "GeoTrust Global CA" -t "C,," -d ~/.certs -i ~/.certs/ali.crt

# 查看证书权限
$ cd ~/.certs && ll
总用量 80
-rw-r--r-- 1 root root  2277 4月  19 10:47 ali.crt
-rw------- 1 root root 65536 4月  19 10:56 cert8.db
-rw------- 1 root root 16384 4月  19 10:56 key3.db
-rw------- 1 root root 16384 4月  19 10:48 secmod.db

# 验证，显示如下信息表示 SSL 证书配置生成及安装完成
$ certutil -A -n "GeoTrust SSL CA - G3" -t "Pu,Pu,Pu" -d ./ -i ali.crt
Notice: Trust flag u is set automatically if the private key is present.
```

生成完成之后，需要在 ```/etc/mail.rc``` 配置文件中，修改 ```nss-config-dir``` 为上面命令生成的目录 ```~/.certs```。

# 配置文件
以下以阿里云邮箱配置为例，其他邮箱类似。

```bash
# 配置邮箱参数，文件末尾添加以下内容
$ vi /etc/mail.rc
# ssl 校验设置，配置 SSL 证书就可以注释掉
# set ssl-verify=ignore
# 邮箱账户，对方收到邮件时显示的发件人
set from=daodaotest@xxx.com
# smtp 服务器地址，不要忘记添加：smtps
set smtp=smtps://smtp.mxhichina.com:465
# 邮箱账户
set smtp-auth-user=daodaotest@xxx.com
# 邮箱密码，部分邮箱（163）为授权密码而非邮箱密码
set smtp-auth-password=xxxxx
# smtp 认证方式。默认是 login
set smtp-auth=login
# 设置 nss 配置目录，上一步骤 SSL 证书目录
set nss-config-dir=~/.certs/
```

# 使用

```bash
# 发送邮件
$ echo "邮件内容" | mail -s "邮件标题" daodaotest@163.com

# 发送邮件，添加抄送人及附件
echo "邮件内容，请查收" | mail -v -c "daodaotest@163.com，daodaotest@qq.com" -s "邮件标题" -a daodaotest.zip daodaotest@163.com
```

参数说明：
- ```-s <邮件主题>```：指定邮件的主题；
- ```-c <地址>```：指定抄送人，多个收件人之间用逗号分隔；
- ```-b <地址>```：指定密送人，多个收件人之间用逗号分隔；
- ```-a```：参数后面跟的文件，将作为附件发送出去；
- ```-v```：执行时，显示详细的信息。

更多参数说明使用 ```man mailx``` 命令查看。

# 问题
## Unexpected EOF on SMTP connection

我个人遇到的问题是设置 ```smtp``` 参数时，没有添加 ```smtps``` 协议。

```
# /etc/mail.rc 之前配置
set smtp=smtp.mxhichina.com:465

# /etc/mail.rc 修改后的配置
set smtp=smtps://smtp.mxhichina.com:465
```

## Error in certificate: Peer's certificate issuer has been marked as not trusted by the.

没有配置 ```SSL``` 证书，具体配置见[生成 SSL 证书](#生成 SSL 证书)。

# 使用场景

## 定时给运营同学发送对账单文件

```bash
# 脚本内容
$ cat sendRecFile.sh
#!/bin/bash
# 定时给运营同学发送对账单文件

# 使用方法
usage() {
	printf "Usage: sh %s RE_USERS CC_USERS [DAY]" "$0"
	printf "\n"
	printf "\n\t RE_USERS 收件人，多个收件人之间用逗号分隔"
	printf "\n\t CC_USERS 抄送人，多个收件人之间用逗号分隔"
	printf "\n\t DAY 发送对账文件日期，默认为：T-1"
}

# 判断参数
if [ $# -lt 2 ]; then
	usage
	exit 1
fi

# 收件人
RE_USERS=$1
# 抄送人
CC_USERS=$2
# 对账文件日期
DAY=$3
if [ "X$DAY" == "X" ]; then
	DAY=$(date -d yesterday +%Y%m%d)
fi

# 对账文件路径
RE_PATH="/data/ftp/PE/RUI/response"

# 确定 ‘zip’ 命令位置
ZIPEXE="/usr/bin/zip"
if [ ! -x "${ZIPEXE}" ]; then
	ZIPEXE="/usr/bin/zip"
	if [ ! -x "${ZIPEXE}" ]; then
		printf "Unable to locate 'zip'."
		printf "Please report this message along with the location of the command on your system."
		exit 1
	fi
fi

# 确定 ‘mail’ 命令位置
MAILEXE="/usr/bin/mail"
if [ ! -x "${MAILEXE}" ]; then
	MAILEXE="/usr/bin/mail"
	if [ ! -x "${MAILEXE}" ]; then
		printf "Unable to locate 'mail'."
		printf "Please report this message along with the location of the command on your system."
		exit 1
	fi
fi

# 打包对账单
if [ -d "${RE_PATH}/${DAY}" ]; then
	cp -r "${RE_PATH}/${DAY}" /tmp
	cd /tmp || exit
	${ZIPEXE} -r "${DAY}.zip" "${DAY}"
else
	printf "Cannot find %s." "${RE_PATH}/${DAY}"
	exit 1
fi

# 发送邮件
printf "xx，您好： \n\n 附件为 %s 对账单文件，请查收。\n" "${DAY}" | ${MAILEXE} -c "${CC_USERS}" -s "[定时自动发送] ${DAY} 对账单文件 " -a "/tmp/${DAY}.zip" "${RE_USERS}"

# 发送完成，删除 zip
if [ -f "/tmp/${DAY}.zip" ]; then
	rm -rf "/tmp/${DAY}.zip"
fi

exit 0

# 使用方法
$ sh sendRecFile.sh
Usage: sh ./sendRecFile.sh RE_USERS CC_USERS [DAY]

         RE_USERS 收件人，多个收件人之间用逗号分隔
         CC_USERS 抄送人，多个收件人之间用逗号分隔
         DAY 发送对账文件日期，默认为：T-1

# 定时任务设置
$ crontab -e
0 9 * * * /root/ops-scripts/sendRecFile.sh daodaotest@163.com daodaotest@163.com  >>/root/ops-scripts/sendRecFile.log
```

> 微信公众号：daodaotest
