---
layout: post
title: "Atlassian 系列软件安装（Crowd+JIRA+Confluence+Bitbucket+Bamboo）"
date: "2020-03-28 22:00"
category: Atlassian
tags: Atlassian Crowd JIRA Confluence Bitbucket Bamboo
author: jiangliheng
---
* content
{:toc}

公司使用的软件开发和协作工具为 Atlassian 系列软件，近期需要从腾讯云迁移到阿里云环境，简单记录下安装和配置过程。（Atlassian 的文档非常详尽，过程中碰见的问题都可以找到解决办法。）



# 简介

 名称 | 简介
 :--- | :---
 Crowd | 易于使用、管理和集成的单点登录和身份管理工具。除了支持 Atlassian 系列软件，也支持 SonarQube，Jenkins，Nexus 等
 JIRA | 使用敏捷团队的首选软件开发工具，规划、追踪和发布世界一流的软件。
 Confluence | 可减少东找西找所花的时间，将更多的时间用在完成工作上。可在同一位置整理工作、创建文档并讨论一切内容。
 Bitbucket | 通过内嵌的评论和拉取请求协作编写代码。整个团队管理并共享 Git 代码库以构建和交付软件。
 Bamboo | 持续集成、部署和发布管理。

# 注意事项

写在最前面，避免安装过程中的坑坑坑。

**友情提示：安装过程中碰见任何问题，直接上 google 或者 Atlassian 官网搜索，一般都有详细的文档支持**

**操作系统字符集**：数据备份迁移时，可能会出现未知错误，如： Crowd 备份导入时，会出现日期转换错误。

**Mysql 驱动**：支持 Mysql 数据库，但是未集成 Mysql jdbc 驱动，请提前准备。

**Mysql 字符集**：库表字符集：utf8，排序字符集：utf8_bin。

阿里云 RDS 控制台，创建的 UTF8 数据库，默认排序字符集为：utf8_general_ci，需要修改为：utf8_bin。

**Git 版本**：安装 Bitbucket 时，Git 版本需要是 2.2.0+。

# 安装环境

软件 | 版本 | 说明
:--- | :--- | :---
Centos | V7.7 | 阿里云 ECS
Oracle JDK | V1.8.0_171 |
Git | V2.8.3 | Bitbucket 依赖 Git 2.2.0+
Mysql | V5.7 | 阿里云 RDS

# Crowd
## Crowd 安装

```bash
# 创建独立安装账号
$ useradd crowd
$ passwd crowd
$ su - crowd

# 下载
$ wget https://product-downloads.atlassian.com/software/crowd/downloads/atlassian-crowd-3.2.3.tar.gz

# 解压
$ tar -zxvf atlassian-crowd-3.2.3.tar.gz

# 设置 crowd.home
$ vi /home/crowd/atlassian-crowd-3.2.3/crowd-webapp/WEB-INF/classes/crowd-init.properties
###############
##           ##
##  UNIX     ##
##           ##
###############
## On Unix-based operating systems, uncomment the following
## line and set crowd.home to a directory Crowd should use to
## store its configuration.

crowd.home=/home/crowd/atlassian-crowd-3.2.3

# Crowd 支持 Mysql 数据库，但是未集成 Mysql jdbc 驱动
$ cp mysql-connector-java-5.1.46.jar /home/crowd/atlassian-crowd-3.2.3/apache-tomcat/lib

# 启动 crowd
$ sh /home/crowd/atlassian-crowd-3.2.3/start_crowd.sh
```

## Crowd 设置

1. 浏览器中访问 http://yourip:8095 进入初始化页面，输入 License
![-w1309](/assets/images/atlassian/15851050826760.jpg)


2. 选择导入备份
![-w1255](/assets/images/atlassian/15851057275615.jpg)

3. 设置数据库信息
![-w1225](/assets/images/atlassian/15851057539463.jpg)

4. 参数设置
![-w1179](/assets/images/atlassian/15851139900001.jpg)

