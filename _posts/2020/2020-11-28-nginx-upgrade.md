---
layout: post
title: "生产环境 Nginx 在线平滑升级"
date: "2020-11-28 20:00"
category: Nginx
tags: Nginx
author: jiangliheng
---
* content
{:toc}




# 背景

生产环境 Nginx 需要增加支持 TCP 反向代理功能，需要再添加```--with-stream```参数重新编译后，在线升级 Nginx。

# 在线升级

```bash
# 查看当前版本（注意为大写 V）
$ cd /usr/local/nginx/sbin
$ nginx -V
nginx version: nginx/1.16.1
built by gcc 4.8.5 20150623 (Red Hat 4.8.5-39) (GCC)
built with OpenSSL 1.0.2k-fips  26 Jan 2017
TLS SNI support enabled
configure arguments: --prefix=/usr/local/nginx --user=nginx --group=nginx --with-http_ssl_module

# 安装编译依赖
$ yum -y install gcc pcre pcre-devel zlib zlib-devel openssl openssl-devel

# 下载、解压、并编译（仅编译不安装）
$ wget http://nginx.org/download/nginx-1.16.1.tar.gz
$ tar -zxvf nginx-1.16.1.tar.gz
$ cd nginx-1.16.1
# 增加 --with-stream 编译
# --pid-path 根据各自情况添加，由于 nginx 执行升级命令时，默认是从 nginx/logs 目录下找 nginx.pid
$ ./configure --prefix=/usr/local/nginx --user=nginx --group=nginx --with-http_ssl_module --with-stream --pid-path=/usr/local/nginx/nginx.pid
# -j 指定CPU内核数量，加快编译速度
$ make -j 4

# 备份旧版本，复制新版本
$ mv /usr/local/nginx/sbin/nginx /usr/local/nginx/sbin/nginx.old
$ cp nginx-1.16.1/objs/nginx /usr/local/nginx/sbin/
$ chown nginx:nginx /usr/local/nginx/sbin/nginx

# 升级前查看进程号，确认进程号与 nginx.pid 一致
$ ps -ef | grep nginx
root     11871     1  0 3月24 ?       00:00:00 nginx: master process /usr/local/nginx/sbin/nginx
nginx    13772 11871  0 11月10 ?      00:00:02 nginx: worker process
nginx    13773 11871  0 11月10 ?      00:00:03 nginx: worker process
nginx    13774 11871  0 11月10 ?      00:00:04 nginx: worker process
nginx    13775 11871  0 11月10 ?      00:00:04 nginx: worker process
$ cat /usr/local/nginx/nginx.pid
11871

# 在 nginx1.16.1 目录下，执行 make upgrade 命令自动升级
$ cd nginx-1.16.1
$ make upgrade
# 使用新版本检测配置文件是否正确
/usr/local/nginx/sbin/nginx -t
nginx: the configuration file /usr/local/nginx/conf/nginx.conf syntax is ok
nginx: configuration file /usr/local/nginx/conf/nginx.conf test is successful
# 发送平滑迁移信号给旧的 nginx 进程
kill -USR2 `cat /usr/local/nginx/nginx.pid`
sleep 1
# 检测旧的 nginx.pid 进程是否变为 nginx.pid.oldbin
test -f /usr/local/nginx/nginx.pid.oldbin
# 结束旧版本的进程，完成升级
kill -QUIT `cat /usr/local/nginx/nginx.pid.oldbin`

# 验证是否升级成功，查看 nginx 进程号是否变化（11871-->31845）
$ ps -ef | grep nginx
root     31845     1  0 16:58 ?        00:00:00 nginx: master process /usr/local/nginx/sbin/nginx
nginx    31847 31845  0 16:58 ?        00:00:00 nginx: worker process
nginx    31848 31845  0 16:58 ?        00:00:00 nginx: worker process
nginx    31849 31845  0 16:58 ?        00:00:00 nginx: worker process
nginx    31850 31845  0 16:58 ?        00:00:00 nginx: worker process
$ cat /usr/local/nginx/nginx.pid
31845

# 查看升级后的版本（注意为大写 V）
$ /usr/local/nginx/sbin/nginx -V
nginx version: nginx/1.16.1
built by gcc 4.8.5 20150623 (Red Hat 4.8.5-39) (GCC)
built with OpenSSL 1.0.2k-fips  26 Jan 2017
TLS SNI support enabled
configure arguments: --prefix=/usr/local/nginx --user=nginx --group=nginx --with-http_ssl_module --with-stream --pid-path=/usr/local/nginx/nginx.pid
```

> 微信公众号：daodaotest
