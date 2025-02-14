---
slug: 228
title: 'Elasticsearch RestClient 入门'
# draft: true
author: yexca
date: '2025-02-03T22:30:00+09:00'
lastmod: '2025-02-14T20:36:55+09:00'
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
> | RestClient 基础操作 | 本文 |
> | RestClient 查询操作 | <https://blog.yexca.net/archives/229> |
> | Elasticsearch 数据聚合 | <https://blog.yexca.net/archives/231> |
> | Elasticsearch 自动补全 | <https://blog.yexca.net/archives/232> |
> | Elasticsearch 数据同步 | <https://blog.yexca.net/archives/234> |

ES 官方提供了各种不同语言的客户端，用来操作 ES，这些客户端的本质就是组装 DSL 语句，通过 HTTP 请求发送给 ES

官方文档：<https://www.elastic.co/guide/en/elasticsearch/client/index.html>

以下使用 Java HighLevel Rest Client 客户端 API

## 创建索引库

数据库表结构如下

```sql
`id` bigint(20) NOT NULL COMMENT '酒店id',
`name` varchar(255) CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci NOT NULL COMMENT '酒店名称',
`address` varchar(255) CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci NOT NULL COMMENT '酒店地址',
`price` int(10) NOT NULL COMMENT '酒店价格',
`score` int(2) NOT NULL COMMENT '酒店评分',
`brand` varchar(32) CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci NOT NULL COMMENT '酒店品牌',
`city` varchar(32) CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci NOT NULL COMMENT '所在城市',
`star_name` varchar(16) CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci NULL DEFAULT NULL COMMENT '酒店星级，1星到5星，1钻到5钻',
`business` varchar(255) CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci NULL DEFAULT NULL COMMENT '商圈',
`latitude` varchar(32) CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci NOT NULL COMMENT '纬度',
`longitude` varchar(32) CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci NOT NULL COMMENT '经度',
`pic` varchar(255) CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci NULL DEFAULT NULL COMMENT '酒店图片',
PRIMARY KEY (`id`) USING BTREE
```

创建索引库最关键的是 mapping 映射，需要考虑

* 字段名、字段数据类型 (参考数据库表结构的名称与类型)
* 是否参与搜索 (根据业务判断，例如图片地址不需要参与搜索)
* 是否需要分词 (看内容，例如城市无需分词)
* 分词器是什么 (可以统一 ik_max_word)

上表示例：

```json
# 创建索引库
PUT /hotel
{
  "mappings": {
    "properties": {
      "id": {
        "type": "keyword"
      },
      "name": {
        "type": "text",
        "analyzer": "ik_max_word",
        "copy_to": "all"
        /* 将当前字段拷贝到指定字段 all */
      },
      "address": {
        "type": "keyword",
        "index": false
      },
      "price": {
        "type": "integer"
      },
      "score": {
        "type": "integer"
      },
      "brand": {
        "type": "keyword",
        "copy_to": "all"
      },
      "city": {
        "type": "keyword",
        "copy_to": "all"
      },
      "starName": {
        "type": "keyword"
      },
      "business": {
        "type": "keyword"
      },
      "location": {
        "type": "geo_point"
        /* ES 支持两种地理坐标数据类型 */
      },
      "pic": {
        "type": "keyword",
        "index": false
      },
      "all": {
        /* 组合字段，不会在查询结果显示，但可用于查询 */
        "type": "text",
        "analyzer": "ik_max_word"
      }
    }
  }
}
```

地理坐标数据类型：

* geo_point：由纬度 (latitude) 和经度 (longitude) 确定的一个点。例如 `"32.84 120.25"`
* geo_shape：由多个 geo_point 组成的复杂几何图形。例如一条直线 `"LINESTRING(-77.03 38.29, +77.00 38.88)"`

## 初始化 RestClient

与 elasticsearch 一切交互都封装在一个名为 RestHighLevelClient 的类中

首先引入 es 的 RestHighLevelClient 依赖

```xml
<dependency>
    <groupId>org.elasticsearch.client</groupId>
    <artifactId>elasticsearch-rest-high-level-client</artifactId>
</dependency>
```

因为 SpringBoot 默认的 ES 版本为 7.6.2，需要覆盖默认 ES 版本

```xml
<properties>
    <java.version>1.8</java.version>
    <elasticsearch.version>7.12.1</elasticsearch.version>
</properties>
```

初始化 RestHighLevelClient 代码如下

```java
RestHighLevelClient client = new RestHighLevelClient(RestClient.builder(
        HttpHost.create("http://IP:9200")
));
```

不过为了测试方便，初始化代码放在 @BeforeEach

```java
public class HotelIndexTest {
    private RestHighLevelClient client;

    @BeforeEach
    void setUp() {
        client = new RestHighLevelClient(RestClient.builder(
                        HttpHost.create("http://127.0.0.1:9200")));
    }

    @AfterEach
    void tearDown() throws IOException {
        this.client.close();
    }
}
```