5. 设置管理员账号
![-w1190](/assets/images/atlassian/15851140339975.jpg)

6. 设置完成
![-w1225](/assets/images/atlassian/15851141200825.jpg)
![-w1172](/assets/images/atlassian/15851141439296.jpg)

## Crowd 迁移

1. 备份之前 Crowd 数据
![-w1381](/assets/images/atlassian/15851056788692.jpg)

2. 导入备份数据即可
![-w1194](/assets/images/atlassian/15851066700134.jpg)
![-w1248](/assets/images/atlassian/15851082274090.jpg)

**操作系统字符集不一致问题**

![-w1354](/assets/images/atlassian/15851070484703.jpg)

```
# 修改操作系统字符集一致
$ echo 'LANG=zh_CN.UTF-8' > /etc/locale.conf
$ source /etc/locale.conf
```

# JIRA
## JIRA 安装

```bash
# 创建独立安装账号
$ useradd jira
$ passwd jira
$ su - jira

# 下载
$ wget https://product-downloads.atlassian.com/software/jira/downloads/atlassian-jira-software-7.10.0-x64.bin

# 安装，一路回车即可
$ ./atlassian-jira-software-7.10.0-x64.bin
Unpacking JRE ...
Starting Installer ...
三月 26, 2020 7:00:44 下午 java.util.prefs.FileSystemPreferences$1 run
信息: Created user preferences directory.
三月 26, 2020 7:00:44 下午 java.util.prefs.FileSystemPreferences$2 run
信息: Created system preferences directory in java.home.
You do not have administrator rights to this machine and as such, some installation options will not be available. Are you sure you want to continue?
Yes [y, Enter], No [n]
y

This will install JIRA Software 7.10.0 on your computer.
OK [o, Enter], Cancel [c]
o
Choose the appropriate installation or upgrade option.
Please choose one of the following:
Express Install (use default settings) [1], Custom Install (recommended for advanced users) [2, Enter], Upgrade an existing JIRA installation [3]
2

Where should JIRA Software be installed?
[/home/jira/atlassian/jira]

Default location for JIRA Software data
[/home/jira/atlassian/application-data/jira]

Configure which ports JIRA Software will use.
JIRA requires two TCP ports that are not being used by any other
applications on this machine. The HTTP port is where you will access JIRA
through your browser. The Control port is used to startup and shutdown JIRA.
Use default ports (HTTP: 8080, Control: 8005) - Recommended [1, Enter], Set custom value for HTTP and Control ports [2]

Details on where JIRA Software will be installed and the settings that will be used.
Installation Directory: /home/jira/atlassian/jira
Home Directory: /home/jira/atlassian/application-data/jira
HTTP Port: 8080
RMI Port: 8005
Install as service: No
Install [i, Enter], Exit [e]

Extracting files ...

Please wait a few moments while JIRA Software is configured.
Installation of JIRA Software 7.10.0 is complete
Start JIRA Software 7.10.0 now?
Yes [y, Enter], No [n]
y

Please wait a few moments while JIRA Software starts up.
Launching JIRA Software ...
Installation of JIRA Software 7.10.0 is complete
Your installation of JIRA Software 7.10.0 is now ready and can be accessed
via your browser.
JIRA Software 7.10.0 can be accessed at http://localhost:8080
Finishing installation ...

# 与 Crowd 类似，需要把 Mysql 驱动包 放到 atlassian/jira/lib 目录下，需要重启生效。
$ cp mysql-connector-java-5.1.46.jar /home/jira/atlassian/jira/lib

# 重启
$ sh /home/jira/atlassian/jira/bin/stop-jira.sh
$ sh /home/jira/atlassian/jira/bin/start-jira.sh
```

## JIRA 设置

设置过程与 Crowd 类似。
1. 浏览器中访问 http://yourip:8080 进入初始化页面；
2. 选择自定义设置，选择数据库 Mysql（utf8 字符集，utf8_bin 排序规则），输入相关参数，测试连接通过，点击下一步；
3. 输入license，设置管理员账号，点击下一步；
4. 跳过邮件通知，设置语言、头像，就可以正式使用 JIRA 了

