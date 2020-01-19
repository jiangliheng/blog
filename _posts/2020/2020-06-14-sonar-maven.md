---
layout: post
title: "SonarQube + Maven 进行代码分析"
date: "2020-06-14 22:00"
category: SonarQube
tags: SonarQube Maven
author: jiangliheng
---
* content
{:toc}



# 安装设置

参见之前的文章：
- 安装：[Docker Compose 方式安装 SonarQube 8.3.1](/2020/06/14/sonar-docker-install)
- 设置：[SonarQube 插件、权限、质量配置](/2020/06/14/sonar-set/)

# 分析权限设置

为了分析方便，这里设置了一个 sonar 用户，默认配置到 Maven 的 ```settings.xml``` 中，用于 Jenkins 或者本地执行 Sonar 分析代码使用，当然也可以配置具有执行分析权限的用户。

![-w1346](/assets/images/sonar/maven/15921300211822.jpg)


# Maven 设置

settings.xml
```
$ cat settings.xml
<settings xmlns="http://maven.apache.org/SETTINGS/1.0.0"
          xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
          xsi:schemaLocation="http://maven.apache.org/SETTINGS/1.0.0 http://maven.apache.org/xsd/settings-1.0.0.xsd">
    ......
    <pluginGroups>
        <pluginGroup>org.sonarsource.scanner.maven</pluginGroup>
    </pluginGroups>
    .....
    <profile>
      <id>sonar</id>

      <activation>
        <activeByDefault>true</activeByDefault>
      </activation>

      <properties>
        <sonar.host.url>https://xxx.com/sonar</sonar.host.url>
        <!-- 具有执行分析权限的用户名和密码 -->
        <sonar.login>sonar</sonar.login>
        <sonar.password>xxxxxx</sonar.password>
      </properties>
    </profile>
  </profiles>
  ......
</settings>
```

# 分析

```bash
# 执行 sonar 代码分析，跳过单元测试
$ mvn clean verify sonar:sonar -Dmaven.test.skip=true

# 配合 Maven 的 -pl、-am 参数，实现模块代码分析
$ mvn clean verify sonar:sonar -pl nait-per -am
```

# 报告

![-w1375](/assets/images/sonar/maven/15921385021487.jpg)

![-w1349](/assets/images/sonar/maven/15921387583322.jpg)

> 微信公众号：daodaotest
