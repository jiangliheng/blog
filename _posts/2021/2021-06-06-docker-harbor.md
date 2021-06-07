---
layout: post
title: "Harbor 2.2.0 搭建与使用"
date: "2021-06-06 20:00"
category: Docker
tags: Docker Harbor
author: jiangliheng
---
* content
{:toc}



# Harbor 简介

> Harbor 是 VMware 公司开源的企业级 Docker Registry 项目，其目标是帮助用户迅速搭建一个企业级的 Docker Registry 服务。
> 它以 Docker 公司开源的 Registry 为基础，提供了管理 UI，基于角色的访问控制(Role Based Access Control)，AD/LDAP 集成、以及审计日志(Audit logging) 等企业用户需求的功能，同时还原生支持中文。

# 搭建 Harbor（master）

> 官方教程：https://goharbor.io/docs/2.2.0/install-config/

Harbor 本地安装支持在线和离线，另外也可以部署到 Kubernetes 中。这里采用本地在线安装方式。

## 先决条件

```
# 配置 docker-ce 的 yum 源
$ cat << EOF > /etc/yum.repos.d/docker-ce.repo
[docker-ce-stable]
name=Docker CE Stable - \$basearch
baseurl=https://mirrors.aliyun.com/docker-ce/linux/centos/7/\$basearch/stable
enabled=1
gpgcheck=1
gpgkey=https://mirrors.aliyun.com/docker-ce/linux/centos/gpg
EOF

# 安装 docker（17.06.0-ce+） 和 docker-compose（1.18.0+）
$ sudo yum install -y docker-ce docker-ce-cli containerd.io bash-completion docker-compose

# 配置 docker 自动提示
$ cp /usr/share/bash-completion/completions/docker /etc/bash_completion.d/

# 配置开机启动
$ systemctl enable --now docker

# 查看安装版本
$ docker --version
Docker version 20.10.6, build 370c289
$ docker-compose --version
docker-compose version 1.18.0, build 8dd22a9
```

## 安装

