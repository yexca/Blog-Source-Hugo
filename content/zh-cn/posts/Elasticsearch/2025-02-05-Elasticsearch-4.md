---
slug: 229
title: 'Elasticsearch RestClient 查询'
# draft: true
author: yexca
date: '2025-02-05T15:50:26+09:00'
lastmod: '2025-02-15T17:17:08+09:00'
categories:
    - 后端
    - Spring
tags:
    - SpringCloud
---

> **Elasticsearch 系列**
>
> | 内容                   | 链接                                  |
> | :--------------------- | :------------------------------------ |
> | Elasticsearch 基础操作 | <https://blog.yexca.net/archives/226> |
> | Elasticsearch 查询操作 | <https://blog.yexca.net/archives/227> |
> | RestClient 基础操作 | <https://blog.yexca.net/archives/228> |
> | RestClient 查询操作 | 本文 |
> | Elasticsearch 数据聚合 | <https://blog.yexca.net/archives/231> |
> | Elasticsearch 自动补全 | <https://blog.yexca.net/archives/232> |
> | Elasticsearch 数据同步 | <https://blog.yexca.net/archives/234> |
> | Elasticsearch 集群 | <https://blog.yexca.net/archives/235> |

文档的查询同样使用 RestHighLevelClient 对象

## match_all

发起请求如下

```java
@Test
public void testMatchAll() throws IOException {
    // 准备request
    SearchRequest request = new SearchRequest("hotel");
    // 组织DSL参数
    request.source().query(QueryBuilders.matchAllQuery());
    // 发送请求
    SearchResponse response = client.search(request, RequestOptions.DEFAULT);

    System.out.println(response);
}
```

解析响应

```java
@Test
public void testMatchAll() throws IOException {
    // 准备request
    SearchRequest request = new SearchRequest("hotel");
    // 组织DSL参数
    request.source().query(QueryBuilders.matchAllQuery());
    // 发送请求
    SearchResponse response = client.search(request, RequestOptions.DEFAULT);

    // 解析结果
    SearchHits searchHits = response.getHits();
    // 查询的总条数
    long total = searchHits.getTotalHits().value;
    // 查询的结果数组
    SearchHit[] hits = searchHits.getHits();
    for (SearchHit hit : hits) {
        String json = hit.getSourceAsString();
        System.out.println(json);
    }
}
```

es 返回的结果是一个 JSON 字符串，包含：

* hits：命中结果
  * total：总条数，其中 value 是具体的总条数值
  * max_score：所有结果中得分最高的文档相关性算分
  * hits：搜索结果的文档数组，其中的每个文档都是一个 JSON 对象
    * source：文档中的原始数据，也是 JSON 对象

因此，解析响应结果，就是逐层解析JSON字符串，流程如下：

* `SearchHits`：通过 response.getHits() 获取，就是 JSON 中的最外层的 hits，代表命中的结果
  * `SearchHits.getTotalHits().value`：获取总条数信息
  * `SearchHits.getHits()`：获取 SearchHit 数组，也就是文档数组
    * `SearchHit.getSourceAsString()`：获取文档结果中的 _source，也就是原始的 json 文档数据

## match 与 multi_match

与 match_all 类似，差别是查询条件

match 代码

```java
@Test
public void testMatch() throws IOException {
    SearchRequest request = new SearchRequest("hotel");
    request.source().query(QueryBuilders.matchQuery("all", "如家"));
    SearchResponse response = client.search(request, RequestOptions.DEFAULT);

    SearchHits searchHits = response.getHits();
    long total = searchHits.getTotalHits().value;
    System.out.println(total);
    SearchHit[] hits = searchHits.getHits();
    for (SearchHit hit : hits) {
        String json = hit.getSourceAsString();
        System.out.println(json);
    }
}
```

multi_match 代码

```java
@Test
public void testMultiMatch() throws IOException {
    SearchRequest request = new SearchRequest("hotel");
    request.source().query(QueryBuilders.multiMatchQuery("如家", "brand", "name"));
    SearchResponse response = client.search(request, RequestOptions.DEFAULT);

    SearchHits searchHits = response.getHits();
    long total = searchHits.getTotalHits().value;
    System.out.println(total);
    SearchHit[] hits = searchHits.getHits();
    for (SearchHit hit : hits) {
        String json = hit.getSourceAsString();
        System.out.println(json);
    }
}
```

可以看到代码重复部分较多，使用 `Ctrl+Alt+M` 进行代码抽取，term 代码展示了抽取

## 精准查询

term 词条精确匹配查询

