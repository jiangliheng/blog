---
layout: post
title: "SonarQube Community 实现多分支扫描分析"
date: "2020-06-19 18:00"
category: SonarQube
tags: SonarQube
author: jiangliheng
---
* content
{:toc}



# 背景

同一个 Git 项目，需要分析多个分支的代码扫描。

# 说明

```SonarQube Community``` 版本不支持多分支扫描，

```SonarQube Developer Edition``` 及以上版本是支持多分支扫描的，扫描时指定分支参数```-Dsonar.branch=develop```即可，就可以实现多分支代码扫描。

```bash
$ mvn clean verify sonar:sonar -Dmaven.test.skip=true -Dsonar.branch=master
```

# 社区版多分支扫描

经过搜索和分析 Sonar 扫描原理，目前有2种方式可以实现。
- 开源插件：sonarqube-community-branch-plugin
- 替换 sonar.projectKey，porjectKey 相等于 Sonar 中每个项目的主键 ID，替换后就会以新项目创建

PS: 由于我使用的是 SonarQube 最新版本，目前开源插件还未支持，就暂时使用了第二种。

## 开源插件

插件地址：https://github.com/mc1arke/sonarqube-community-branch-plugin

大致操作步骤：
- 下载插件放到```${SONAR_HOME}/extensions/plugins```目录下，重启 Sonar。
- 扫描时，增加```-Dsonar.branch.name=${GIT_BRANCH}```即可。

## 替换 sonar.projectKey

扫描时，指定不同的 ```sonar.projectKey``` 即可。

```bash
# jenkins 设置 projectName，projectKey 为 job 名称
# job 名称规范： 工程名称-分支名称
$ clean verify sonar:sonar -Dmaven.test.skip=true -Dsonar.projectName=${JOB_NAME} -Dsonar.projectKey=${JOB_NAME}
```

> 微信公众号：daodaotest
