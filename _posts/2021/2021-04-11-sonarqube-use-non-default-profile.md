---
layout: post
title: "SonarQube 使用非默认质量配置"
date: "2021-04-11 19:00"
category: SonarQube
tags: SonarQube
author: jiangliheng
---
* content
{:toc}



# 背景

SonarQube 代码扫描时使用设置的默认质量配置，不同项目组或同项目不同分支扫描时，会有使用非默认的质量配置需求。

# 不同版本的实现方法

质量配置建议采用继承方式管理，父质量配置为全公司都需要遵守的规则，子质量配置可以自定义。代码扫描时采用子质量配置。

## ```-Dsonar.profile``` 实现（SonarQube 4.5.1之前版本）

```
# 分析时，加上参数 -Dsonar.profile 即可
$ mvn clean verify sonar:sonar -Dmaven.test.skip=true -Dsonar.profile=doadoatest-java
```

> SonarQube 4.5版本之前可以通过```-Dsonar.profile```参数使用非默认质量配置。在 7.6之后的版本已经彻底移除。
> 官方解释：https://jira.sonarsource.com/browse/SONAR-5370

## 项目设置处可自主选择非默认质量配置（SonarQube 8.3 版本验证）

> https://groups.google.com/g/sonarqube/c/aLjY9vSpEwE/m/nSPYOdqVAQAJ

两种实现方式：
1. 先在 SonarQube 的 Web 中设置项目，在项目配置要使用的质量配置；
2. 先首次分析（采用默认的质量配置），然后再在项目配置中选择要使用的质量配置，之后的扫描就采用设置的质量配置。

![](/assets/images/sonar/16181383632343.jpg)

> 微信公众号：daodaotest