```java
@Test
public void testTerm() throws IOException {
    SearchRequest request = new SearchRequest("hotel");
    request.source().query(QueryBuilders.termQuery("city", "上海"));
    SearchResponse response = client.search(request, RequestOptions.DEFAULT);

    responseHandle(response);
}

// 响应处理代码抽取
private static void responseHandle(SearchResponse response) {
    SearchHits searchHits = response.getHits();
    long total = searchHits.getTotalHits().value;
    System.out.println(total);
    SearchHit[] hits = searchHits.getHits();
    for (SearchHit hit : hits) {
        String json = hit.getSourceAsString();
        System.out.println(json);
    }
}
```

range 范围查询

```java
@Test
public void testRange() throws IOException {
    SearchRequest request = new SearchRequest("hotel");
    request.source().query(QueryBuilders
                           .rangeQuery("price")
                           .gte(100)
                           .lte(400));
    SearchResponse response = client.search(request, RequestOptions.DEFAULT);

    responseHandle(response);
}
```

## 布尔查询

```java
@Test
public void testBool() throws IOException {
    SearchRequest request = new SearchRequest("hotel");

    // 构建bool查询
    BoolQueryBuilder booledQuery = QueryBuilders.boolQuery();
    // 添加must条件
    booledQuery.must(QueryBuilders.termQuery("city", "上海"));
    // 添加filter组件
    booledQuery.filter(QueryBuilders.rangeQuery("price").lte(300));

    request.source().query(booledQuery);
    SearchResponse response = client.search(request, RequestOptions.DEFAULT);

    responseHandle(response);
}
```

## 排序与分页

```java
@Test
public void testSort() throws IOException {
    SearchRequest request = new SearchRequest("hotel");
    request.source().query(QueryBuilders.matchAllQuery());
    request.source().from(10).size(10);
    request.source().sort("price", SortOrder.ASC);
    SearchResponse response = client.search(request, RequestOptions.DEFAULT);

    responseHandle(response);
}
```

## 高亮

高亮与上述代码差异较大，请求构建

```java
@Test
public void testHigh() throws IOException {
    SearchRequest request = new SearchRequest("hotel");
    // DSL
    request.source().query(QueryBuilders.matchQuery("all", "汉庭"));
    // 高亮
    request.source().highlighter(
            new HighlightBuilder()
            .field("name")
            .requireFieldMatch(false)
    );
    // 发送请求
    SearchResponse response = client.search(request, RequestOptions.DEFAULT);
        
    // 解析
    responseHandle(response);
}
```

因为查询文档结果与高亮分离，结果解析要额外处理

```java
@Test
public void testHigh() throws IOException {
    SearchRequest request = new SearchRequest("hotel");
    // DSL
    request.source().query(QueryBuilders.matchQuery("all", "汉庭"));
    // 高亮
    request.source().highlighter(
            new HighlightBuilder()
            .field("name")
            .requireFieldMatch(false)
    );
    // 发送请求
    SearchResponse response = client.search(request, RequestOptions.DEFAULT);

    // 解析
    SearchHits searchHits = response.getHits();
    // 总条数
    long total = searchHits.getTotalHits().value;
    System.out.println(total);
    // 文档数组
    SearchHit[] hits = searchHits.getHits();
    for (SearchHit hit : hits) {
        String json = hit.getSourceAsString();
        // 反序列化
        HotelDoc hotelDoc = JSON.parseObject(json, HotelDoc.class);
        // 获取高亮结果
        Map<String, HighlightField> highlightFields = hit.getHighlightFields();
        if(!CollectionUtils.isEmpty(highlightFields)){
            // 获取高亮结果
            HighlightField highlightField = highlightFields.get("name");
            if (highlightField != null){
                String name = highlightField.getFragments()[0].toString();
                // 覆盖非高亮
                hotelDoc.setName(name);
            }
        }
        System.out.println(hotelDoc);
    }
}
```

## 酒店查询案例

实现四部分功能：

* 酒店搜索和分页
* 酒店结果过滤
* 我周边的酒店
* 酒店竞价排名

### 搜索和分页

搜索请求：

* 请求方式：POST
* 请求路径：/hotel/list
* 请求参数：JSON对象，包含4个字段：
  * key：搜索关键字
  * page：页码
  * size：每页大小
  * sortBy：排序，目前暂不实现
* 返回值：分页查询，需要返回分页结果PageResult，包含两个属性：
  * `total`：总条数
  * `List<HotelDoc>`：当前页的数据

首先定义实体类，接收参数

```java
@Data
public class RequestParams {
    private String key;
    private Integer page;
    private Integer size;
    private String sortBy;
}
```

定义返回类