# Confluence
## Confluence 安装

```bash
# 创建独立安装账号
$ useradd confluence
$ passwd confluence
$ su - confluence

# 下载
$ wget https://product-downloads.atlassian.com/software/confluence/downloads/atlassian-confluence-6.9.1-x64.bin

# 安装过程与 JIRA 类似
$ ./atlassian-confluence-6.9.1-x64.bin

# 与 JIRA 类似，需要把 Mysql 驱动包 放到 /home/confluence/atlassian/confluence/confluence/WEB-INF/lib 目录下，需要重启生效。
$ cp mysql-connector-java-5.1.46.jar /home/confluence/atlassian/confluence/confluence/WEB-INF/lib

# 重启
$ sh /home/confluence/atlassian/confluence/bin/stop-confluence.sh
$ sh /home/confluence/atlassian/confluence/bin/stop-confluence.sh
```

## Confluence 设置

设置过程与 JIRA 类似。
1. 浏览器中访问 http://yourip:8090 进入初始化页面；
2. 选择生产安装，根据情况选择附加插件，点击下一步；
3. 输入license，选择数据库 Mysql（utf8 字符集，utf8_bin 排序规则），输入相关参数，测试连接通过，点击下一步；
4. 选择初始内容，这里可以选择 “Example Site”，会初始新建示例项目；
5. 选择用户策略，因为我们后续会集成 Crowd，所以这里选择 “Manage users and groups within Confluence”；
6. 设置管理员账户，就正式使用 Confluence 了。

# Bitbucket
## Bitbucket 安装

```bash
# 创建独立安装账号
$ useradd bitbucket
$ passwd bitbucket
$ su - bitbucket

# 下载
$ wget https://product-downloads.atlassian.com/software/stash/downloads/atlassian-bitbucket-5.11.1-linux-x64.bin

# 安装前提条件：Git 版本需要 2.2.0+
# 安装过程与 JIRA 和 Confluence 类似
$ ./atlassian-bitbucket-5.11.1-linux-x64.bin

# 与 JIRA 类似，需要把 Mysql 驱动包 放到 /home/bitbucket/atlassian/bitbucket/5.11.1/app/WEB-INF/lib 目录下，需要重启生效。
$ cp mysql-connector-java-5.1.46.jar /home/bitbucket/atlassian/bitbucket/5.11.1/app/WEB-INF/lib

# 重启
$ sh /home/bitbucket/atlassian/bitbucket/5.11.1/bin/stop-bitbucket.sh
$ sh /home/bitbucket/atlassian/bitbucket/5.11.1/bin/start-bitbucket.sh
```

## Bitbucket 设置

设置过程与 JIRA 和 Confluence 类似。
1. 浏览器中访问 http://yourip:7990 进入初始化页面；
2. 选择数据库 Mysql（utf8 字符集，utf8_bin 排序规则），输入相关参数，测试连接通过，点击下一步；
3. 输入license，设置管理员账户，就正式使用 Bitbucket 了。

# Bamboo
## Bamboo 安装

```bash
# 创建独立安装账号
$ useradd bamboo
$ passwd bamboo
$ su - bamboo

# 下载
$ wget https://product-downloads.atlassian.com/software/bamboo/downloads/atlassian-bamboo-6.6.0.tar.gz

# 解压
$ tar -zxvf atlassian-bamboo-6.6.0.tar.gz

# 设置 bamboo.home
$ vi /home/bamboo/atlassian-bamboo-6.6.0/atlassian-bamboo/WEB-INF/classes/bamboo-init.properties
bamboo.home=/home/bamboo/atlassian-bamboo-6.6.0

# Crowd 支持 Mysql 数据库，但是未集成 Mysql jdbc 驱动
$ cp mysql-connector-java-5.1.46.jar /home/crowd/atlassian-crowd-3.2.3/apache-tomcat/lib

# 启动 crowd
$ sh atlassian-crowd-3.2.3/start_crowd.sh
```