```
# 下载在线安装包
$ cd /usr/local
$ curl -O -L https://github.com/goharbor/harbor/releases/download/v2.2.2/harbor-online-installer-v2.2.2.tgz

# 解压
$ tar -zxvf harbor-online-installer-v2.2.2.tgz

# 生成 CA 证书
$ cd harbor && mkdir cert && cd cert
$ openssl genrsa -out ca.key 4096
$ openssl req -x509 -new -nodes -sha512 -days 3650 -subj "/C=CN/ST=Beijing/L=Beijing/O=example/OU=Personal/CN=daodaotest.com" -key ca.key -out ca.crt

# 生成服务证书
$ openssl genrsa -out daodaotest.com.key 4096
$ openssl req -sha512 -new -subj "/C=CN/ST=Beijing/L=Beijing/O=example/OU=Personal/CN=daodaotest.com" -key daodaotest.com.key -out daodaotest.com.csr
$ cat > v3.ext <<-EOF
authorityKeyIdentifier=keyid,issuer
basicConstraints=CA:FALSE
keyUsage = digitalSignature, nonRepudiation, keyEncipherment, dataEncipherment
extendedKeyUsage = serverAuth
subjectAltName = @alt_names

[alt_names]
IP.1 = 172.17.167.181
DNS.1=daodaotest.com
EOF
$ openssl x509 -req -sha512 -days 3650 -extfile v3.ext -CA ca.crt -CAkey ca.key -CAcreateserial -in daodaotest.com.csr -out daodaotest.com.crt

# 生成 docker 证书
$ openssl x509 -inform PEM -in daodaotest.com.crt -out daodaotest.com.cert
$ mkdir -p /etc/docker/certs.d/daodaotest.com/
$ cp daodaotest.com.cert /etc/docker/certs.d/daodaotest.com/
$ cp daodaotest.com.key /etc/docker/certs.d/daodaotest.com/
$ cp ca.crt /etc/docker/certs.d/daodaotest.com/

# 重启 docker
$ sudo systemctl restart docker

# 修改配置文件
$ cd .. && cp harbor.yml.tmpl harbor.yml
# 修改内容如下
$ diff harbor.yml harbor.yml.tmpl
5c5
< hostname: daodaotest.com
---
> hostname: reg.mydomain.com
17,18c17,18
<   certificate: /usr/local/harbor/cert/daodaotest.com.crt
<   private_key: /usr/local/harbor/cert/daodaotest.com.key
---
>   certificate: /your/certificate/path
>   private_key: /your/private/key/path
34c34
< harbor_admin_password: 8XHeH5bC6i6bTttZ
---
> harbor_admin_password: Harbor12345
39c39
<   password: TbZC8gBss5A7DedM
---
>   password: root123

# 初始化配置
$ sudo mkdir /data
$ sudo ./prepare
prepare base dir is set to /usr/local/harbor
Unable to find image 'goharbor/prepare:v2.2.2' locally
v2.2.2: Pulling from goharbor/prepare
b31150c04016: Pull complete
d504272addf9: Pull complete
a9c2d9be0ec7: Pull complete
ba14108b237f: Pull complete
888a2dd12a77: Pull complete
08591f736052: Pull complete
e9a06c50605c: Pull complete
fcc257111f80: Pull complete
Digest: sha256:d12185f2c925416fa260d2af8764d8c27d35b4f66d9bcff67bf7e35d9409789e
Status: Downloaded newer image for goharbor/prepare:v2.2.2
Generated configuration file: /config/portal/nginx.conf
Generated configuration file: /config/log/logrotate.conf
Generated configuration file: /config/log/rsyslog_docker.conf
Generated configuration file: /config/nginx/nginx.conf
Generated configuration file: /config/core/env
Generated configuration file: /config/core/app.conf
Generated configuration file: /config/registry/config.yml
Generated configuration file: /config/registryctl/env
Generated configuration file: /config/registryctl/config.yml
Generated configuration file: /config/db/env
Generated configuration file: /config/jobservice/env
Generated configuration file: /config/jobservice/config.yml
loaded secret from file: /data/secret/keys/secretkey
Generated configuration file: /compose_location/docker-compose.yml
Clean up the input dir

# 安装，包括 Notary, Trivy, 和 Chart Repository Service
$ sudo ./install.sh --with-notary --with-trivy --with-chartmuseum

[Step 0]: checking if docker is installed ...

Note: docker version: 20.10.6

[Step 1]: checking docker-compose is installed ...

Note: docker-compose version: 1.18.0


[Step 2]: preparing environment ...

[Step 3]: preparing harbor configs ...
prepare base dir is set to /usr/local/harbor
Clearing the configuration file: /config/registry/config.yml
Clearing the configuration file: /config/registry/passwd
Clearing the configuration file: /config/db/env
Clearing the configuration file: /config/portal/nginx.conf
Clearing the configuration file: /config/jobservice/config.yml
Clearing the configuration file: /config/jobservice/env
Clearing the configuration file: /config/core/app.conf
Clearing the configuration file: /config/core/env
Clearing the configuration file: /config/registryctl/config.yml
Clearing the configuration file: /config/registryctl/env
Clearing the configuration file: /config/log/logrotate.conf
Clearing the configuration file: /config/log/rsyslog_docker.conf
Clearing the configuration file: /config/nginx/nginx.conf
Generated configuration file: /config/portal/nginx.conf
Generated configuration file: /config/log/logrotate.conf
Generated configuration file: /config/log/rsyslog_docker.conf
Generated configuration file: /config/nginx/nginx.conf
Generated configuration file: /config/core/env
Generated configuration file: /config/core/app.conf
Generated configuration file: /config/registry/config.yml
Generated configuration file: /config/registryctl/env
Generated configuration file: /config/registryctl/config.yml
Generated configuration file: /config/db/env
Generated configuration file: /config/jobservice/env
Generated configuration file: /config/jobservice/config.yml
loaded secret from file: /data/secret/keys/secretkey
Successfully called func: create_root_cert
Successfully called func: create_cert
Copying certs for notary signer
Copying nginx configuration file for notary
Generated configuration file: /config/nginx/conf.d/notary.upstream.conf
Generated configuration file: /config/nginx/conf.d/notary.server.conf
Generated configuration file: /config/notary/server-config.postgres.json
Generated configuration file: /config/notary/server_env
Generated and saved secret to file: /data/secret/keys/defaultalias
Generated configuration file: /config/notary/signer_env
Generated configuration file: /config/notary/signer-config.postgres.json
Generated configuration file: /config/trivy-adapter/env
Generated configuration file: /config/chartserver/env
Generated configuration file: /compose_location/docker-compose.yml
Clean up the input dir

[Step 4]: starting Harbor ...
Creating network "harbor_harbor" with the default driver
Creating network "harbor_harbor-notary" with the default driver
Creating network "harbor_harbor-chartmuseum" with the default driver
Creating network "harbor_notary-sig" with the default driver
Pulling core (goharbor/harbor-core:v2.2.2)...
v2.2.2: Pulling from goharbor/harbor-core
b31150c04016: Already exists
4fd558bf3277: Already exists
09fd06630656: Already exists
c8359bc44335: Already exists
3e3f85560d2f: Already exists
6613976eb94c: Already exists
06a00c38c3fa: Already exists
2c5b1b654d3f: Already exists
1463750ae243: Already exists
e2fa58294c0c: Already exists
Digest: sha256:6a2a8c05dfe088c14700853683a5856697e82b74ab35990c1df15cf323ae739c
Status: Downloaded newer image for goharbor/harbor-core:v2.2.2
Pulling jobservice (goharbor/harbor-jobservice:v2.2.2)...
v2.2.2: Pulling from goharbor/harbor-jobservice
b31150c04016: Already exists
e90b8292d722: Pull complete
21570fc83884: Pull complete
2ac76f3a2cdc: Pull complete
a194c99570b8: Pull complete
59c65e440e8e: Pull complete
Digest: sha256:6206e3eed55177102832be4c258f060483d233b407ee0f1d0c2d1ce65f7acb4b
Status: Downloaded newer image for goharbor/harbor-jobservice:v2.2.2
Pulling proxy (goharbor/nginx-photon:v2.2.2)...
v2.2.2: Pulling from goharbor/nginx-photon
b31150c04016: Already exists
ea22aad1496e: Pull complete
Digest: sha256:ea7e2a056d4ae18165f116397e7f6473c6fce21ee7078d3bff0e966abcdb38cd
Status: Downloaded newer image for goharbor/nginx-photon:v2.2.2
Pulling notary-signer (goharbor/notary-signer-photon:v2.2.2)...
v2.2.2: Pulling from goharbor/notary-signer-photon
b31150c04016: Already exists
77b0eeb6bb5b: Pull complete
ad12ce7b7d07: Pull complete
9c78f39afcfe: Pull complete
c7591a9d8a65: Pull complete
10a898710e5d: Pull complete
8559810b9178: Pull complete
Digest: sha256:e963210826b2d0a31071de6d47cf470f58505c47a0c01f722fd90f3b4c88f273
Status: Downloaded newer image for goharbor/notary-signer-photon:v2.2.2
Pulling notary-server (goharbor/notary-server-photon:v2.2.2)...
v2.2.2: Pulling from goharbor/notary-server-photon
b31150c04016: Already exists
49a1181de268: Pull complete
57b1fa698760: Pull complete
fae278e6af1f: Pull complete
10cbb7ccfd1f: Pull complete
30c63c551bf3: Pull complete
011baa64e627: Pull complete
Digest: sha256:116ae7af80e59f5b740659d7e6337cb8477a72ded5dea48b4525b8845dcc1f07
Status: Downloaded newer image for goharbor/notary-server-photon:v2.2.2
Pulling trivy-adapter (goharbor/trivy-adapter-photon:v2.2.2)...
v2.2.2: Pulling from goharbor/trivy-adapter-photon
b31150c04016: Already exists
fb0481cd4216: Pull complete
e42944e3b258: Pull complete
48930f550697: Pull complete
f907b5a107a9: Pull complete
e92ef87c1a88: Pull complete
58a6884cd2da: Pull complete
Creating harbor-log ... done
Status: Downloaded newer image for goharbor/trivy-adapter-photon:v2.2.2
Pulling chartmuseum (goharbor/chartmuseum-photon:v2.2.2)...
v2.2.2: Pulling from goharbor/chartmuseum-photon
b31150c04016: Already exists
767fbf17fa65: Pull complete
Creating redis ... done
Creating chartmuseum ... done
d7d45173a427: Pull complete
Creating notary-signer ... done
Creating harbor-core ... done
Status: Downloaded newer image for goharbor/chartmuseum-photon:v2.2.2
Creating nginx ... done
Creating harbor-db ...
Creating redis ...
Creating registry ...
Creating registryctl ...
Creating harbor-portal ...
Creating chartmuseum ...
Creating trivy-adapter ...
Creating harbor-core ...
Creating notary-signer ...
Creating notary-server ...
Creating harbor-jobservice ...
Creating nginx ...
✔ ----Harbor has been installed and started successfully.----
```

