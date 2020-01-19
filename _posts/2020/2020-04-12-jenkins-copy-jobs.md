---
layout: post
title: "Jenkins 批量创建任务的三种方法"
date: "2020-04-12 00:30"
category: Jenkins
tags: Jenkins jenkins-cli python-jenkins
author: jiangliheng
---
* content
{:toc}

最近，要搭建多套测试环境，需要把 Jenkins 中 dev 视图下的所有任务批量复制到 sit 等视图下。



# 说明

Jenkins 任务名称规则为：```[测试环境标识]-[工程名称]，如：dev-daodaotest，sit-daodaotest```。

视图中显示任务的正则表达式：```[测试环境标识]-.* ，如：dev-.*，sit-.*```。

# 第一种：目录下批量复制

Jenkins 的任务都是以 ```xml``` 文件方式存储的，所有可以通过复制 ```xml``` 的方式来批量创建。

```bash
# 进入 jobs 目录下
$ cd ~/.jenkins/jobs

# 创建批量复制 shell 脚本
$ vi copyViewJobs.sh
#!/bin/bash

# 视图名称
viewName=$1
# 新视图名称
newViewName=$2

# 循环复制任务
for jobName in `ls /home/jenkins/.jenkins/jobs/`
 do
    # 判断文件存在并且是目录
    if test -d $jobName
    then
        # 目录为 $viewName 开头，则进行复制
        if [[ $jobName == *$viewName* ]]; then
           # 截取工程名称
           name=`echo $jobName|awk 'BEGIN{FS="'$viewName'-"} {print $2}'`
           newJobName=$newViewName-$name
           echo $newJobName
           # 复制 config.xml
           mkdir $newJobName && cp $jobName/config.xml $newJobName/
        fi
     fi
 done

 # 执行批量复制脚本，dev 视图下的任务负责到 sit 视图下
 $ sh copyViewJobs.sh dev sit
```

**注意**：复制完成后，Jenkins 需要重新加载配置才可以生效。操作菜单路径：```Manage Jenkins``` --》 ```Reload Configuration from Disk```。

# 第二种：jenkins-cli

实现步骤与第一种类似，大家可以根据自己擅长的脚本语言来实现即可。下面简单介绍下关键命令。

> ```jenkins-cli``` 使用方法见：http://localhost:8080/cli

```bash
# 下载 jenkins-cli.jar
$ wget http://localhost:8080/jnlpJars/jenkins-cli.jar

# 获取视图下的所有任务
$  java -jar jenkins-cli.jar -s http://localhost:8080/ -auth daodaotest:daodaotest list-jobs dev

# 复制任务
$ java -jar jenkins-cli.jar -s http://localhost:8080/ -auth daodaotest:daodaotest copy-job dev-daodaotest sit-daodaotest
```

# 第三种：REST API

同第二种，仅介绍关键命令。这里以 ```python-jenkins``` api 为例。

> ```python-jenkins``` 官网地址：https://opendev.org/jjb/python-jenkins

## 安装 Python Jenkins

```bash
# 安装 pip
$ sudo yum install epel-release && sudo yum install python-pip

# 安装 python-jenkins
$ pip install python-jenkins
```

## 获取视图下任务名称

```python
import jenkins

server = jenkins.Jenkins('http://localhost:8080', username='daodaotest', password='daodaotest')

# 查询 dev 视图下的所有任务
jobs = server.get_jobs(folder_depth=0, view_name='dev')

# 循环打印任务名称
for job in jobs:
    print(job['fullname'])
```

## 复制任务

```python
import jenkins

server = jenkins.Jenkins('http://localhost:8080', username='daodaotest', password='daodaotest')

# 任务是否存在，True 为存在，Fasle 为不存在
print(server.job_exists('dev-daodaotest'))

# 复制任务
server.copy_job('dev-daodaotest','sit-daodaotest')

# 打印任务信息
jobinfo = server.get_job_info('sit-daodaotest')
print(jobinfo)
```

### 请求报错 "Error 403 No valid crumb was included in the request"

**错误原因**： jenkins 在 http 请求头部中放置了一个名为 .crumb 的 token。在使用了反向代理，并且在 jenkins 设置中勾选了“防止跨站点请求伪造（Prevent Cross Site Request Forgery exploits）”之后此 token 会被转发服务器 nginx 认为是不合法头部而去掉，导致跳转失败。

**解决办法**：在 Jenkins 的安全设置中取消“防止跨站点请求伪造（Prevent Cross Site Request Forgery exploits）”。

> 微信公众号：daodaotest
