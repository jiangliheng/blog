---
layout: post
title: "性能测试--3、性能测试过程"
date: "2020-03-09 19:00"
category: 性能测试
tags: 性能测试
author: jiangliheng
---
* content
{:toc}

在性能测试项目中大部分的时间花费在获取需求、验证需求以及实现需求上，只有这样才能为性能测试打下坚实的基础。其余的时间则用于录制事务脚本、执行性能测试和分析测试结果。



# 概念验证（Proof of Concept：POC）
维基百科
> **概念验证（Proof of concept，简称POC）** 是对某些想法的一个较短而不完整的实现，以证明其可行性，示范其原理，其目的是为了**验证一些概念或理论**。概念验证通常被认为是一个有里程碑意义的实作的原型 。

网络解释
> **概念性验证(Proof of Concept；POC)** 正如其字面意义一般，是1种架构师(Architect)为了**「验证」概念是否能确实执行**，所撷取出最精要、核心的解决方案(Solution)，以作为解释架构的概念依据。POC可以协助架构师在验证概念时，以更宏观的角度看待复杂系统，并让所有关连的人更容易提供意见，修改架构，避免落入计较细节，本末倒置的情况发生。 POC除了可以协助架构师更了解系统的概念全貌外，也有助于帮助了解系统内部的结构分析与设计呈现。
POC一般来说，会包含以下几个部分：1、为了验证概念所需的技术架构，如Framework、Pattern；2、利用UML语法所建构的概念模型； 3、模拟解决方案； 4、可被实际执行的解决方案原型(Prototype)。
解决方案的原型，必须要是1个可被验证的框架，强调的是对系统的整体观与结构观，而非单纯的图形介面。这个原型的功用在确定系统架构的大方向，然后才是校正细节。

**在销售环节中，POC提供如下信息：**
- POC提供了一个在技术上评估针对目标程序的性能测试工具的机会 （从技术角度验证性能测试工具的可行性；在被测应用程序上对测试工具进行试验。）；
- 识别脚本数据需求 （针对少数的事务，组建性能测试的基本部分，是一次针对脚本测试阶段的“彩排”，能识别出执行成功所必需的输入数据和运行中的数据需求。）；
- 评估脚本 （估算出编写脚本所需要的时间，为以后的测试提供决策基准）；
- 在目标应用程序上演示性能测试解决方案的能力（POC象形的展示了自动化测试工具的优越性，为你的计划和方案提供决策支持）。

## POC一览表
### 前提
1. 与客户共同制定一套成功或者退出标准，并以书面的形式确定；
2. 配备一个标准的能够满足性能测试工具及其解决方案的最低规格的软件和硬件环境；
3. 应用环境安装必要的监控软件，如服务器和网络监控器；
4. 理想的情况下，有一个独立的权限去访问POC过程中的程序；
5. 业务人员或者熟悉软件的用户，当出现易用性的问题时，提供咨询和建议；
6. 提供技术支持人员（了解程序的构架以及中间环节的工作原理并能解决技术性问题）；
7. 应用环境和被测程序的权限账号（至少准备两套权限用户，多用户操作）；
8. 要至少有两个样本事务组成的POC的基础（一个是简单的只读，另一个应该是对目标数据进行更新的复杂事务）。

## 流程
1. 为每一个样本事务录制两个事件，然后对比两者的不同，确定需要什么样的“运行时数据”（对比工具：Windiff、ExamDiff Pro from prestoSoft、WinMerge）；
2. 完成针对输入和运行时数据需求的识别，以及所有对脚本必需的修改后，还要确定事务能够在单用户和多用户的条件下正确的回放（数据库更新、预期结果、事务回放日志没有报错）。

## 可交付
1. 测试工具成功的运行脚本，回放应用程序的事务，是评价POC通过的标准；
2. POC通过后，可以确认范例事务的输入和运行时数据的要求，并且能够大致了解性能测试项目的数据需求；
3. 确定为了保证脚本准确回放做所有修改，以及评估录制一个应用程序事务脚本所需的大致时间；
4. 在销售方面，可以给客户提供一个良好的印象，满足所有已承诺的成功标准。

# 性能测试具体过程（从需求到完成）
## 过程时间指南
在性能测试项目中大部分的时间花费在获取需求、验证需求以及实现需求上，只有这样才能为性能测试打下坚实的基础。其余的时间则用于录制事务脚本、执行性能测试和分析测试结果。

**关键任务的时间尺度指导：**
1. 录制性能测试脚本：每个事务需要半天的时间；
2. 创建验证测试阶段或者测试场景：一般需要一到两天；
3. 执行性能测试时间：需要至少五天时间（验证问题的测试是未知数；数据库重建也会耗时很多）；
4. 数据收集：需要一天的时间（收集测试结果和（关键性能指标）KPI监控数据）。

### 第一步：需求分析
从所有利益相关者那里收集或咨询各种性能需求（详细说明）。