## 启停服务

```
# 进去配置目录
$ cd /usr/local/harbor

# 构建并后台启动容器
$ docker-compose up -d

# 查看服务
$ docker-compose ps

# 启动
$ docker-compose start

# 停止
$ docker-compose stop

# 重启
$ docker-compose restart
```

## 登录

```
# 本地添加域名映射
$ sudo echo "172.17.167.181 daodaotest.com" >> /etc/hosts

# docker 登录， 输入密码：8XHeH5bC6i6bTttZ
$ docker login -u admin https://daodaotest.com
```

登录地址：https://daodaotest.com 或 https://172.17.167.181
用户名/密码：admin / 8XHeH5bC6i6bTttZ

PS：访问 https://daodoatest.com 时，会提示证书不受信任

## 重置 Harbor

```
# 停止 harbor 服务并删除容器
$ docker-compose down -v

# 删除相关数据
$ rm -rf /var/log/harbor/
$ rm -rf /data/database
$ rm -rf /data/registry
```

# 配置使用

## 配置 docker 证书（node）

```
# 各 node 节点，在本地添加域名映射
$ sudo echo "172.17.167.181 daodaotest.com" >> /etc/hosts

#  从 master 复制 docker 证书到本地
$ mkdir -p /etc/docker/certs.d/daodaotest.com/
$ scp root@172.17.167.181:/etc/docker/certs.d/daodaotest.com/* /etc/docker/certs.d/daodaotest.com/
```

