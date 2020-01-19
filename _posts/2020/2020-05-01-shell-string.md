---
layout: post
title: "Shell 字符串操作符实例"
date: "2020-05-01 22:00"
category: Shell
tags: Linux Shell
author: jiangliheng
---
* content
{:toc}




# 字符串操作符

表达式|含义
:----- | :-----
```${parameter-word}``` |	```parameter``` 变量未声明，取默认值 ```word```
```${parameter:-word}``` |	```parameter``` 变量未声明或值为空时，取默认值 ```word```
```${parameter=word}``` | ```parameter``` 变量未声明，则取默认值 ```word```
```${parameter:=word}``` | ```parameter``` 变量未声明或值为空时，取默认值 ```word```
```${parameter+word}``` | ```parameter``` 变量未声明, 取值为空，否则取值为 ```word```
```${parameter:+word}``` | ```parameter``` 变量声明, 取值为 ```word```，否则取值为空
```${parameter?word}``` | 	```parameter``` 变量未声明, 标准错误输出 ```word``` 且退出 shell
```${parameter:?word}``` | ```parameter``` 取值为空, 标准错误输出 ```word``` 且退出 shell
```${!prefix*}``` | 	匹配所有以 ```prefix``` 开头且声明的变量
```${!prefix@}``` | 	匹配所有以 ```prefix``` 开头且声明的变量
```${#parameter}``` | 	```$parameter``` 的长度
```${parameter:offset}``` | 从左边指定位置 ```offset``` 开始，截取后面所有字符串
```${parameter:offset:length}``` | 从左边指定位置```offset```开始，截取指定长度```length```字符串
```${parameter#pattern}``` | 从右边开始，删除最短匹配```pattern```的子字符串
```${parameter##pattern}``` | 从右边开始，删除最长匹配```pattern```的子字符串
```${parameter%pattern}``` | 从左边开始，删除最短匹配```pattern```的子字符串
```${parameter%%pattern}``` | 从左边开始，删除最长匹配```pattern```的子字符串
```${parameter/pattern/string}``` | 从右边开始，替换第一次出现匹配项  ```pattern``` 为 ```string```
```${parameter//pattern/string}``` | 替换所有匹配项 ```pattern``` 为 ```string```
```${parameter/#pattern/string}``` | 替换开头匹配```pattern```字符串为 ```string```
```${parameter/%pattern/string}``` | 替换结尾匹配```pattern```字符串为 ```string```
```${parameter^pattern}``` | 开头第一个小写字母转换为大写
```${parameter^^pattern}``` | 所有小写字母转换为大写
```${parameter,pattern}``` | 开头第一个大写字母转换为小写
```${parameter,,pattern}``` | 所有大写字母转换为小写

# 实例脚本

可以使用 ```sh -v testString.sh``` 命令来执行如下脚本，查看原始命令及输出内容，为了方便区分命令和内容，其中输出内容以深蓝色显示。

