---
layout: post
title: "Jenkins 基于 Crowd 和 Role-based 插件的角色权限管理"
date: "2021-04-09 19:40"
category: Jenkins
tags: Jenkins Crowd Role-based
author: jiangliheng
---
* content
{:toc}



# 背景

测试环境的 Jenkins 是开发和测试混用的，未做细粒度的权限控制，开发总是构建测试的任务（不提前打招呼），导致测试任务中断，故需要隔离开发和测试用户权限。

PS：我司是使用 Crowd 进行用户的权限管理，来实现所有办公软件的统一登录。

# 配置

**配置约定**
1. Jenkins 任务命名规范：环境标识-项目组或业务标识-具体项目名称，eg：dev-pay-payManager;
2. Jenkins 视图正则表达式筛选规范：环境标识-.*，eg: dev-.\*。

**用户组及权限**
1. ```development```：开发人员组，查看开发环境相关的任务（比如：dev、dev2、dev3）；
2. ```test```：测试人员组，查看测试环境相关的任务（比如：sit、open、per）；
3. ```ops```：运维人员组，维护 Jenkins，分配 Admin 权限。

**实施步骤**
1. 首先，在 Crowd 配置用户组：```development```（开发人员组）、```test```（测试人员组）、```ops```（运维组），并与 Jenkins 应用关联；
2. 其次，Jenkins 上使用 Crowd 安全域，即用户和用户组通过 Crowd 获取及认证；
3. 最后，Jenkins 上配置授权策略为```Role-Based Strategy```，并配置角色、分配角色。

## Crowd 配置用户及用户组

Crowd 配置用户及用户组配置如下：
![](/assets/images/jenkins/16179608693085.jpg)

## Jenkins 配置

### 插件安装

首先，插件管理中安装```Crowd 2 Integration```、```Role-based Authorization Strategy```插件。

> ```Role-based Authorization Strategy```：https://plugins.jenkins.io/role-strategy
> ```Crowd 2 Integration```：https://plugins.jenkins.io/crowd2

### Crowd 配置

具体操作细节可参考之前写的 [Atlassian 系列软件安装（Crowd+JIRA+Confluence+Bitbucket+Bamboo）]()，[Nexus3 集成 crowd 插件]()

![](/assets/images/jenkins/16179617821394.jpg)

**说明**
* Crowd URL：Crowd 的地址；
* Application Name：Crowd 里面配置的应用名称；
* Application Password：Crowd 里面配置应用的密码；
* Restrict groups：配置需要认证的用户组；
* cookie.tokenkey：Crowd 的 token 变量。

### Role-Based Strategy 配置

菜单路径：```Manage Jenkins```--```Security```--```Manage and Assign Roles```

```Manage Roles（定义角色）```配置
![](/assets/images/jenkins/16179624030055.jpg)

**说明**
* 全局角色：```admin```--管理员权限；```read```--仅配置只读权限；
* 项目角色：根据环境标识或者其他属性划分的系列任务组，一般与视图保持一致，具体权限根据具体需求设置即可。

```Assign Roles（分配角色）```配置
![](/assets/images/jenkins/16179624461466.jpg)


**说明**
* 全局角色分配：运维组设置为管理员角色，其他组设置为只读角色，未认证的用户无任何权限；
* 项目角色分配：```development```配置开发环境的权限；```test```配置测试环境的权限；```ops```配置运维自建的权限（此处为了扩展，可以不分配，运维是 Admin，有所有权限）。

> 微信公众号：daodaotest
