---
layout: post
title: "技术债务（Technical debt）的产生原因及衡量解决"
date: "2020-03-10 19:00"
category: 技术债务
tags: 技术债务 SQALE SonarQube
author: jiangliheng
---
* content
{:toc}

> 第一次发布代码，就好比借了一笔钱。只要通过不断重写来偿还债务，小额负债可以加速开发。但久未偿还债务会引发危险。复用马马虎虎的代码，类似于负债的利息。整个部门有可能因为松散的实现，不完全的面向对象的设计或其他诸如此类的负债而陷入窘境。  ---维基百科



# 简介
> 技术负债（英语：Technical debt），又译技术债，也称为设计负债（design debt）、代码负债（code debt），是编程及软件工程中的一个比喻。指开发人员为了加速软件开发，在应该采用最佳方案时进行了妥协，改用了短期内能加速软件开发的方案，从而在未来给自己带来的额外开发负担。这种技术上的选择，就像一笔债务一样，虽然眼前看起来可以得到好处，但必须在未来偿还。软件工程师必须付出额外的时间和精力持续修复之前的妥协所造成的问题及副作用，或是进行重构，把架构改善为最佳实现方式。
1992年，沃德·坎宁安首次将技术的复杂比作为负债。    ---维基百科

通俗易懂的例子：
技术债务类似于金融债务。软件开发就像是去“贷款”，而技术债务就像是它的“利息”，“利息”是需要以未来额外的时间来还的。重构才相当于是支付“本金”。

# 技术负债的产生原因
1. **业务压力**：为了满足业务的快速要求，在必要的修改并没有完成时就匆匆发布，这些未完成的修改就形成了技术负债。
2. **缺少过程和理解**：业务人员不清楚不理解技术负债的概念，在决策时就不会考虑到其带来的影响。
3. **模块之间解耦不够**：功能没有模块化，软件柔性不够，不足适应业务变化的要求。
4. **缺少配套的自动化测试**：导致鼓励快速而风险很大的“创可贴”式的BUG修复。
5. **缺少必要文档**：需求和代码都没有必要的支撑性文档或注释。
6. **缺少协作**：组织中的知识共享和业务效率较低，或者初级开发者缺少必要的指导。
7. **重构延迟**：在开发的过程中，某些部分的代码会变得难以控制，这时候就需要进行重构，以适应将来的需求变化。重构越是推迟，这些已有的代码被使用的越多，形成的技术负债就越多，直到重构完成。
8. **不遵循标准或最佳实践**：忽略了已有的业界标准、框架、技术和最佳实践。
9. **缺少相关技能**：开发人员有时候技能缺失，并不知道如何编写优雅的代码。

# 技术债务的危害
技术负债的“利息”会越滚越多，甚至最后都无法计算其带来的影响。将有技术负债的代码发布到生产系统，就是提高了“利息”的“利率”。最终将会导致技术团队破产，质量无法保障。

# 技术债务衡量（SQALE & SonarQube）
> SQALE (Software Quality Assessment based on Lifecycle Expectations) is a method to support the evaluation of a software application source code. It is a generic method, independent of the language and source code analysis tools.    ---wikipedia

SQALE（基于生命周期期望的软件质量评估）是一种支持软件应用程序源代码评估的方法。它是一种通用方法，独立于语言和源代码分析工具。

SonarQube 中的技术债务就是基于SQALE方法，是通过代码规则和问题来实现的。

**SonarQube 项目技术债务**
![-w1348](/assets/images/debt/15838575705500.jpg)

**SonarQube 代码技术债务详情**
![-w1256](/assets/images/debt/15838576956102.jpg)

**SonarQube 技术债务配置**
![-w1331](/assets/images/debt/15838571566141.jpg)

**SonarQube SQALE 商业插件提供更详细的报告**
![](/assets/images/debt/15838584617527.jpg)
> 原图链接：https://www.bitegarden.com/img/sonarqube-sqale/sonarqube-sqale-dashboard.png

# 技术债务解决（SonarQube）
SonarQube 已经详细的列出了技术债务相关的所有指标和问题，也有很完善的管理和推荐解决方案。

**SonarQube 问题中推荐的解决方法**
![-w1372](/assets/images/debt/15838602071302.jpg)

我强烈推荐大家使用 SonarQube 来管理和解决技术债务。
**后续我会写关于 SonarQube 的系列文章，大家敬请期待~**

# 参考资料
- https://wiki.hk.wjbk.site/baike-%E6%8A%80%E6%9C%AF%E8%B4%9F%E5%80%BA
- https://en.wikipedia.org/wiki/SQALE
- https://www.sonarqube.org/
- https://www.bitegarden.com/sonarqube-sqale

> 微信公众号：daodaotest
