---
layout: post
title: "Shell 变量引用实例"
date: "2020-04-12 08:30"
category: Shell
tags: Shell
author: jiangliheng
---
* content
{:toc}

初学 Shell 编程时，对变量各种引用使用不太熟悉，走了很多弯路，本文记录变量引用的一些用法，希望对大家有所帮助。



# 引用

**引用**指将字符串用引用符号引起来，以防止特殊字符被 ```shell``` 脚本解释为其他意义。 ```shell``` 中定义了 4 种引用符号。

引用符 | 名称 | 说明
:----- | :----- | :-----
'' | 单引号 | 称全引用或弱引用，引用所有的字符
"" | 双引号 | 称部分引用或强引用，引用除美元符号（$）、反引号（‘）和反斜线（\）之外的所有字符。
`` | 反引号 | shell 把反引符中的内容解释为系统命令
\ | 反斜杠 | 转义符，屏蔽下一个字符的特殊意义

# 实例脚本

可以使用 ```sh -v testVar.sh``` 命令来执行如下脚本，查看原始命令及输出内容。

```bash
# 实例脚本
$ cat testVar.sh
#!/bin/bash
# 变量引用示例

var=daodaotest

## 双引号
# 正常赋值输出
echo "Hello $var"
# 正常赋值输出，${} 方式
echo "Hello ${var}"
# 不会有任何输出，shell 会去引用变量 var2 的值
echo "$var2"
# 正常输出，推荐使用 ${} 方式来引用变量
echo "${var}2"


## 反引号
# 把 pwd 解释为系统命令，即：先执行 pwd，再 echo 打印
echo `pwd`
# 相等于 `pwd`
echo $(pwd)
# 同理
echo `echo $var`

## 单引号
echo '单引号引用时，输出字面内容：$var'
echo '单引号引用时，输出字面内容：${var}'

## 转义符
echo '使用单引号引用，不需要使用转义符号: $、`、"、\'
echo "使用双引号引用，需要使用转义符号: \$、\`、\"、\\"
echo "\$var"

## 反引号嵌套单引号和双引号
jobName=dev-daodaotest
viewName=dev
# 此处的变量 ${viewName} 其实是放在了两对单引号中间，起到拼接的作用
name=`echo ${jobName}|awk 'BEGIN{FS="'${viewName}'-"} {print $2}'`
echo ${name}
```

> 微信公众号：daodaotest