```java
@Data
public class PageResult {
    private Long total;
    private List<HotelDoc> hotels;

    public PageResult(){

    }

    public PageResult(Long total, List<HotelDoc> hotels) {
        this.total = total;
        this.hotels = hotels;
    }
}
```

定义 Controller

```java
@RestController
@RequestMapping("/hotel")
public class HotelController {
    @Autowired
    private IHotelService hotelService;

    @PostMapping("/list")
    public PageResult search(@RequestBody RequestParams params){
        return hotelService.search(params);
    }
}
```

实现搜索业务，首先注册一个 Bean 对象

```java
@Bean
public RestHighLevelClient client(){
    return new RestHighLevelClient(RestClient
            .builder(HttpHost.create("http://ip:9200")
            ));
}
```

编写逻辑

```java
public PageResult search(RequestParams params) {
    // request
    SearchRequest request = new SearchRequest("hotel");
    BoolQueryBuilder boolQuery = QueryBuilders.boolQuery();
    // DSL
    String key = params.getKey();
    if (key == null || "".equals(key)){
        boolQuery.must(QueryBuilders.matchAllQuery());
    }else {
        boolQuery.must(QueryBuilders.matchQuery("all", key));
    }
    // 分页
    int page = params.getPage();
    int size = params.getSize();
    request.source().from((page - 1) * size).size(size);
    // 查询
    request.source().query(boolQuery);
    // 发送请求
    try {
        SearchResponse response = client.search(request, RequestOptions.DEFAULT);

        // 响应解析
        SearchHits searchHits = response.getHits();
        // 总数
        long total = searchHits.getTotalHits().value;
        // 文档
        SearchHit[] hits = searchHits.getHits();
        // 遍历
        List<HotelDoc> hotels = new ArrayList<>();
        for (SearchHit hit : hits) {
            String json = hit.getSourceAsString();
            HotelDoc hotelDoc = JSON.parseObject(json, HotelDoc.class);
            hotels.add(hotelDoc);
        }
        return new PageResult(total, hotels);
    } catch (IOException e) {
        throw new RuntimeException(e);
    }
}
```

### 结果过滤

包含的过滤条件：

* brand：品牌值
* city：城市
* minPrice~maxPrice：价格范围
* starName：星级

修改实体类

```java
@Data
public class RequestParams {
    private String key;
    private Integer page;
    private Integer size;
    private String sortBy;
    // 下面是新增的过滤条件参数
    private String city;
    private String brand;
    private String starName;
    private Integer minPrice;
    private Integer maxPrice;
}
```

修改查询条件

```java
@Override
public PageResult search(RequestParams params) {
    // request
    SearchRequest request = new SearchRequest("hotel");
    basicQuery(params, request);
    // 分页
    int page = params.getPage();
    int size = params.getSize();
    request.source().from((page - 1) * size).size(size);

    // 发送请求
    try {
        SearchResponse response = client.search(request, RequestOptions.DEFAULT);

        // 响应解析
        SearchHits searchHits = response.getHits();
        // 总数
        long total = searchHits.getTotalHits().value;
        // 文档
        SearchHit[] hits = searchHits.getHits();
        // 遍历
        List<HotelDoc> hotels = new ArrayList<>();
        for (SearchHit hit : hits) {
            String json = hit.getSourceAsString();
            HotelDoc hotelDoc = JSON.parseObject(json, HotelDoc.class);
            hotels.add(hotelDoc);
        }
        return new PageResult(total, hotels);
    } catch (IOException e) {
        throw new RuntimeException(e);
    }
}

private static void basicQuery(RequestParams params, SearchRequest request) {
    BoolQueryBuilder boolQuery = QueryBuilders.boolQuery();
    // 输入内容
    String key = params.getKey();
    if (key == null || "".equals(key)){
        boolQuery.must(QueryBuilders.matchAllQuery());
    }else {
        boolQuery.must(QueryBuilders.matchQuery("all", key));
    }
    // brand
    if (params.getBrand() != null && !params.getBrand().equals("")){
        boolQuery.filter(QueryBuilders.termQuery("brand", params.getBrand()));
    }
    // starName
    if (params.getStarName() != null && !params.getStarName().equals("")){
        boolQuery.filter(QueryBuilders.termQuery("starName", params.getStarName()));
    }
    // city
    if (params.getCity() != null && !params.getStarName().equals("")){
        boolQuery.filter(QueryBuilders.termQuery("city", params.getCity()));
    }
    // price
    if (params.getMinPrice() != null && params.getMaxPrice() != null){
        boolQuery.filter(QueryBuilders.rangeQuery("price").gte(params.getMinPrice()).lte(params.getMaxPrice()));
    }
    // 查询
    request.source().query(boolQuery);
}
```

