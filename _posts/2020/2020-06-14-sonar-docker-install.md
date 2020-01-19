---
layout: post
title: "Docker Compose 方式安装 SonarQube 8.3.1"
date: "2020-06-14 02:00"
category: SonarQube
tags: SonarQube PostgreSQL docker docker-compose
author: jiangliheng
---
* content
{:toc}



# 验证环境

- Centos 7.7
- Docker 1.13.1
- docker-compose 1.18.0
- SonarQube 8.3.1.34397
- postgreSQL 12.3-1.pgdg100+1

# 前提

由于 SonarQube 使用 ```Elasticsearch``` 作为全文模糊搜索引擎，故需要设置如下内核参数。

```bash
# 查看
$ sysctl vm.max_map_count
$ sysctl fs.file-max
$ ulimit -n
$ ulimit -u

# 实时修改生效
$ sysctl -w vm.max_map_count=262144
$ sysctl -w fs.file-max=65536
$ ulimit -n 65536
$ ulimit -u 4096

# 永久生效
$ echo "sonar   -   nofile   65536
sonar   -   nproc    4096" > /etc/security/limits.d/99-sonarqube.conf
$ echo "vm.max_map_count=262144
fs.file-max=65536" > /etc/sysctl.d/99-sonarqube.conf
```

# 安装

```bash
# 安装 docker docker-compose
$ yum install -y docker docker-compose

# 启动 docker
$ systemctl start docker

# 拉取 postgres 镜像
$ docker pull postgres

# 拉取 sonarqube 镜像
$ docker pull sonarqube

# 查看下载的镜像
$ docker images
```

# Docker Compose 配置

采用 Docker Compose 方式配置 SonarQube。

```yaml
$ docker-compose.yml
version: '3'
services:
  postgres:
    image: postgres
    restart: always
    container_name: postgres
    ports:
      - 5432:5432
    volumes:
      - /home/sonar/postgres/postgresql:/var/lib/postgresql
      - /home/sonar/postgres/data:/var/lib/postgresql/data
    environment:
      TZ: Asia/Shanghai
      POSTGRES_USER: sonar
      POSTGRES_PASSWORD: sonar
      POSTGRES_DB: sonar

  sonar:
    image: sonarqube
    container_name: sonar
    depends_on:
      - postgres
    volumes:
      - /home/sonar/sonarqube/extensions:/opt/sonarqube/extensions
      - /home/sonar/sonarqube/logs:/opt/sonarqube/logs
      - /home/sonar/sonarqube/data:/opt/sonarqube/data
      - /home/sonar/sonarqube/conf:/opt/sonarqube/conf
    ports:
      - 9000:9000
    command:
      # 内存设置
      - -Dsonar.ce.javaOpts=-Xmx2048m
      - -Dsonar.web.javaOpts=-Xmx2048m
      # nginx 反向代理
      - -Dsonar.web.context=/sonar
      # crowd 集成，实现统一登录
      #- -Dsonar.security.realm=Crowd
      #- -Dcrowd.url=http://x.x.x.x:8095/crowd
      #- -Dcrowd.application=sonar
      #- -Dcrowd.password=xxxxxx
      #- -Dsonar.security.localUsers=admin
    environment:
      SONARQUBE_JDBC_USERNAME: sonar
      SONARQUBE_JDBC_PASSWORD: sonar
      SONARQUBE_JDBC_URL: jdbc:postgresql://postgres:5432/sonar
```

# 启停

```bash
# 切换到 sonar 用户
$ su - sonar

# 构建并后台启动容器
$ docker-compose up -d

# 查看运行容器
$ docker-compose ps
或
$ docker ps

# 动态查看日志
$ docker-compose logs -f
或
$ docker logs -f sonar

# 重启
$ docker-compose restart
```

# 访问

Nginx 反向代理设置。

```
location ^~ /sonar {

        proxy_pass http://x.x.x.x:9000/sonar;

        sendfile off;

        proxy_set_header   Host             $host:$server_port;
        proxy_set_header   X-Real-IP        $remote_addr;
        proxy_set_header   X-Forwarded-For  $proxy_add_x_forwarded_for;
        proxy_max_temp_file_size 0;

        # This is the maximum upload size
        client_max_body_size       50m;
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

IP访问地址：http://x.x.x.x:9000/sonar

域名访问地址：http://xxx.com/sonar

> 微信公众号：daodaotest