## Bamboo 设置

设置过程与 Bitbucket 等都类似。
1. 浏览器中访问 http://yourip:8085 进入初始化页面；
2. 输入license，选择数据库 Mysql（连接参数需要加：autoReconnect=true），输入相关参数，测试连接通过，点击下一步；
3. 设置管理员账户，就正式使用 Bamboo 了。

# Crowd 与 JIRA、Confluence、Bitbucket、Bamboo集成
## Crowd 与 JIRA 集成
1. 使用管理员用户登录 Crowd，新建 Group。
点击 “Groups” --> “Add group”，输入组名、描述、归属目录，点击创建。
![-w848](/assets/images/atlassian/15854047482429.jpg)
添加用户
![-w820](/assets/images/atlassian/15854049303244.jpg)

2. 新建Application
点击 “Applications” --> “Add application”，输入名称、描述、密码，点击创建。
![-w1134](/assets/images/atlassian/15854049786171.jpg)

3. 输入 JIRA 地址 和 其 ip 地址，点击下一步。
![-w1146](/assets/images/atlassian/15854050752397.jpg)
![-w1138](/assets/images/atlassian/15854051286684.jpg)

4. 选择允许访问 JIRA 的组，加入进去
![-w1125](/assets/images/atlassian/15854051548511.jpg)

5. 确认信息，点击 “Add application” ，添加应用完成。
![-w1144](/assets/images/atlassian/15854051854271.jpg)

6. 使用 JIRA 管理员，登录 JIRA。
点击用户管理 --> 用户目录 --> 添加目录，选择 “Atlassian 人群”，点击下一步。
![-w1157](/assets/images/atlassian/15854053372060.jpg)

7. 输入 Crowd 服务器的配置，点击测试，并保存。
![-w973](/assets/images/atlassian/15854055763799.jpg)
名称：Crowd Server
服务器的URL： Crowd的URL地址，比如：http://www.example.com:8095/crowd/
应用程序名称：与在 Crowd 里配置的 Application 名称一致
应用程序密码：与在 Crowd 里配置的 Application 密码一致

8. 系统默认每 1 小时从 Crowd 同步一次用户（系统管理员可修改），点击同步按钮也可手动同步。

9. 注销管理员用户，使用 Crowd 里的用户尝试登陆 JIRA，发现能够登录进去了。

## Crowd 与 Confluence、Bitbucket、Bamboo 集成
参考 Crowd 与 JIRA 集成。

# context-path 配置
默认安装好的 Atlassian 各应用是这样的：
- Crowd：http://ip:8095/crowd
- JIRA：http://ip:8080
- Confluence：http://ip:8090
- Bitbucket：http://ip:7990
- Bamboo：http://ip:8085

为了快捷访问，需要设置 Atlassian 各应用的访问地址为统一规范，如下：
- Crowd：http://www.xxx.com/crowd
- JIRA：http://www.xxx.com/jira
- Confluence：http://www.xxx.com/confluence
- Bitbucket：http://www.xxx.com/bitbucket
- Bamboo：http://www.xxx.com/bamboo

## 方案
1. 设置各应用的 context-path 为 "/"+"应用名"
2. 使用 nginx 代理，使 www.xxx.com/xxx 分别指向 http://ip:port/xxx

```
# jira 的 nginx 设置，其他类似
location ^~ /jira {

        proxy_pass http://x.x.x.x:8080/jira;

        sendfile off;

        proxy_set_header   Host             $host:$server_port;
        proxy_set_header   X-Real-IP        $remote_addr;
        proxy_set_header   X-Forwarded-For  $proxy_add_x_forwarded_for;
        proxy_max_temp_file_size 0;

        # This is the maximum upload size
        client_max_body_size       10m;
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

## 修改 context-path
### Crowd
默认支持，无需修改。

### JIRA

```bash
# server.xml，在 Context 标签中添加 path="/jira"
$ vi /home/jira/atlassian/jira/conf/server.xml
<Context path="/jira" docBase="${catalina.home}/atlassian-jira" reloadable="false" useHttpOnly="true">

