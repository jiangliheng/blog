---
layout: post
title: "JVM 调优之 Eclipse 启动调优实战"
date: "2020-03-21 10:00"
category: JVM
tags: JVM Eclipse
author: jiangliheng
---
* content
{:toc}

本文是我12年在学习《深入理解Java虚拟机：JVM高级特性与最佳实践》时，做的一个 JVM 简单调优实战笔记，版本都有些过时，不过调优思路和过程还是可以分享给大家参考的。



# 环境基础配置

**硬件：** Dell E5410， Intel i3 CPU M 370， 2GB内存

**系统：** 32位 Windows XP

**虚拟机：** Java HotSpot(TM) Client VM (build 17.1-b03， mixed mode， sharing)

**Eclipse版本：** Release 4.2.0  Last revised June 8th， 2012
# 调优前
## Eclipse初始配置文件
**eclipse.ini**

```
-startup
plugins/org.eclipse.equinox.launcher_1.3.0.v20120522-1813.jar
--launcher.library
plugins/org.eclipse.equinox.launcher.win32.win32.x86_1.1.200.v20120522-1813
-product
org.eclipse.epp.package.jee.product
--launcher.defaultAction
openFile
--launcher.XXMaxPermSize
256M
-showsplash
org.eclipse.platform
--launcher.XXMaxPermSize
256m
--launcher.defaultAction
openFile
-vmargs
-Dosgi.requiredJavaVersion=1.5
-Dhelp.lucene.tokenizer=standard
-Xms40m
-Xmx512m
```

## 调优前的测试结果
- 平均耗时约: 10 秒
- 垃圾收集器总耗时 1.710 s，其中:
  - Full GC被触发了 8 次，耗时 1.438 s；
  - Minor GC被触发了 29 次，耗时 272.185 ms；
  - Eclipse 运行一段时间后，会显式的调用 System.gc() 方法做 Full GC。
- 加载类：8279 个，耗时 7.598 s
- JIT的编译时间为： 1.829 s
- 虚拟机 512MB 的内存被分配为 170MB 的新生代(Eden 区为 136MB 和两个 17MB的Survivor 区)和 342MB 的老年代
![](/assets/images/jvm-eclipse/15847600762230.jpg)

Eclipse启动过程:
![](/assets/images/jvm-eclipse/15847600801609.jpg)

Eclipse启动后，运行一段时间:
![](/assets/images/jvm-eclipse/15847600848474.jpg)


# 分析及调优
## 升级JDK版本
获取免费的“性能提升”(这里暂时不做考虑)。

## 类加载和编译时间优化
**类加载：** 字节码验证优化。 考虑到实际情况: Eclipse 使用者甚多，它的编译代码是可靠的，不需要在加载的时候再进行字节码验证。

**优化措施：** 添加参数 ```-Xverify:none```，禁止字节码验证。

**编译时间：** 无

## 调整内存设置控制垃圾收集频率
### 第一次
新生代 Minor GC 的发生是因为分配给新生代的空间太小导致的。

Full GC产生的原因，从GC日志中分析:
```
2.558: [Full GC 2.558: [Tenured: 5029K->6089K(27328K)， 0.0769168 secs] 7919K->6089K(39616K)， [Perm : 16383K->16383K(16384K)]， 0.0769992 secs] [Times: user=0.08 sys=0.00， real=0.08 secs]
4.740: [Full GC 4.740: [Tenured: 9085K->11372K(27328K)， 0.1007947 secs] 13555K->11372K(39680K)， [Perm : 20479K->20479K(20480K)]， 0.1008572 secs] [Times: user=0.08 sys=0.00， real=0.09 secs]
5.853: [Full GC 5.853: [Tenured: 13120K->16847K(27328K)， 0.1368830 secs] 25460K->16847K(39680K)， [Perm : 24575K->24575K(24576K)]， 0.1369769 secs] [Times: user=0.14 sys=0.00， real=0.14 secs]
6.317: [Full GC 6.317: [Tenured: 16847K->17236K(28080K)， 0.1789713 secs] 20615K->17236K(40752K)， [Perm : 28671K->28646K(28672K)]， 0.1791253 secs] [Times: user=0.17 sys=0.00， real=0.17 secs]
7.247: [Full GC 7.248: [Tenured: 19043K->20238K(28728K)， 0.1739436 secs] 20408K->20238K(41720K)， [Perm : 32767K->32767K(32768K)]， 0.1740637 secs] [Times: user=0.17 sys=0.00， real=0.17 secs]
7.928: [Full GC 7.928: [Tenured: 20238K->22897K(33732K)， 0.2087824 secs] 31501K->22897K(48964K)， [Perm : 36863K->36863K(36864K)]， 0.2093529 secs] [Times: user=0.20 sys=0.00， real=0.20 secs]
9.892: [Full GC 9.892: [Tenured: 22897K->25238K(38164K)， 0.2628295 secs] 38655K->25238K(55444K)， [Perm : 40959K->40959K(40960K)]， 0.2629460 secs] [Times: user=0.27 sys=0.00， real=0.27 secs]
```