## 代理 Docker Hub

> 代理仓库仅能 pull，不能 push

- 用户管理--创建用户：```test```
![](/assets/images/harbor/16215084980552.jpg)

- 仓库管理--创建```Docker Hub```目标
![](/assets/images/harbor/16215076204697.jpg)

- 项目--新建项目：```docker-hub```，镜像代理选中上面创建的```Docker Hub```目标
![](/assets/images/harbor/16215079266526.jpg)

- 将用户```test```加入```docker-hub```项目中，设置为```项目管理员```角色
![](/assets/images/harbor/16215080267042.jpg)

- 通过代理拉取```Docker Hub```中的```hello-world```镜像

```
# 通过代理拉取镜像
$ docker pull daodaotest.com/docker-hub/library/hello-world
Using default tag: latest
latest: Pulling from docker-hub/library/hello-world
b8dfde127a29: Pull complete
Digest: sha256:5122f6204b6a3596e048758cabba3c46b1c937a46b5be6225b835d091b90e46c
Status: Downloaded newer image for daodaotest.com/docker-hub/library/hello-world:latest
daodaotest.com/docker-hub/library/hello-world:latest
```

- Harbor UI 查看拉取的```hello-world```镜像
![](/assets/images/harbor/16215086880016.jpg)

## 提交镜像

- 项目--创建项目：```daodaotest```
- 将用户```test```加入```daodaotest```项目中，设置为```项目管理员```角色
- 提交镜像

```
# 登录 Harbor
$ docker login -u test https://daodaotest.com

# 提交镜像
$ docker tag hello-world daodaotest.com/daodaotest/hello-world
$ docker push daodaotest.com/daodaotest/hello-world
Using default tag: latest
The push refers to repository [daodaotest.com/daodaotest/hello-world]
f22b99068db9: Layer already exists
latest: digest: sha256:1b26826f602946860c279fce658f31050cff2c596583af237d971f4629b57792 size: 525
```

- Harbor UI 查看镜像
![](/assets/images/harbor/16215087718065.jpg)


> 微信公众号：daodaotest
