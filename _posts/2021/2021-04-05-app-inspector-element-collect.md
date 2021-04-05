---
layout: post
title: "APP 元素定位总结（未完待补充）"
date: "2021-04-05 18:30"
category: APP自动化
tags: uiautomatorviewer Appium weditor
author: jiangliheng
---
* content
{:toc}



# 背景

个人记录，团队分享使用，好记性不如烂笔头~

# 定位工具

推荐使用顺序：```weditor``` > ```uiautomatorviewer``` > ```Appium inspector```

**三种定位工具**
- ```Python uiautomator2``` 中的 ```weditor```
- Android SDK 自带的 ```uiautomatorviewer```
- ```Appium inspector```

**三种工具异同点**
- ```Appium inspector``` 需要配置启动参数，相对较复杂些；```uiautomatorviewer```最方便；
- ```Appium inspector``` 不能直接定位手机打开的应用，需要重新启动（比如：钉钉每次都要重新登录）；```uiautomatorviewer```和```weditor```不需要，可直接定位；
- ```uiautomatorviewer```原生不支持 xPath 定位，可二次开发支持；```Appium inspector```和```weditor```支持；


**多种定位工具交替使用时遇到的问题**
1. ```uiautomatorviewer``` 定位时，手机上需要关闭 Appium 的```io.appium.uiautomator2.server```服务以及 ATX 的 ```UIAutomator```服务；
2. Appium 与 Python uiautomator2 同时使用时需要注意，```Appium inspector``` 与 ATX 的 ```UIAutomator```服务也会存在冲突；
3. ```Appium inspector``` 启动失败报错：An unknown server-side error occurred while processing the command. Original error: Failed to launch Appium Settings app: Condition unmet after 5001 ms. Timing out.），一种原因是```Appium Settings```在手机上被卸载了，未卸载干净造成的，解决方法：```adb uninstall io.appium.settings```重新连接即可（```io.appium.uiautomator2.server```同理也可能出现）。

# 定位技巧

定位方式推荐顺序：
1. 优先使用```resourceId```定位方式；
2. 其次采用```text```、```description```、```className```、相对定位（uiautomator2支持）、组合定位等；
3. 最后采用```xPath```定位，结合```text```、```description```等缩短 ```xPath``` 长度；
4. 无法识别的元素使用坐标定位方式（需要考虑不同分辨率，按照比例封装工具方法）。


> 微信公众号：daodaotest
