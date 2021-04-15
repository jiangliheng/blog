---
layout: post
title: "ElasticSearch 深度分页总结"
date: "2021-04-15 16:40"
category: ElasticSearch
tags: ElasticSearch
author: jiangliheng
---
* content
{:toc}



# 背景

我们的应用是采用```NLPchina```开源的```elasticsearch-sql```插件来进行查询分页和导出，由于```ElasticSearch```的```max_result_window```的限制，在深度分页和大批量数据导出时就会出现问题，故简单研究下。

```ElasticSearch```的```max_result_window```默认为```10000```条，当使用```elasticsearch-sql```执行```select * from test limit 10000,1```时，```ElasticSearch```就返回错误。

# ```ElasticSearch``` 分页总结

```ElasticSearch``` 是搜索引擎，从搜索的意义上来说，如果筛选条件或前几页都找不到需要的数据，继续深度分页也不会找到想要的数据。

```ElasticSearch``` 不要做深度分页和随机深度跳页。

**ES 分页建议**
- 增加默认的筛选条件，尽量减少数据量的展示，比如：最近一个月；
- 限制总分页数，比如：淘宝、京东仅显示100页查询结果，百度仅显示76页；
- 修改跳页的展现方式，改为滚动显示，或小范围跳页，比如：谷歌、百度的小范围跳页。

**小范围跳页示例**

![](/assets/images/es/16184063304531.jpg)

## ES 三种分页比较

- ```from+size```：适用于浅分页（数据量小于```max_result_window```），在增大```max_result_window```情况下，也可实现深度分页，但效率低下，可能出现 OOM。
- ```scroll```：适用于数据导出，基于生成的历史快照查询，对于数据的变更不会反映到快照上。
- ```search_after```：适用于实时请求和高并发场景（深度分页+排序），由于每一页的数据依赖于上一页最后一条数据，所以无法做到随机跳页（滚动显示）。

## elasticsearch-sql 分页

**分页（```limit```）**：深度跳页和深度随机跳页无法实现，但可做限制页数+小范围跳页的替代方案。

**导出**
- ```scroll```：支持```scroll```方式，具体 sql 语句示例：```SELECT /*! USE_SCROLL(100,30000)*/ firstname , balance FROM accounts```；
- ```csv-result```：有个 csv 导出的实验类功能（未验证）。

> https://github.com/NLPchina/elasticsearch-sql/wiki

# 参考文章
- https://juejin.cn/post/6850037275456339975
- https://www.cnblogs.com/jpfss/p/10815172.html

> 微信公众号：daodaotest
