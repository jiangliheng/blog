---
layout: post
title: "组合测试术语：Pairwise/All-Pairs、OATS（Orthogonal Array Testing Strategy）"
date: "2020-10-21 20:00"
category: 组合测试
tags: Pairwise All-Pairs OAT 配对测试 正交表
author: jiangliheng
---
* content
{:toc}

组合测试（Combinatorial Test）是一种黑盒测试用例生成方法，主要针对多输入参数组合场景。



# 组合测试

组合测试（Combinatorial Test）是一种黑盒测试用例生成方法，主要针对多输入参数组合场景。

目前业界较流行的两种组合测试方法，一种是```Pairwise/All-Pairs```，即配对组合。```OATS（Orthogonal Array Testing Strategy）```，即正交表法。

#  Pairwise/All-Pairs

> In computer science, all-pairs testing or pairwise testing is a combinatorial method of software testing that, for each pair of input parameters to a system (typically, a software algorithm), tests all possible discrete combinations of those parameters. Using carefully chosen test vectors, this can be done much faster than an exhaustive search of all combinations of all parameters, by "parallelizing" the tests of parameter pairs.   --- 维基百科

```Pairwise/All-Pairs```，也叫配对测试 或 结对测试，是一种软件测试的组合方法，核心在于用最少的测试用例来覆盖多个因素取值的两两组合。

**配对测试示例**

```
# 影响因素及可取值
操作系统: macOS, Windows, Linux
浏览器: Chrome, Safari, Firefox
分辨率: 1366x768, 1600×900, 2880x1800

# 配对测试结果集
操作系统	浏览器	分辨率
Linux	Safari	1600×900
Windows	Safari	1366x768
Linux	Firefox	2880x1800
Windows	Chrome	2880x1800
Windows	Firefox	1600×900
macOS	Safari	2880x1800
Linux	Chrome	1366x768
macOS	Chrome	1600×900
macOS	Firefox	1366x768
```

## Pairwise 算法

```Pairwise``` 是 L. L. Thurstone 在 1927 年首先提出来的。他是美国的一位心理统计学家。```Pairwise``` 是基于数学统计和对传统的正交分析法进行优化后得到的产物。

```Pairwise``` 基于如下 2 个假设：
- 每一个维度都是正交的，即每一个维度互相都没有交集；
- 根据数学统计分析，73% 的缺陷（单因素是 35%，双因素是 38%）是由单因素或两个因素相互作用产生的。19% 的缺陷是由三个因素相互作用产生的。

因此，```Pairwise``` 基于覆盖所有两因素的交互作用产生的用例集合性价比最高而产生的。

# N-wise

```N-wise``` 是对 N 个因素的所有取值进行全排列组合（笛卡尔积）而生成的一组测试用例集。理论上，该测试用例集可以发现所有 N 个因素共同作用引发的缺陷。

```Pairwise/All-Pairs``` 是 ```N-wise``` 的一个具体化实例，```Pairwise/All-Pairs``` 实际上就是 ```2-wise```。

《微软软件测试之道》中，建议从 ```Pairwise/All-Pairs``` 开始测试，逐渐提高组合维度，直至```6-wise```组合测试。因为据研究表明，```6-wise```可以发现绝大多数的程序缺陷。但是，实际上随着组合维度的提升，测试用例呈指数爆炸增长，所以 ```Pairwise/All-Pairs``` 或 ```3-wise``` 比较适合实际项目。

**组合数量对比**

组合|全排列组合数量|All-Pairs 组合数量|3-wise 组合数量
:----|:----|:----|:----
2*2|4|4|-
3\*3\*3|27|9|27
4\*4\*4\*4|256|20|78
4\*4\*4\*4\*3\*3\*3\*2\*2\*2|55296|25|100

# N-wise 与 OATS 的区别

## 相同点
- 都属于组合测试方法
- 都可减少测试成本
- 使用频率较高的均是两两组合

## 不同点
- **N-wise**：适用于多因素组合情况下的测试用例生成。
    - 仅考虑两两组合，故测试用例数量固定，但内容不一定一致（引入随机种子，可生成不同的测试用例，如 PICT 中的参数“/r[:N]”）
    - 相比正交表法，测试成本较低
    - 生成用例较方便，有相关工具支持，如 PICT、Allpair等
- **正交表法**：是为正交试验服务的，强调试验数据的均衡搭配。
    - 强调因素间取值组合的“等概率覆盖”
    - 受到“等概率覆盖”的约束，通常比配对测试生成的用例要多，测试成本较高
    - 生成用例较复杂，需要通过正交表进行裁剪、替换参数后才可生成用例

# 组合测试相关工具

**Pairwise 工具集**：http://www.pairwise.org/tools.asp
**正交表查询**：https://www.york.ac.uk/depts/maths/tables/orthogonal.htm

```Pairwise``` 工具推荐微软的 **[PICT](https://github.com/Microsoft/pict)（Pairwise Independent Combinatorial Testing）**。

# 参考文档
- https://www.developsense.com/pairwiseTesting.html
- https://en.wikipedia.org/wiki/All-pairs_testing
- http://www.pairwise.org/

> 微信公众号：daodaotest