定义索引库需要 DSL 语句的 JSON 部分，可以单独抽出

```java
public class HotelContants {
    public static final String MAPPING_TEMPLATE = "{\n" +
            "  \"mappings\": {\n" +
            "    \"properties\": {\n" +
            "      \"id\": {\n" +
            "        \"type\": \"keyword\"\n" +
            "      },\n" +
            "      \"name\": {\n" +
            "        \"type\": \"text\",\n" +
            "        \"analyzer\": \"ik_max_word\",\n" +
            "        \"copy_to\": \"all\"\n" +
            "      },\n" +
            "      \"address\": {\n" +
            "        \"type\": \"keyword\",\n" +
            "        \"index\": false\n" +
            "      },\n" +
            "      \"price\": {\n" +
            "        \"type\": \"integer\"\n" +
            "      },\n" +
            "      \"score\": {\n" +
            "        \"type\": \"integer\"\n" +
            "      },\n" +
            "      \"brand\": {\n" +
            "        \"type\": \"keyword\",\n" +
            "        \"copy_to\": \"all\"\n" +
            "      },\n" +
            "      \"city\": {\n" +
            "        \"type\": \"keyword\",\n" +
            "        \"copy_to\": \"all\"\n" +
            "      },\n" +
            "      \"starName\": {\n" +
            "        \"type\": \"keyword\"\n" +
            "      },\n" +
            "      \"business\": {\n" +
            "        \"type\": \"keyword\"\n" +
            "      },\n" +
            "      \"location\": {\n" +
            "        \"type\": \"geo_point\"\n" +
            "      },\n" +
            "      \"pic\": {\n" +
            "        \"type\": \"keyword\",\n" +
            "        \"index\": false\n" +
            "      },\n" +
            "      \"all\": {\n" +
            "        \"type\": \"text\",\n" +
            "        \"analyzer\": \"ik_max_word\"\n" +
            "      }\n" +
            "    }\n" +
            "  }\n" +
            "}";
}
```

创建索引代码

```java
// 注意CreateIndexRequest的包
import org.elasticsearch.client.indices.CreateIndexRequest;

@Test
void testCreateHotelIndex() throws IOException {
    // 创建Request对象
    CreateIndexRequest request = new CreateIndexRequest("hotel");
    // 请求参数
    request.source(MAPPING_TEMPLATE, XContentType.JSON);
    // 发起请求
    client.indices().create(request, RequestOptions.DEFAULT);
}
```

## 删除索引库

DSL 语句

```json
DELETE /hotel
```

与创建的代码差异仅在 Request 对象上，并且无参数

```java
@Test
void testDeleteHotelIndex() throws IOException {
    // 创建请求
    DeleteIndexRequest request = new DeleteIndexRequest("hotel");
    // 发起请求
    client.indices().delete(request, RequestOptions.DEFAULT);
}
```

## 判断索引库是否存在

DSL 语句

```json
GET /hotel
```

差异依旧为 Request 对象

```java
@Test
void testExistsHotelIndex() throws IOException {
    // 创建请求
    GetIndexRequest request = new GetIndexRequest("hotel");
    // 发请求
    boolean exists = client.indices().exists(request, RequestOptions.DEFAULT);
    // 输出
    System.out.println(exists);
}
```

> **总结**
>
> JavaRestClient 操作 ES 的流程基本类似，使用 `client.indices()` 方法获取索引库的操作对象

## RestClient

一般数据在数据库通过查询数据库对索引库 CRUD

初始化 RestHighLevelClient

```java
@SpringBootTest
public class HotelDocumentTest {
    // 注入服务
    @Autowired
    private IHotelService hotelService;

    private RestHighLevelClient client;

    @BeforeEach
    void setUp() {
        // 构造 RestClient
        this.client = new RestHighLevelClient(
                RestClient.builder(
                        HttpHost.create("http://127.0.0.1:9200")));
    }

    @AfterEach
    void tearDown() throws IOException {
        this.client.close();
    }
}
```

## 新增文档

将数据库的数据查询出来，写入 ES 中

### 结构调整

数据库查询后的结果是一个 Hotel 类型的对象

```java
@Data
@TableName("tb_hotel")
public class Hotel {
    @TableId(type = IdType.INPUT)
    private Long id;
    private String name;
    private String address;
    private Integer price;
    private Integer score;
    private String brand;
    private String city;
    private String starName;
    private String business;
    private String longitude;
    private String latitude;
    private String pic;
}
```

与索引库结构存在差异，需要定义一个新的类型，与索引库结构相同

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

### 新增语法

DSL 的语法为

```json
POST /{indexName}/_doc/{id}
{
    "name": "Jack",
    "age": 21
}
```

对应的 Java

