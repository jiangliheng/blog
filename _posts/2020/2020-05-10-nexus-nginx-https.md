---
layout: post
title: "Nexus 使用 nginx 代理支持 HTTPS 协议"
date: "2020-05-10 22:00"
category: Nexus
tags: Nexus Nginx HTTPS
author: jiangliheng
---
* content
{:toc}




# 背景

公司全部网站需要支持 HTTPS 协议，在阿里云负载均衡配置 SSL 证书后，导致 Nexus 的 HTTPS 访问出错。

**网站访问路径**： 域名解析到阿里云的负载均衡，负载均衡配置 80 端口强转 443 端口，443 端口配置 SSL 证书，并转发到内网 nginx，内网的 nginx 再代理 Nexus 服务。

# 解决

浏览器 HTTPS 访问 Nexus 的 Console 报错信息：
![-w1393](/assets/images/nexus/15891208611164.jpg)

报错信息大致意思是：HTTPS 访问的页面上不允许出现 HTTP 请求。


**解决方法**： 在 nginx 配置文件增加 “proxy_set_header   X-Forwarded-Proto https;” ，这样 nginx 在转发时就使用 HTTPS 协议。


nginx.conf 中的 nexus 配置内容：
```
location ^~ /nexus {

        proxy_pass http://x.x.x.x:8080/nexus;

        sendfile off;

        proxy_set_header   Host             $host;
        proxy_set_header   X-Real-IP        $remote_addr;
        proxy_set_header   X-Forwarded-For  $proxy_add_x_forwarded_for;
	proxy_set_header   X-Forwarded-Proto https;  # 转发时使用https协议
        proxy_max_temp_file_size 0;

        # This is the maximum upload size
        client_max_body_size       20m;
        client_body_buffer_size    128k;

        proxy_connect_timeout      90;
        proxy_send_timeout         90;
        proxy_read_timeout         90;

        proxy_temp_file_write_size 64k;

        # Required for new HTTP-based CLI
        proxy_http_version 1.1;
        proxy_request_buffering off;
        proxy_buffering off; # Required for HTTP-based CLI to work over SSL
    }
```

> 微信公众号：daodaotest