```bash
# 实例脚本
$ cat testString.sh
#!/bin/bash
# 字符串操作符实例

# 判断操作系统，解决 mac下 echo 不支持“-e”参数问题
if [[ "$(uname)" != "Darwin" ]];then
    ee="-e"
fi

# var 变量未声明
echo ${ee} "\033[36mvar 变量未声明，输出为空： ${var}\033[0m"

# 变量未声明，取默认值
echo ${ee} "\033[36mvar 变量未声明，则取默认值：${var-daodaotest}\033[0m"
echo ${ee} "\033[36mvar 变量未声明，则取默认值：${var=daodaotest}\033[0m"

# 变量未声明或取值为空时，取默认值
# 变量未声明，取默认值
echo ${ee} "\033[36mvar2 变量未声明，则取默认值：${var2:-daodaotest2}\033[0m"
echo ${ee} "\033[36mvar2 变量未声明，则取默认值：${var2:=daodaotest2}\033[0m"

# 取值为空时
var3=
echo ${ee} "\033[36mvar3 变量声明，但值为空时，取默认值：${var3:-daodaotest3}\033[0m"
echo ${ee} "\033[36mvar3 变量声明，但值为空时，取默认值：${var3:=daodaotest3}\033[0m"

# 变量未声明，值为空；声明了为设置值
echo ${ee} "\033[36mvar4 变量未声明，值为空：${var4+daodaotest4}\033[0m"
var5=daodaotest5
echo ${ee} "\033[36mvar5 变量声明，取设置值：${var5+daodaotest}\033[0m"

# 变量未声明或取值为空时，打印设置信息且程序退出
var6=daodaotest6
echo ${ee} "\033[36mvar6 变量声明且取值，不打印设置信息：${var6?变量未声明或取值为空}\033[0m"

# 变量未声明或取值为空时，打印设置信息且程序退出
# 为了脚本继续运行注释掉
#echo ${ee} "\033[36mvar7 变量未声明，打印设置信息：${var7:?变量未声明}\033[0m"
#var8=
#echo ${ee} "\033[36mvar8 取值为空，打印设置信息：${var7:?变量取值为空}\033[0m"

# 通过前缀字符匹配声明过的变量名
x1=1
x2=2
x3=3
echo ${ee} "\033[36m通过前缀字符匹配声明过的变量名：${!x*}\033[0m"
echo ${ee} "\033[36m通过前缀字符匹配声明过的变量名：${!x@}\033[0m"

url="https://www.toutiao.com/i6820392125645980174"
## 字符串长度
echo ${ee} "\033[36m字符串内容：${url}\033[0m"
echo ${ee} "\033[36m字符串长度：${#url}\033[0m"

## 字符串截取
# 字符串位置截取
echo ${ee} "\033[36m从左边指定位置开始，截取后面所有字符串：${url:8}\033[0m"
echo ${ee} "\033[36m从左边指定位置开始，截取指定长度字符串：${url:8:15}\033[0m"
echo ${ee} "\033[36m从右边指定位置长度开始，截取后面所有字符串（注意“:”右边有空格）：${url: -20}\033[0m"
echo ${ee} "\033[36m从右边指定位置长度开始，截取后面所有字符串（同上，推荐）：${url:0-20}\033[0m"
echo ${ee} "\033[36m从右边指定位置长度开始，截取后面所有字符串（同上，推荐）：${url:(-20)}\033[0m"
echo ${ee} "\033[36m从右边指定位置开始，截取指定长度字符串（注意“:”右边有空格）：${url:0-36:15}\033[0m"
echo ${ee} "\033[36m从右边指定位置开始，截取指定长度字符串（同上，推荐）：${url:0-36:15}\033[0m"
echo ${ee} "\033[36m从右边指定位置开始，截取指定长度字符串（同上，推荐）：${url:(-36):15}\033[0m"

## 截取不匹配的字符串，即删除匹配的字符串
echo ${ee} "\033[36m从右边开始，删除最短匹配字符串：${url#*/}\033[0m"
echo ${ee} "\033[36m从右边开始，删除最长匹配字符串：${url##*/}\033[0m"

echo ${ee} "\033[36m从左边开始，删除最短匹配字符串：${url%/*}\033[0m"
echo ${ee} "\033[36m从左边开始，删除最长匹配字符串：${url%%/*}\033[0m"

## 匹配项替换
echo ${ee} "\033[36m从右边开始，替换第一次出现匹配项：${url/\//#}\033[0m"
echo ${ee} "\033[36m替换所有匹配项：${url//\//#}\033[0m"

echo ${ee} "\033[36m替换开头匹配字符串：${url/#https/http}\033[0m"
echo ${ee} "\033[36m替换结尾匹配字符串：${url/%i6820392125645980174/daodaotest}\033[0m"

param=daodaotest
# macOS zsh 不支持
echo ${ee} "\033[36m开头第一个小写字母转换为大写：${param^}\033[0m"
echo ${ee} "\033[36m所有小写字母转换为大写：${param^^}\033[0m"

param=DAODAOTEST
# macOS zsh 不支持
echo ${ee} "\033[36m开头第一个大写字母转换为小写：${param,}\033[0m"
echo ${ee} "\033[36m所有大写字母转换为小写：${param,,}\033[0m"
```

> 微信公众号：daodaotest