```java
@Test
public void HotelCreateTest() throws IOException {
    // 准备request对象
    IndexRequest request = new IndexRequest("indexName").id("1");
    // 准备JSON对象
    request.source("{\n" +
            "    \"name\": \"Jack\",\n" +
            "    \"age\": 21\n" +
            "}", XContentType.JSON);
    // 发送请求
    client.index(request, RequestOptions.DEFAULT);
}
```

### 新增文档实例代码

```java
@Test
public void HotelCreateTest() throws IOException {
    // 查询酒店数据
    Hotel hotel = hotelService.getById(61083L);
    // 转换文档类型
    HotelDoc hotelDoc = new HotelDoc(hotel);
    // 转为JSON
    String json = JSON.toJSONString(hotelDoc);

    // 准备request对象
    IndexRequest request = new IndexRequest("hotel").id(hotelDoc.getId().toString());
    // 准备JSON对象
    request.source(json, XContentType.JSON);
    // 发送请求
    client.index(request, RequestOptions.DEFAULT);
}
```

## 查询文档

### 查询语法

DSL 语句

```json
GET /{indexName}/_doc/{id}
```

Java 语句

```java
@Test
public void HotelGetTest() throws IOException {
    // 准备request
    GetRequest request = new GetRequest("indexName", "id");
    // 发送请求
    GetResponse response = client.get(request, RequestOptions.DEFAULT);
    // 解析结果
    String json = response.getSourceAsString();
    
    System.out.println(json);
}
```

### 查询文档实例代码

```java
@Test
public void HotelGetTest() throws IOException {
    // 准备request
    GetRequest request = new GetRequest("hotel", "61083");
    // 发送请求
    GetResponse response = client.get(request, RequestOptions.DEFAULT);
    // 解析结果
    String json = response.getSourceAsString();
    // 反序列化为Java对象
    HotelDoc hotelDoc = JSON.parseObject(json, HotelDoc.class);
    System.out.println(hotelDoc);
}
```

## 删除文档

### 删除语法

DSL 语句

```json
DELETE /{indexName}/_doc/{id}
```

Java 语句

```java
@Test
public void HotelDeleteTest() throws IOException {
    // 准备request
    DeleteRequest request = new DeleteRequest("indexName", "id");
    // 发送请求
    client.delete(request, RequestOptions.DEFAULT);
}
```

### 删除文档实例代码

```java
@Test
public void HotelDeleteTest() throws IOException {
    // 准备request
    DeleteRequest request = new DeleteRequest("hotel", "61083");
    // 发送请求
    client.delete(request, RequestOptions.DEFAULT);
}
```

## 修改文档

### 修改语法

修改有全量修改与增量修改，在 RestClient 的 API 中，两种方式的 API 完全一致，判断依据是 ID，新增时：

* ID 存在，修改
* ID 不存在，新增

这里主要关注增量修改，DSL 语句

```json
POST /{indexName}/_update/{id}
{
    "doc": "Rose",
    "age": 18
}
```

Java 语句

```java
@Test
public void HotelUpdateTest() throws IOException {
    // 创建request对象
    UpdateRequest request = new UpdateRequest("indexName", "id");
    // 准备参数
    request.doc(
            "name", "Rose",
            "age", 18
    );
    // 更新文档
    client.update(request, RequestOptions.DEFAULT);
}
```

### 修改文档实例代码

```java
@Test
public void HotelUpdateTest() throws IOException {
    // 创建request对象
    UpdateRequest request = new UpdateRequest("hotel", "61083");
    // 准备参数
    request.doc(
            "price", 950,
            "starName", "四钻"
    );
    // 更新文档
    client.update(request, RequestOptions.DEFAULT);
}
```

## 批量导入文档

利用 BulkRequest 批量将数据库数据导入到索引库中。本质是将多个普通的 CRUD 请求组合在一起发送，示例

```java
@Test
public void testBulk() throws IOException {
    // 创建bulk请求
    BulkRequest request = new BulkRequest();
    // 添加批量请求
    request.add(new IndexRequest("indexName").id("id1").source("json1", XContentType.JSON));
    request.add(new IndexRequest("indexName").id("id2").source("json2", XContentType.JSON));
    // 发起bulk请求
    client.bulk(request, RequestOptions.DEFAULT);
    }
```

### 批量导入文档实例代码

```java
@Test
public void testBulk() throws IOException {
    // 批量查询数据
    List<Hotel> hotelList = hotelService.list();

    // 创建bulk请求
    BulkRequest request = new BulkRequest();
    // 添加批量请求
    for (Hotel hotel : hotelList) {
        // 转换文档类型
        HotelDoc hotelDoc = new HotelDoc(hotel);
        // 创建新增文档request对象
        request.add(new IndexRequest("hotel")
                .id(hotelDoc.getId().toString())
                .source(JSON.toJSONString(hotelDoc), XContentType.JSON)
        );
    }

    // 发起bulk请求
    client.bulk(request, RequestOptions.DEFAULT);
}
```