**日志打印参数**
```
-XX:+PrintGCTimeStamps
-XX:+PrintGCDetails
-Xloggc:gc.log
```

从上面可以看出来，7 次 Full GC 中，最后 4 次 Full GC 是为了老年代扩容，从 27328K 扩容到最后 38164K 。从数据中，可以看出前 3 次的 Full GC 是为了永久代的扩容而触发的。

**由上述分析可以得出结论**

Eclipse 启动的时候，Full GC 大多数是由于老年代和永久代容量扩展而导致的，为了避免性能浪费，可以把```-Xms```和```-XX:PermSize```参数值分别设置为```-Xmx```和```-XX:PermSizeMax```参数值，即大小设置一致，减少自动扩容。

**优化具体措施**

新生代设置为 170MB，避免新生代频繁 GC 做扩容:
把 Java 堆、永久代的容量分别固定为 512MB 和 256MB，避免内存扩展。

**调整后的eclipse.ini内容**

```
-startup
plugins/org.eclipse.equinox.launcher_1.3.0.v20120522-1813.jar
--launcher.library
plugins/org.eclipse.equinox.launcher.win32.win32.x86_1.1.200.v20120522-1813
-product
org.eclipse.epp.package.jee.product
--launcher.defaultAction
openFile
--launcher.XXMaxPermSize
256M
-showsplash
org.eclipse.platform
--launcher.XXMaxPermSize
256m
--launcher.defaultAction
openFile
-vmargs
-Dosgi.requiredJavaVersion=1.5
-Dhelp.lucene.tokenizer=standard
-Xverify:none
-Xms512m
-Xmx512m
-Xmn170m
-XX:PermSize=256m
```

### 第二次
调优后运行结果，再次分析:
![](/assets/images/jvm-eclipse/15847607559229.jpg)

![](/assets/images/jvm-eclipse/15847607595782.jpg)

![](/assets/images/jvm-eclipse/15847607640128.jpg)

从 Old Gen 曲线上看，老年代直接固定在 342MB，使用 25.69MB，并且一直很平滑，完全不应该发生 Full GC 才对。再从 GC 日志看，发现 Full GC 后面标记了一个 System，而图中GC time  的 Last Cause:System.gc() 为，说明系统显式的调用了 System.gc()。


**优化措施：**

去除显式的调用 System.gc() ，添加参数```-XX:+DisableExplicitGC```屏蔽掉 System.gc()。

**调整后的eclipse.ini内容：**

```
-startup
plugins/org.eclipse.equinox.launcher_1.3.0.v20120522-1813.jar
--launcher.library
plugins/org.eclipse.equinox.launcher.win32.win32.x86_1.1.200.v20120522-1813
-product
org.eclipse.epp.package.jee.product
--launcher.defaultAction
openFile
--launcher.XXMaxPermSize
256M
-showsplash
org.eclipse.platform
--launcher.XXMaxPermSize
256m
--launcher.defaultAction
openFile
-vmargs
-Dosgi.requiredJavaVersion=1.5
-Dhelp.lucene.tokenizer=standard
-Xverify:none
-Xms512m
-Xmx512m
-Xmn170m
-XX:PermSize=256m
-XX:+DisableExplicitGC
```

# 调优后
## 类加载优化结果
添加参数```-Xverify:none```后，类加载时间从 7.598s 减少到 4.665s。

![](/assets/images/jvm-eclipse/15847622779831.jpg)

![](/assets/images/jvm-eclipse/15847622963954.jpg)

## 调整内存设置控制后结果
### 第一次:
![](/assets/images/jvm-eclipse/15847623065278.jpg)

![](/assets/images/jvm-eclipse/15847623106869.jpg)


![](/assets/images/jvm-eclipse/15847623155266.jpg)


GC日志:
```
2.845: [GC 2.846: [DefNew: 139264K->7351K(156672K)， 0.0463358 secs] 139264K->7351K(506880K)， 0.0463981 secs] [Times: user=0.05 sys=0.00， real=0.05 secs]
4.581: [GC 4.582: [DefNew: 146615K->16818K(156672K)， 0.0786477 secs] 146615K->16818K(506880K)， 0.0787217 secs] [Times: user=0.08 sys=0.00， real=0.08 secs]
64.279: [Full GC (System) 64.279: [Tenured: 0K->26315K(350208K)， 0.3266376 secs] 127364K->26315K(506880K)， [Perm : 39195K->39195K(262144K)]， 0.3267561 secs] [Times: user=0.31 sys=0.01， real=0.33 secs]
```

在调整后的参数设置下， GC 次数已经大幅度减少。上图是 Eclipse 启动后 1分钟的监视曲线，只发生了 2 次 Minor GC 和 1 次 Full GC，总耗时 451.603ms。相比之前的1.710s，减少了2倍多.


### 第二次:
![](/assets/images/jvm-eclipse/15847623607815.jpg)

已经完全没有 Full GC 了。

# 总结

以上只是专门针对 Eclipse 的启动过程进行分析和调优，并未对 Eclipse 日常开发工作进行分析和调优。

# 参考资料
- 《深入理解Java虚拟机：JVM高级特性与最佳实践》

> 微信公众号：daodaotest
