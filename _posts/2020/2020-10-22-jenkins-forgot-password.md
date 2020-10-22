---
layout: post
title: "jenkins 忘记密码或认证配置出错后解决办法"
date: "2020-10-22 21:00"
category: Jenkins
tags: Jenkins
author: jiangliheng
---
* content
{:toc}



# 背景

我们测试环境的 Jenkins 是通过 Crowd 进行统一登录认证，授权策略采用“项目矩阵授权策略”，运维同事在配置```Role-Based Strategy```时出错，导致所有用户登录后都没有权限了。

# 解决办法

Jenkins 的所有信息都是存储在 xml 文件中，目录为：$HOME/.jenkins，其中配置文件信息保存在：$HOME/.jenkins/config.xml，用户信息保存在：$HOME/.jenkins/users/admin_1669049878327248561/config.xml。

## 去掉安全认证（推荐）
```bash
# 终极方案，直接去掉安全认证
# 编辑 $HOME/.jenkins/config.xml，将 useSecurity 选项内容 true 改为 false
$ cat $HOME/.jenkins/config2.xml | grep -n useSecurity
12:<useSecurity>true</useSecurity>
$ sed -i "s/.*useSecurity.*/\<useSecurity\>false\<\/useSecurity\>/g" $HOME/.jenkins/config.xml

# 重启 jenkins，重新设置安全认证或者修改用户密码即可
```

## 重置 admin 用户密码

```bash
# 编辑 admin 用户的 config.xml 文件，替换 passwordHash 行为如下，密码为： 123456
$ vi $HOME/.jenkins/users/admin_1669049878327248561/config.xml
<passwordHash>#jbcrypt:$2a$10$MiIVR0rr/UhQBqT.bBq0QehTiQVqgNpUGyWW2nJObaVAM/2xSQdSq</passwordHash>

# 重启 jenkins，用新密码登录即可
```

> 微信公众号：daodaotest
