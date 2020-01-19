---
layout: post
title: "Python3 实现批量创建 Crowd 用户并分配组"
date: "2020-05-17 21:00"
category: Python
tags: Python Crowd
author: jiangliheng
---
* content
{:toc}



# 背景

迁移 Crowd 完成后（之前采用 LDAP 方式，新迁移 Crowd 不采用），需要批量创建公司所有员工的用户以及分配组，手工创建以及之前 Postman 的方式还是比较低效。

Python 在 N 多年前入门，写了几个爬虫脚本后，再也没用过，借这个机会顺便再熟悉下 Python 脚本。

归根结底的原因就是：本人很懒~

# Crowd Api

> https://docs.atlassian.com/atlassian-crowd/3.2.0/REST/


如下示例是基于 Crowd 3.2.0 版本的 Api，不同版本间的 Api 稍有差异。

```bash
# 添加用户
$ curl -u "application-name:password" -X POST -H "Content-Type: application/json" -H "Accept: application/json" -d "{\"name\" : \"test.user\", \"display-name\" : \"Test User\", \"active\" : true, \"first-name\" : \"Test\", \"email\" : \"test.user@ourdomain.com\", \"last-name\" : \"User\", \"password\" : {\"value\" : \"mypassword\"} }" http://localhost:8095/crowd/rest/usermanagement/1/user

# 用户添加到组
$ curl -u "application-name:password" -X POST -H "Content-Type: application/json" -d "{\"name\" : \"all-users\"}" http://localhost:8095/crowd/rest/usermanagement/1/user/group/direct\?username\=daodaotest
```

**注意**：此处```-u```的参数为 Crowd 中应用（Application）的用户名和密码，Crowd 的管理员是不能添加用户。

# Python 实现脚本

实现添加 Crowd 用户，用户添加到指定组，读取 csv 文件批量添加用户和设定的多个组。



**crowdUsers.csv** 用户数据 csv 文件

```csv
name,displayName,email
daodaotest1,daodaotest1,daodaotest1@daodaotest.com
daodaotest2,daodaotest2,daodaotest2@daodaotest.com
daodaotest3,daodaotest3,daodaotest3@daodaotest.com
......
```

**addCrowdUsers.py** 批量添加 Crowd 用户和用户组脚本

```python
#!/usr/bin/python
# -*- coding: UTF-8 -*-
#
# Filename         addCrowdUsers.py
# Revision         0.0.1
# Date             2020/5/14
# Author           jiangliheng
# Email            jiang_liheng@163.com
# Website          https://jiangliheng.github.io/
# Description      批量添加 Crowd 用户和用户组

import requests
from requests.auth import HTTPBasicAuth
import csv
from itertools import islice

# 请求 headers
headers = {
    'Accept': 'application/json',
    'Content-type': 'application/json',
}

# crowd 访问基础路径
base_url='http://localhost:8095'

# 添加用户的默认用户组和密码
auth_username='application-name'
auth_password='password'

# 用户默认密码
password='daodaotest'

def addUser(name,displayName,email):
    """
    添加单用户

    :param name: 登录用户，建议拼音全称，如：jiangliheng
    :param displayName: 显示名称，建议中文全称，如：蒋李恒
    :param email: 邮箱地址
    :return: status_code 状态码,text 响应报文信息
    """

    # 请求 json 数据
    data = '{ \
        "name" :"' + name + '", \
        "email" : "' + email + '", \
        "active" : true, \
        "first-name" : "' + displayName + '", \
        "last-name" : "' + displayName + '", \
        "display-name" : "'+ displayName + '", \
        "password" : { \
            "value" : "' + password + '" \
        } \
    }'

    # 发起请求
    # 解决中文乱码问题 data.encode("utf-8").decode("latin1")
    response = requests.post(
        base_url + '/crowd/rest/usermanagement/1/user',
        headers=headers,
        auth=HTTPBasicAuth(auth_username,auth_password),
        data=data.encode("utf-8").decode("latin1")
    )

    # 状态码
    status_code=response.status_code
    # 响应报文信息
    text=response.text

    # 状态判断
    if str(status_code).startswith("2"):
        print("%s 用户添加成功，状态码：%s ，响应报文信息：%s" % (name,status_code,text))
    else:
        print("%s 用户添加失败，状态码：%s ，响应报文信息：%s" % (name,status_code,text))

    # 返回 状态码，响应报文信息
    return status_code,text

def addGroup(username,groupname):
    """
    用户添加到组

    :param username: 登录用户，建议拼音全称，如：jiangliheng
    :param groups: 用户组，用逗号隔开，如：bitbucket-users,bamboo-users
    :return: status_code 状态码,text 响应报文信息
    """

    # 请求 json 数据
    data = '{ \
        "name" :"' + groupname + '" \
    }'

    # 发起请求
    response = requests.post(
        base_url + '/crowd/rest/usermanagement/1/user/group/direct?username='+username,
        headers=headers,
        auth=HTTPBasicAuth(auth_username,auth_password),
        data=data
    )

    # 状态码
    status_code=response.status_code
    # 响应报文信息
    text=response.text

    # 状态判断
    if str(status_code).startswith("2"):
        print("%s 用户添加组 %s 成功，状态码：%s ，响应报文信息：%s" % (username,groupname,status_code,text))
    else:
        print("%s 用户添加组 %s 失败，状态码：%s ，响应报文信息：%s" % (username,groupname,status_code,text))

    # 返回 状态码，响应报文信息
    return status_code,text

def addUserByCsv(csvfile):
    """
    通过 CSV 文件批量添加用户，并加到组

    :param filename: Crowd 用户 csv 文件
    """

    # 批量读取 csv 的用户
    with open(csvfile, 'r', encoding='utf-8') as f:
        fieldnames = ("name", "displayName", "email")
        reader = csv.DictReader(f, fieldnames)

        for row in islice(reader, 1, None):
            print("批量添加用户 %s" % (row["name"]))
            # 添加用户
            addUser(row["name"],row["displayName"],row["email"])
            # 添加多个组
            addGroup(row["name"],"all-users")
            addGroup(row["name"],"bitbucket-users")
            addGroup(row["name"],"confluence-users")
            addGroup(row["name"],"jira-software-users")
            addGroup(row["name"],"sonar-users")

        f.close()

def main():
    # 通过 CSV 文件批量添加用户，并加到组
    addUserByCsv("crowdUsers.csv")

    # 添加单用户
    # addUser("daodaotest","叨叨软件测试","daodaotest@daodaotest.com")

    # 添加用户到组
    # addGroup("daodaotest","all-users")

if __name__ == "__main__":
    main()
```

> 微信公众号：daodaotest