### 附近的酒店

基于 location 坐标，按距离对周围的酒店排序

修改实体类

```java
@Data
public class RequestParams {
    private String key;
    private Integer page;
    private Integer size;
    private String sortBy;
    private String city;
    private String brand;
    private String starName;
    private Integer minPrice;
    private Integer maxPrice;
    // 我当前的地理坐标
    private String location;
}
```

添加距离排序

```java
if (params.getLocation() != null) {
    // 距离排序
    request.source().sort(SortBuilders
            .geoDistanceSort("location", new GeoPoint(params.getLocation()))
            .order(SortOrder.ASC)
            .unit(DistanceUnit.KILOMETERS)
    );
}
```

### 距离显示

修改 HotelDoc，添加距离

```java
@Data
@NoArgsConstructor
public class HotelDoc {
    private Long id;
    private String name;
    private String address;
    private Integer price;
    private Integer score;
    private String brand;
    private String city;
    private String starName;
    private String business;
    private String location;
    private String pic;
    // 距离
    private Object distance;

    public HotelDoc(Hotel hotel) {
        this.id = hotel.getId();
        this.name = hotel.getName();
        this.address = hotel.getAddress();
        this.price = hotel.getPrice();
        this.score = hotel.getScore();
        this.brand = hotel.getBrand();
        this.city = hotel.getCity();
        this.starName = hotel.getStarName();
        this.business = hotel.getBusiness();
        this.location = hotel.getLatitude() + ", " + hotel.getLongitude();
        this.pic = hotel.getPic();
    }
}
```

修改响应处理

```java
for (SearchHit hit : hits) {
    String json = hit.getSourceAsString();
    HotelDoc hotelDoc = JSON.parseObject(json, HotelDoc.class);
    Object[] sortValues = hit.getSortValues();
    if (sortValues.length > 0){
        Object sortValue = sortValues[0];
        hotelDoc.setDistance(sortValue);
    }
    hotels.add(hotelDoc);
}
```

### 添加广告酒店

需求：让指定的酒店在搜索结果中排名置顶

给指定酒店添加标记，在过滤条件中根据此标记判断是否提高 function_score

在 HotelDoc 添加广告标记字段

```java
private Boolean isAD;
```

用 DSL 给一些酒店添加标记

```json
# 添加广告
POST /hotel/_update/607915
{
    "doc": {
        "isAD": true
    }
}
POST /hotel/_update/728461
{
    "doc": {
        "isAD": true
    }
}
POST /hotel/_update/7094829
{
    "doc": {
        "isAD": true
    }
}
POST /hotel/_update/198323591
{
    "doc": {
        "isAD": true
    }
}
```

添加算分函数查询，修改 basicQuery() 方法

```java
private static void basicQuery(RequestParams params, SearchRequest request) {
    BoolQueryBuilder boolQuery = QueryBuilders.boolQuery();
    // 输入内容
    String key = params.getKey();
    if (key == null || "".equals(key)){
        boolQuery.must(QueryBuilders.matchAllQuery());
    }else {
        boolQuery.must(QueryBuilders.matchQuery("all", key));
    }
    // brand
    if (params.getBrand() != null && !params.getBrand().equals("")){
        boolQuery.filter(QueryBuilders.termQuery("brand", params.getBrand()));
    }
    // starName
    if (params.getStarName() != null && !params.getStarName().equals("")){
        boolQuery.filter(QueryBuilders.termQuery("starName", params.getStarName()));
    }
    // city
    if (params.getCity() != null && !params.getStarName().equals("")){
        boolQuery.filter(QueryBuilders.termQuery("city", params.getCity()));
    }
    // price
    if (params.getMinPrice() != null && params.getMaxPrice() != null){
        boolQuery.filter(QueryBuilders
                .rangeQuery("price")
                .gte(params.getMinPrice())
                .lte(params.getMaxPrice())
        );
    }

    // 算分 function_score
    FunctionScoreQueryBuilder functionScoreQuery = QueryBuilders.functionScoreQuery(
            // 原始查询
            boolQuery,
            // 数组
            new FunctionScoreQueryBuilder.FilterFunctionBuilder[]{
                    // 其中一个 function score 元素
                    new FunctionScoreQueryBuilder.FilterFunctionBuilder(
                            // 过滤条件
                            QueryBuilders.termQuery("isAD", true),
                            // 算分函数
                            ScoreFunctionBuilders.weightFactorFunction(10)
                    )
            }
    );
    // 查询
    request.source().query(functionScoreQuery);
}
```