**其他注意点：**
1. 为性能测试设定一个截止日期，包括已经计划好的时间安排；
2. 决定是外部资源测试还是用内部资源来执行测试（取决于时间进度和自身资源）；
3. 制定测试环境设计方案（尽量接近真实环境，创建的时间要充分考虑）；
4. 确保测试周期汇中，都会把代码冻结应用于测试环境；
5. 确保性能测试中，不会受到其他用户的影响（防止对性能测试执行和结果造成影响）；
6. 确定所有性能测试的目标，并征求各利益方（整个测试团队和相关人员）的同意，在性能测试目标上达成一致；
7. 确认软件的关键失误，并记录在案，以备录制（很重要的过程，可能导致性能测试面临失效的风险）；
8. 确定事务的检查点，特别是一些特殊的监视要求（比如登陆、搜索）；
9. 检查您所选择的事务的输入、目标、运行时数据的需求（数据对于性能测试十分重要；要保证性能测试项目的时间框架内获取足够的准确的数据；同时考虑数据的安全和保密）；
10. 验证性能测试的数目、类型、事务内容以及虚拟用户的配置；思考时间、步进以及负载生成策略；
11. 验证并记录服务器，应用服务器以及KPI（关键性能指标）；尽可能全面的监视软件环境，以保证有可用的信息来验证并解决发生的问题；
12. 验证性能测试结果，并生成性能测试结果与测试目标对比的报告；
13. 制定性能测试中发现缺陷的提交步骤规范。

**内部性能测试额外关注的点：**
1. 团队成员以及汇报制度（建立专门的性能测试团队或有内部测试专家组成的核心团队（大型公司）；最起码要确保您有一位项目经理和足够的性能测试工程师）；
2. 准备好性能测试中需要用到的测试工具和资源（满足团队进行有效性能测试的所有需求）；
3. 确保所有成员都接收了关于所有测试工具的相关培训。

**满足一手要求，继续一下活动：**
1. 制定一个包含资源、时间点以及基于需求的里程碑的总体规划；
2. 制定详细的性能测试计划，包括所有相关的时间点、测试场景和测试用例、负载量，以及环境信息等；
3. 确保计划中包含风险评估，例如时间进度偏差、性能目标未实现等，以防止实际测试与计划的偏离。

**技巧（常被忽略的问题）：**
如果在性能测试执行过程中发现了软件的问题，您要确保计划中为额外测试环境和缺陷解决方案预备了意外事件处理机制。

### 第二步：搭建测试环境
尽早为测试环境准备好硬件、软件、网络设备，它耗费的时间可能比预期要长的多；测试环境尽量和真实环境相似；

**搭建测试环境需要考虑的步骤：**
1. 提前搜索相关设备和配置，为搭建环境准备足够的时间；
2. 考虑所有的配置模型（局域网，广域网）；
3. 把外部系统的链接考虑进去；外部链接可能是性能瓶颈的主要所在；
4. 为目前的测试模型准备足够的负载生成能力；（负载生成的位置：本地、远程）；
5. 确保被测应用程序在测试环境中进行正确的配置；
6. 为被测应用程序和支持应用程序的软件准备足够的软件许可协议；
7. 配置调试性能测试工具；
8. 配置（关键业务指标）KPI监控工具。

### 第三步：录制事务脚本
**事务录制之前，需要做的几点：**
1. 验证事务的运行时数据需求；
2. 确定并运用事务输入数据需求；
3. 决定如何为事务需要特别监控的部分添加检查点（Checkpoint），以评估特定事务的响应时间；
4. 识别并执行应用所录制的脚本之间的不同，这些是事务成功回放所必需的（LR中的关联技术）；
5. 创建性能测试场景之前，确保事务无论从单用户或者多用户的角度都能回放成功。

### 第四步：创建性能测试场景
**考虑如下几点：**
1. 你所做的性能测试属于哪种类型的性能测试：基准测试、负载测试、渗透测试（疲劳测试）、压力测试（峰值测试）、非性能测试；
2. 设置思考时间和步进时间（压力测试除外），真实反映用户情况；
3. 负载生成器配置策略；
4. 为每个负载生成器设置负载生成策略：爆炸式（Big-Bang）、渐进（ramp-up）、渐进（ramp-up）/渐退（ramp-down）；
5. 测试数据准备充分；
6. 考虑是否使用IP欺骗技术；如果用，提前准备合法的IP清单；
7. 考虑带宽的不同情况；
8. 根据服务器和网络KPI指标，确定性能监控软件；
9. 基于网络的性能测试，考虑浏览器缓存（新用户、旧用户、以及访问过的用户），取决于软件的解决方案；
10. 考虑应用技术对您的性能测试设计的所有影响。

### 第五步：执行性能测试
执行性能测试仅仅是验证软件的性能目标。

**注意点：**
1. 证实测试之前，进行预演测试；检查负载生成器是否达标；确定测试环境配置正确；
2. 执行基准测试为性能测试建立一个响应时间的理想值；（每个事务单用户运行一定时间或者多次重复一个事务获得的响应时间）；
3. 执行负载测试时，下一次负载测试前，执行重置数据库（保证性能基线）；
4. 负载测试中发现的问题，需要单独进行测试（考虑计划时，需要安排额外的时间）；
5. 渗透测试（疲劳测试）发现内存泄露或者发现与高数据交互事务执行相关的问题；
6. 压力测试（容量测试或峰值测试），对系统容量的设置具有参考价值；另外，为以后测试中增长的事务容量和最终系统用户提供数据的参考，还可以利用压力测试为处于特定应用级别的服务器设定水平扩展性限制；
7. 执行其他与性能无关的测试（配置不同的负载平衡性、容错灾备等）。

### 第六步（后测试阶段）：分析测试结果、撰写测试报告和环境恢复
1. 数据收集（收集并备份所有在性能测试项目中生成的数据）；
2. 对比项目需求设定的性能目标和测试结果，确定性能测试是否达标（提前确定性能指标的“一致性”）；
3. 根据报告模板将测试结果整理成文档；
4. 将最终结果作为基线数据并用于跟踪最终用户体验。

# 参考文档
- 《应用程序性能测试的艺术》

> 微信公众号：daodaotest