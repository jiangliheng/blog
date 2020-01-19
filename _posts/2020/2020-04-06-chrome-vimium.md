---
layout: post
title: "使用 Chrome 插件 Vimium 打造黑客浏览器"
date: "2020-04-06 15:30"
category: Chrome
tags: Chrome Vimium
author: jiangliheng
---
* content
{:toc}

之前一直用 cVim，与 Vimium 功能类似，但是之后不在更新了，故转战到 Vimium。



# 简介

> 官网：http://vimium.github.io/

Vimium 是 Google Chrome 浏览器的扩展程序，它提供了 Vim 编辑器中用于导航和控制的键盘快捷键。


## 特点
- 全键盘操作浏览器，直接放弃鼠标；
- 使用醒目的显示方法来浏览链接；
- 自定义的键盘快捷键；
- 具有页面内的帮助快捷方式，页面内输入```?```即可快捷键帮助。

# 安装

Chrome 应用商店搜索 Vimium 下载安装即可。

# 查看帮助

在页面内输入 ```?``` 就可以查看帮助，再次输入回到原页面。

**注意**：与 Vim 一样，命令需要区分大小写。

![-w839](/assets/images/chrome/vimium/15861451830605.jpg)

**查看更多高级命令**

![-w837](/assets/images/chrome/vimium/15861454880680.jpg)

# 常用操作

**注意**：与 Vim 一样，```Esc``` 为退出命令模式。

## 快速打开

快捷键|说明
:----|:----
```o``` | 		当前页签打开 网址, 书签 或 历史页面
```O``` |		新页签打开 网址, 书签 或 历史页面
```b``` |		当前页签打开 书签
```B``` |		新页签打开书签
```T``` |		搜索当前打开标签页

## 标签操作

快捷键|说明
:----|:----
```t``` | 创建标签页
```J```,```gT``` | 切换到左边标签页
```K```,```gt``` | 切换到左边标签页
```^``` | 	切换到上一个标签页，多次点击互相切换
```g0``` |	切换到第一个标签页
```g$``` |	切换到最后一个标签页
```yt``` |	复制当前标签页
```x``` | 关闭当前标签页
```X``` | 恢复关闭的标签页

## 页面操作

快捷键|说明
:----|:----
```f``` |	在当前标签打开链接
```F``` |	在新页签打开链接
```j```	|	向下移动
```k``` |	向上移动
```h``` |	向左移动
```l``` |	向右移动
```d``` |	向下翻半页
```u``` |	向上翻半页
```gg``` |	移动到页面顶部
```G``` |	移动到页面底部
```L``` | 历史浏览前进
```H``` | 历史浏览后退
```r``` |	刷新页面
```yy``` |	复制浏览器地址栏的网址
```p``` |	读取剪切板内容，粘贴到地址栏搜索，并在当前页签打开
```P``` |	读取剪切板内容，粘贴到地址栏搜索，并在新页签打开
```i``` |	切换到输入模式
```v``` |	切换到视图模式
```gi``` |	光标定位到第一个输入框

## 页面搜索

快捷键|说明
:----|:----
```/``` |		搜索模式
```n``` |		循环向下搜索关键字
```N``` |		循环向上搜索关键字

# 自定义配置

以下为我的自定义设置，大家可以参考下。

## 自定义快捷键

我个人操作习惯为：链接新页签打开。

```
# 修改快捷键 f 为新页签后台打开
unmap f
map f LinkHints.activateModeToOpenInNewTab
```

![-w894](/assets/images/chrome/vimium/15861563447627.jpg)

**Show available commands** 为所有的快捷键和对应功能代码。

![-w839](/assets/images/chrome/vimium/15861565095090.jpg)

## 自定义直达网站

自定义快速直达网站。

```
# 一键直达网站
# 今日头条
map zt createTab https://www.toutiao.com/c/user/6973555764/#mid=1660416476789771
# 简书
map zs createTab https://www.jianshu.com/u/aa29f3eacc01
# csdn
map zc createTab https://blog.csdn.net/jlh21
# 博客园
map zb createTab https://www.cnblogs.com/daodaotest/
# 个人博客
map zj createTab https://jiangliheng.github.io/
```

![-w855](/assets/images/chrome/vimium/15861571974543.jpg)

## 自定义搜索引擎

```
b: https://www.baidu.com/s?wd=%s baidu
g: https://www.google.com/search?q=%s google
gh: https://github.com/search?q={query} github
s: https://www.stackoverflow.com/search?q={query} stackoverflow
m: http://www.mvnrepository.com/search?q={query} mvnrepository
w: https://www.wikipedia.org/w/index.php?title=Special:Search&search=%s Wikipedia
z: https://www.zhihu.com/search?type=content&q=%s zhihu
```

![-w888](/assets/images/chrome/vimium/15861553101422.jpg)

## 修改默认搜索引擎

```
https://www.baidu.com/s?wd=
```

![-w839](/assets/images/chrome/vimium/15861553380200.jpg)

> 微信公众号：daodaotest
