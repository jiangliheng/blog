---
layout: post
title: "当年偶然发现的 Java Bug（JDK 9及之前仍未修复）"
date: "2020-09-05 13:00"
category: Java
tags: Java JDK
author: jiangliheng
---
* content
{:toc}



# 背景

15年在中信银行做持续集成时，由于当时的项目是基于三方采购的 Java 配置开发平台做的，平台自己基于 ```Ant``` 插件实现了增量和热部署。

其中有几个项目在持续集成部署时，经常发现 ```Linux``` 平台部署成功后（```Windows``` 不会出现，```Linux``` 也是偶发现象），新版本代码并没有生效（反编译 class）。

起初我是在本地 ```windows``` 上跟踪调试基于 ```Ant``` 插件的代码，但始终重现不了（最后测试发现 Windows 无此 Bug）。

后来，通过分析代码逻辑，其中有段逻辑是通过文件的最后修改时间（```File.lastModified()```）来判断要不要覆盖部署的，最后通过单测发现，是由于 ```Java``` 的 ```File.lastModified()``` 方法在 ```Windows``` 和 ```Linux/Unix``` 平台获取的精度不一样导致的，```Windows``` 精度为毫秒，而 ```Linux/Unix``` 只能到秒（JDK Bug：JDK-8177809）。

所以也解释了，为什么是偶发现象，文件修改时间如果判断的两个值正好跨秒时，部署就是成功的，否则失败。

# Bug 重现

**测试代码：FileTest.java**
```java
import java.io.File;
import java.io.IOException;
import java.nio.file.Files;
import java.text.SimpleDateFormat;

public class FileTest {
    private static final long LM = 1599276952718L;

    public static void main(String[] args) throws IOException {
        // java版本号
        System.out.println("Java Version：" + System.getProperty("java.version"));

        File f = new File("test.txt");
        f.createNewFile();

        // 设置最后修改时间
        f.setLastModified(LM);

        // 获取修改时间，存在 bug
        System.out.printf("Test f.lastModified [%s]: %b\n",
f.lastModified(), f.lastModified() == LM);
        // 格式化输出，正确不存在 bug
        System.out.printf("Test f.lastModified DateFormat [%s]\n",new SimpleDateFormat("yyyy-MM-dd HH:mm:ss.sss").format(f.lastModified()));

        // Files.getLastModifiedTime() 获取修改时间，同样存在 bug
        System.out.printf("Test Files.getLastModifiedTime [%s]: %b\n",
Files.getLastModifiedTime(f.toPath()).toMillis(),
(Files.getLastModifiedTime(f.toPath()).toMillis() == LM));
        // 格式化输出，正确不存在 bug
        System.out.printf("Test Files.getLastModifiedTime DateFormat [%s]\n",new SimpleDateFormat("yyyy-MM-dd HH:mm:ss.sss").format(f.lastModified()));

        f.delete();
    }
}
```

命令行下编译、执行:
```bash
# 编译执行
$ javac FileTest.java && java FileTest
```

## Windows 执行结果

Windows 平台不存在此 Bug。

```bash
# 编译执行
$ javac FileTest.java && java FileTest
Java Version：1.8.0_202
Test f.lastModified [1599276952718]: true
Test f.lastModified DateFormat [2020-09-05 11:35:52.052]
Test Files.getLastModifiedTime [1599276952718]: true
Test Files.getLastModifiedTime DateFormat [2020-09-05 11:35:52.052]
```

## Mac 执行结果

JDK 8 最新版本，目前仍然没有修复该问题。

```bash
# 编译执行
$ javac FileTest.java && java FileTest
Java Version：1.8.0_261
Test f.lastModified [1599276952000]: false
Test f.lastModified DateFormat [2020-09-05 11:35:52.052]
Test Files.getLastModifiedTime [1599276952000]: false
Test Files.getLastModifiedTime DateFormat [2020-09-05 11:35:52.052]
```

## Linux 执行结果

```bash
# 编译执行
$ javac FileTest.java && java FileTest
Java Version：1.8.0_171
Test f.lastModified [1599276952000]: false
Test f.lastModified DateFormat [2020-09-05 11:35:52.052]
Test Files.getLastModifiedTime [1599276952000]: false
Test Files.getLastModifiedTime DateFormat [2020-09-05 11:35:52.052]
```

# 官方 Bug 链接
JDK10 已修复，但是之前版本目前仍然未修复。
- https://bugs.openjdk.java.net/browse/JDK-8177809

> 微信公众号：daodaotest
