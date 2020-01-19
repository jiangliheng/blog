---
layout: post
title: "PLSQL Developer 中文乱码踩坑记"
date: "2020-04-19 18:00"
category: Linux
tags: Linux mail mailx
author: jiangliheng
---
* content
{:toc}





# 环境

操作系统版本： Windows 7
PL/SQL 版本： 12.0.1.1814

# 原因

由于 Oracle 服务器端和客户端字符集编码不一致引起的。

# 注意点

写在最前面，减少踩坑！！！

网上教程大多未强调这些注意点，像我这样的 Oracle 小白就完美踩坑而过。

- 设置完环境变量```NLS_LANG```后，我个人重启 ```PL/SQl``` 多次不生效，重启操作系统才生效。
- 设置客户端和服务器端的字符集后，需要再次```UPDATE```后，此时```SELECT```才不是乱码。
- 执行完 ```SQL``` 语句，记得 ```commit```，否则其他会话无法获取最新数据。

# 解决方法
## 服务端

检查 Oracle 服务器端字符编码是否一致。

```sql
-- 检查字符集是否一致
select userenv('language') from dual;
-- AMERICAN_AMERICA.AL32UTF8

select * from v$nls_parameters a where a.PARAMETER = 'NLS_CHARACTERSET';
-- AL32UTF8
```

## 客户端

### 设置客户端字符集
在系统环境变量中，新增变量 ```NLS_LANG```，设置字符集为：```AMERICAN_AMERICA.AL32UTF8```（服务器端的字符集）。

我个人重启 PL/SQL 不生效，重启系统才生效。

**验证是否生效**

打开 ```PL/SQL``` 工具的：帮助--支持信息--信息 选项卡里进行检查，在“Character Sets”下面，有一项是：“NLS_LANG”， 检查是否与环境变量设置的```NLS_LANG```一致，一致即生效。

### 设置字体字符集

打开 ```PL/SQL``` 工具的：配置--首选项--用户界面--字体--主字体，设置字体字符集为“西欧语言”，默认为“中文 GB2312”。

> 微信公众号：daodaotest