# 重启生效
```

### Confluence

```bash
# server.xml，在 Context 标签中添加 path="/confluence"
$ vi /home/confluence/atlassian/confluence/conf/server.xml
<Context path="/confluence" docBase="../confluence" debug="0" reloadable="false" useHttpOnly="true">

# 重启生效
```

### Bitbucket

 ```bash
# bitbucket.properties
$ echo "server.context-path=/bitbucket" >> /home/bitbucket/atlassian/application-data/bitbucket/shared/bitbucket.properties

# 重启生效
```

### Bamboo

 ```bash
# server.xml，在 Context 标签中添加 path="/bamboo"
$ vi /home/bamboo/atlassian-bamboo-6.6.0/conf/server.xml
<Context path="/bamboo" docBase="${catalina.home}/atlassian-bamboo" reloadable="false" useHttpOnly="true">

# 重启生效
```

# 单点登录（SSO）配置
单点登录（SSO）需要有域名支持，也就是说，在配置sso之前，各应用系统已配置好相应的域名。

## Confluence 配置 SSO

```
# 编辑 seraph-config.xml
$ vi /home/confluence/atlassian/confluence/confluence/WEB-INF/classes/seraph-config.xml
# 注释掉
<!--<authenticator class="com.atlassian.confluence.user.ConfluenceGroupJoiningAuthenticator"/>-->

# 打开注释
<authenticator class="com.atlassian.confluence.user.ConfluenceCrowdSSOAuthenticator"/>

# 修改 crowd.properties
$ vi /home/confluence/atlassian/confluence/confluence/WEB-INF/classes/crowd.xml
application.name：配置 crowd 里该 Application 的名称
application.password：配置 crowd 里该 Application 的密码
application.login.url：配置 crowd 的地址

crowd.server.url：配置 crowd 的 services 地址
crowd.base.url：配置 crowd 的 baseurl 地址

session.tokenkey：与 crowd 管理页面的SSO cookie name保持一致

# 重启 Confluence 生效
```

**验证**

先登录crowd，然后在打开一个页面，输入confluence地址，发现不需要在手动输入账户密码，直接就登进去了

**注意**：必须使用带域名的地址，登录的用户必须是同步过的用户。

## JIRA 配置 SSO
参考 Confluence 配置 sso，基本一样，只是 JIRA 的安装目录里没有 crowd.properties 文件，可以从 Confluence 或者 Crowd 拷贝一份，然后修改配置的内容即可。

## Bitbucket 配置 SSO

```
# 编辑 bitbucket.properties
$ vi /home/bitbucket/atlassian/application-data/bitbucket/shared/bitbucket.properties
# 添加下面的属性
plugin.auth-crowd.sso.enabled=true

# 重启 Bitbucket 生效
```

## Bamboo 配置 SSO

```
# 编辑 seraph-config.xml
$ vi /home/bamboo/atlassian-bamboo-6.6.0/atlassian-bamboo/WEB-INF/classes/seraph-config.xml
# 注释掉
<!--<authenticator class="com.atlassian.bamboo.user.authentication.BambooAuthenticator"/>-->

# 打开注释
<authenticator class="com.atlassian.crowd.integration.seraph.v25.BambooAuthenticator"/>

# 修改 crowd.properties
$ vi /home/bamboo/atlassian-bamboo-6.6.0/xml-data/configuration/crowd.properties
application.name：配置 crowd 里该 Application 的名称
application.password：配置 crowd 里该 Application 的密码
application.login.url：配置 crowd 的地址

crowd.server.url：配置 crowd 的 services 地址
crowd.base.url：配置 crowd 的 baseurl 地址

session.tokenkey：与 crowd 管理页面的SSO cookie name保持一致

# 重启 Bamboo 生效
```

> 微信公众号：daodaotest
