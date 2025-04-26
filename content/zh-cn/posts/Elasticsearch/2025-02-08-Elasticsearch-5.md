---
slug: 231
title: 'Elasticsearch 数据聚合'
# draft: true
author: yexca
date: '2025-02-08T14:56:36+09:00'
lastmod: '2025-02-15T17:17:08+09:00'
categories:
    - 技术学习
tags:
    - 后端技术
    - Elasticsearch
    - SpringCloud
---

> **Elasticsearch 系列**
>
> | 内容                   | 链接                                  |
> | :--------------------- | :------------------------------------ |
> | Elasticsearch 基础操作 | <https://blog.yexca.net/archives/226> |
> | Elasticsearch 查询操作 | <https://blog.yexca.net/archives/227> |
> | RestClient 基础操作 | <https://blog.yexca.net/archives/228> |
> | RestClient 查询操作 | <https://blog.yexca.net/archives/229> |
> | Elasticsearch 数据聚合 | 本文 |
> | Elasticsearch 自动补全 | <https://blog.yexca.net/archives/232> |
> | Elasticsearch 数据同步 | <https://blog.yexca.net/archives/234> |
> | Elasticsearch 集群 | <https://blog.yexca.net/archives/235> |

[聚合 (aggregations)](https://www.elastic.co/guide/en/elasticsearch/reference/current/search-aggregations.html) 可以让我们极其方便地实现对数据的统计、分析、运算。例如：

* 什么品牌的手机最受欢迎？
* 这些手机的平均价格、最高价格、最低价格？
* 这些手机每月的销售情况如何？

## 聚合的种类

常见的有三类：

* 桶 (bucket) 聚合：用来对文档做分组
  * TermAggregation：按照文档字段值分组，例如按照品牌、国家分组
  * Date Histogram：按照日期阶梯分组，例如一周或一月为一组
* 度量 (metric) 聚合：用以计算一些最大值、最小值、平均值等
  * Avg：平均值
  * Max：最大值
  * Min：最小值
  * Stats：同时求 max、min、avg、sum 等
* 管道 (pipeline) 聚合：其他聚合的结果为基础做聚合

> 参加聚合的字段必须是 keyword、日期、数值、布尔类型

## DSL 聚合语句

### bucket

统计所有数据中酒店的品牌有几种，即按品牌对数据分组

```json
# bucket term
GET /hotel/_search
{
  "size": 0, // 设置size为0，结果中不包含文档，只包含聚合结果
  "aggs": {
    "brandAgg": { // 聚合名称
      "terms": { // 聚合类型
        "field": "brand", // 参与聚合字段
        "size": 20 // 获取的聚合结果数量
      }
    }
  }
}
```

### 聚合结果排序

默认情况下，bucket 聚合会统计 bucket 内的文档数量，记为 count，并且按 count 降序排序。通过指定 order 属性，自定义聚合的排序方式

```json
GET /hotel/_search
{
  "size": 0,
  "aggs": {
    "brandAgg": {
      "terms": {
        "field": "brand",
        "size": 20,
        "order": { // 排序
          "_count": "asc"
        }
      }
    }
  }
}
```

### 限定聚合范围

默认情况下会对索引库所有文档聚合，但实际使用时，用户会输入搜索条件，因此聚合必须是对搜索结果的聚合，聚合就必须要添加限定条件

```json
# bucket query
GET /hotel/_search
{
  "query": {
    "range": {
      "price": {
        "lte": 200 // 只对价格小于200的文档聚合
      }
    }
  },
  "size": 0,
  "aggs": {
    "brandAggQuery": {
      "terms": {
        "field": "brand",
        "size": 20
      }
    }
  }
}
```

### Metric

上述 bucket 聚合按品牌分组，现在要获取每个品牌用户评分的 min、max、avg

```json
# metric
GET /hotel/_search
{
  "size": 0,
  "aggs": {
    "brandAgg": {
      "terms": {
        "field": "brand",
        "size": 20
      },
      "aggs": { // bucket的子聚合，对分组后每个组运算
        "scoreStats": { // 聚合名称
          "stats": { // 聚合类型
            "field": "score" // 聚合字段
          }
        }
      }
    }
  }
}
```

根据平均值排序

```json
GET /hotel/_search
{
  "size": 0,
  "aggs": {
    "brandAgg": {
      "terms": {
        "field": "brand",
        "size": 20,
        "order": {
          "scoreStats.avg": "desc" // 平均值降序
        }
      },
      "aggs": {
        "scoreStats": {
          "stats": {
            "field": "score"
          }
        }
      }
    }
  }
}
```

## RestAPI 聚合

### 语法

聚合条件与 query 同级，因此使用 `request.source()` 指定聚合条件

```java
@Test
public void testAggTerm() throws IOException {
    SearchRequest request = new SearchRequest("hotel");

    request.source().size(0);
    request.source().aggregation(
            AggregationBuilders
                    .terms("brandAgg")
                    .field("brand")
                    .size(20)
    );

    SearchResponse response = client.search(request, RequestOptions.DEFAULT);
}
```

响应处理

```java
@Test
public void testAggTerm() throws IOException {
    SearchRequest request = new SearchRequest("hotel");

    request.source().size(0);
    request.source().aggregation(
            AggregationBuilders
                    .terms("brandAgg")
                    .field("brand")
                    .size(20)
    );

    SearchResponse response = client.search(request, RequestOptions.DEFAULT);

    // 解析聚合结果
    Aggregations aggregations = response.getAggregations();
    // 按名称获取聚合结果
    Terms term = aggregations.get("brandAgg");
    // 获取bucket
    List<? extends Terms.Bucket> buckets = term.getBuckets();
    // 遍历
    for (Terms.Bucket bucket : buckets) {
        // 获取key
        String name = bucket.getKeyAsString();
        System.out.println(name);
    }
}
```

### 实例需求

前端页面的城市、星级、品牌都是固定供选择的，不会随着搜索输入而改变

但是假如搜索 "东方明珠"，那城市只能是上海，不应该显示其他城市

也就是可供选择的城市等应该随着搜索输入的内容改变，为此，前端需要根据内容请求可选城市，假设接口如下：

* 请求方式：`POST`
* 请求路径：`/hotel/filters`
* 请求参数：`RequestParams`，与搜索文档的参数一致
* 返回值类型：`Map<String, List<String>>`

Controller

```java
@PostMapping("/filters")
public Map<String, List<String>> getFilters(@RequestBody RequestParams params){
    return hotelService.getFilters(params);
}
```

Service

```java
public Map<String, List<String>> getFilters(RequestParams params) {
    // request
    SearchRequest request = new SearchRequest("hotel");
    // DSL
    basicQuery(params, request);
    // 设置size
    request.source().size(0);
    // 聚合
    request.source().aggregation(
            AggregationBuilders
                    .terms("brandAgg")
                    .field("brand")
                    .size(100)
    );
    request.source().aggregation(
            AggregationBuilders
                    .terms("cityAgg")
                    .field("city")
                    .size(100)
    );
    request.source().aggregation(
            AggregationBuilders
                    .terms("starAgg")
                    .field("starName")
                    .size(100)
    );
    // 请求
    try {
        SearchResponse response = client.search(request, RequestOptions.DEFAULT);

        // 解析响应
        Map<String, List<String>> result = new HashMap<>();
        Aggregations aggregations = response.getAggregations();
        // 品牌
        List<String> brandList = getAggName(aggregations, "brandAgg");
        result.put("品牌", brandList);
        // 城市
        List<String> cityList = getAggName(aggregations, "cityAgg");
        result.put("城市", cityList);
        // 星级
        List<String> starList = getAggName(aggregations, "starAgg");
        result.put("星级", starList);
        return result;
    } catch (IOException e) {
        throw new RuntimeException(e);
    }
}

private static List<String> getAggName(Aggregations aggregations, String name) {
    // 获取品牌
    Terms brand = aggregations.get(name);
    // 获取bucket
    List<? extends Terms.Bucket> buckets = brand.getBuckets();
    // 遍历
    List<String> brandList = new ArrayList<>();
    for (Terms.Bucket bucket : buckets) {
        // 获取key
        String key = bucket.getKeyAsString();
        brandList.add(key);
    }
    return brandList;
}
```
