---
layout: post
title: "Jenkins Pipeline 动态参数传递 Git 分支"
date: "2021-04-06 19:40"
category: Jenkins
tags: Jenkins Pipeline Git
author: jiangliheng
---
* content
{:toc}



# 背景

公司其中一个项目采用分支上线模式，每次生产上线都需要修改 Jenkins 任务中的 Git 分支版本，改为参数传递 Git 分支。

# 实现

我们采用参数传递 Git 分支，另外也可使用```Git Parameter```插件实现，会列出所有的 Git 分支。

1. 在 Jenkins 任务中添加 String 类型参数：```GIT_BRANCH```。用于存储 Git 分支名称。![](/assets/images/jenkins/16177042932646.jpg)
2. 在 Pipeline 中配置 Git 分支参数变量：```${GIT_BRANCH}```。![](/assets/images/jenkins/16177070525764.jpg)
3. 就可以将 Git 分支名称通过```GIT_BRANCH```参数传递进行构建。![](/assets/images/jenkins/16177071077545.jpg)


执行后报错：
```stderr: fatal: Couldn't find remote ref refs/heads/${GIT_BRANCH}```

**解决办法**
取消 Pipeline 的```lightweight checkout（轻量级检出）```选项，就可以正常构建。
![](/assets/images/jenkins/16177077824339.jpg)

> https://issues.jenkins.io/plugins/servlet/mobile#issue/JENKINS-28447

> 微信公众号：daodaotest
