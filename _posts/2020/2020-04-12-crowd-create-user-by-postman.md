---
layout: post
title: "Crowd 批量添加用户（Postman 数据驱动）"
date: "2020-04-12 11:00"
category: Atlassian
tags: Crowd Postman
author: jiangliheng
---
* content
{:toc}

最近公司大量新员工入职，需要批量创建 Crowd 用户、设置密码、分配应用组等机械性重复工作（主要还是懒~），故把这个加餐任务分配给刚来的测试同学去研究。



# 背景
最近公司大量新员工入职，需要批量创建 Crowd 用户、设置密码、分配应用组等机械性重复工作（主要还是懒~），故把这个加餐任务分配给刚来的测试同学去研究。

一是：让他了解下 Postman 的数据驱动，RESTful api 的相关基础知识；二是：考察下新员工独立完成任务的能力；三是我比较懒~。

# Crowd api 添加用户

> https://community.atlassian.com/t5/Answers-Developer-Questions/How-to-add-user-via-Crowd-REST-API/qaq-p/482434

```bash
curl -u "test:password" -X POST -H "Content-Type: application/json" -H "Accept: application/json" -d "{\"name\" : \"test.user\", \"display-name\" : \"Test User\", \"active\" : true, \"first-name\" : \"Test\", \"email\" : \"test.user@ourdomain.com\", \"last-name\" : \"User\", \"password\" : {\"value\" : \"mypassword\"} }" http://localhost:8095/crowd/rest/usermanagement/1/user
```

**注意**：此处```-u```的参数为 Crowd 中应用（Application）的用户名和密码，Crowd 的管理员是不能添加用户。

# Postman 数据驱动

1. ```curl``` 命令方式导入到 Postman，测试添加单个用户
2. 数据驱动批量添加用户

## curl 命令方式导入 Postman

Postman 支持使用 ```curl``` 命令方式导入。

打开左上角“Import”，选择 “Paste Raw Text”方式，输入```curl``` 命令即可。
![-w1402](/assets/images/atlassian/crowd/15866555711563.jpg)

权限认证方式：Basic Auth。
![-w1402](/assets/images/atlassian/crowd/15866556180906.jpg)

 导入的 Headers 参数。
![-w1402](/assets/images/atlassian/crowd/15866556333782.jpg)

导入的 Body 内容。
![-w1402](/assets/images/atlassian/crowd/15866556477060.jpg)

## Postman 数据驱动批量添加用户

### 创建 Collections，添加 api

设置全局变量 password。
![-w1402](/assets/images/atlassian/crowd/15866585587579.jpg)

body 字段参数化。
```json
{
    "name": "{{name}}",
    "display-name": "{{display-name}}",
    "active": true,
    "first-name": "{{display-name}}",
    "email": "{{email}}",
    "last-name": "{{display-name}}",
    "password": {
        "value": "{{password}}"
    }
}
```

### 准备 csv 数据文件

```
# crowdUsers.csv
name,display-name,email
daodaotest1,叨叨软件测试1,daodaotest1@test.com
daodaotest2,叨叨软件测试2,daodaotest2@test.com
```

### 执行

选择 csv 数据文件。
![-w1280](/assets/images/atlassian/crowd/15866588319084.jpg)

预览参数。
![-w1280](/assets/images/atlassian/crowd/15866594824841.jpg)


查看执行结果。
![-w1280](/assets/images/atlassian/crowd/15866589116336.jpg)


crowd 添加成功。
![-w828](/assets/images/atlassian/crowd/15866589384887.jpg)

![-w583](/assets/images/atlassian/crowd/15866589613406.jpg)

> 微信公众号：daodaotest
