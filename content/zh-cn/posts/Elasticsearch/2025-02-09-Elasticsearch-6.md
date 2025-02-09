---
slug: 232
title: 'Elasticsearch 自动补全'
# draft: true
author: yexca
date: '2025-02-09T17:29:28+09:00'
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
> | RestClient 查询操作 | <https://blog.yexca.net/archives/229> |
> | Elasticsearch 数据聚合 | <https://blog.yexca.net/archives/231> |
> | Elasticsearch 自动补全 | 本文 |

当用户在搜索框输入字符时，应提示与该字符有关的搜索项，根据输入的字母提供完整词条功能，就是自动补全

## 拼音分词

要实现根据字母做补全，就必须对文档按照拼音分词

项目地址：<https://github.com/medcl/elasticsearch-analysis-pinyin>

安装方式同 IK 分词器，以下为在线安装方式，首先进入容器

```bash
docker exec -it es /bin/bash
```

执行命令

```bash
./bin/elasticsearch-plugin  install https://github.com/infinilabs/analysis-pinyin/releases/download/v7.12.1/elasticsearch-analysis-pinyin-7.12.1.zip
```

然后退出重启

```bash
# 退出
exit
# 重启
docker restart es
```

测试

```json
# 测试拼音分词
POST /_analyze
{
  "text": "世界第一可爱",
  "analyzer": "pinyin"
}
```

## 自定义分词器

默认的拼音分词器会将每个汉字单独分为拼音，而我们希望的是每个词条形成一组拼音，需要对拼音分词器做个性化定制，形成自定义分词器

es 中分词器 (analyzer) 的组成分为三部分：

- character filters：在 tokenizer 之前对文本进行处理。例如删除字符、替换字符
- tokenizer：将文本按照一定的规则切割成词条 (term)。例如 keyword，就是不分词；还有 ik_smart
- tokenizer filter：将 tokenizer 输出的词条做进一步处理。例如大小写转换、同义词处理、拼音处理等

![image](https://raw.githubusercontent.com/yexca/picx-images-hosting/master/2024/01-SpringCloud/image.x508n2xgxgw.webp)

声明自定义分词器的语法如下

```json
# 自定义分词器
PUT /test
{
  "settings": {
    "analysis": {
      "analyzer": { // 自定义分词器
        "my_analyzer": { // 分词器名称
          "tokenizer": "ik_max_word",
          "filter": "py"
        }
      },
      "filter": { // 自定义tokenizer filter
        "py": { // 过滤器名称
          "type": "pinyin", // 过滤器类型
            // 配置项在Github有说明
          "keep_full_piny": false,
          "keep_joined_full_pinyin": true,
          "keep_original": true,
          "limit_first_letter_length": 16,
          "remove_duplicated_term": true,
          "none_chinese_pinyin_tokenize": false
        }
      }
    }
  },
  "mappings": {
    "properties": {
      "name": {
        "type": "text",
        "analyzer": "my_analyzer",
        "search_analyzer": "ik_smart"
      }
    }
  }
}
```

测试

```json
# 测试自定义分词器
POST /test/_analyze
{
  "text": "世界第一可爱",
  "analyzer": "my_analyzer"
}
```

## 自动补全查询

es 提供了 [Completion Suggester]([https://www.elastic.co/guide/en/elasticsearch/reference/7.6/search-suggesters.html#completion-suggester](https://www.elastic.co/guide/en/elasticsearch/reference/7.6/search-suggesters.html)) 查询来实现自动补全功能。这个查询会匹配以用户输入内容开头的词条并返回。为了提高补全查询的效率，对于文档中字段的类型有一些约束：

- 参与补全查询的字段必须是 completion 类型
- 字段的内容一般是用来补全的多个词条形成的数组

创建测试索引库

```json
PUT /test
{
  "mappings": {
    "properties": {
      "title": {
        "type": "completion"
      }
    }
  }
}
```

插入测试数据

```json
# 示例数据
POST /test/_doc
{
  "title": ["Sony", "WH-1000XM3"]
}
POST /test/_doc
{
  "title": ["SK-II", "PITERA"]
}
POST /test/_doc
{
  "title": ["Nintendo", "switch"]
}
```

查询

```json
# 自动补全查询
GET /test/_search
{
  "suggest": {
    "title_suggest": {
      "text": "s", // 关键字
      "completion": {
        "field": "title", // 自动补全查询的字段
        "skip_duplicates": true, // 跳过重复
        "size": 10 // 获取前10条数据
      }
    }
  }
}
```

## 自动补全 Java

上述 DSL 的 Java 请求

```java
@Test
public void testAutoIn(){
    SearchRequest request = new SearchRequest("hotel");
    // 请求参数
    request.source()
            .suggest(new SuggestBuilder().addSuggestion(
                    "title_suggest", // 查询名称
                    SuggestBuilders
                            .completionSuggestion("title") // 自动补全查询的字段
                            .prefix("s") // 关键字
                            .skipDuplicates(true) // 跳过重复
                            .size(10) // 获取前10条数据
            ));
    // 发送请求
    SearchResponse response = client.search(request, RequestOptions.DEFAULT);
}
```

响应处理：

```java
@Test
public void testAutoIn(){
    SearchRequest request = new SearchRequest("hotel");
    // 请求参数
    request.source()
            .suggest(new SuggestBuilder().addSuggestion(
                    "mySuggestion",
                    SuggestBuilders
                            .completionSuggestion("suggestion")
                            .prefix("h")
                            .skipDuplicates(true)
                            .size(10)
            ));
    // 发送请求
    try {
        SearchResponse response = client.search(request, RequestOptions.DEFAULT);
        // 处理响应
        Suggest suggest = response.getSuggest();
        // 根据名称获取补全结果
        CompletionSuggestion mySuggestion = suggest.getSuggestion("mySuggestion");
        // 获取options并遍历
        for (CompletionSuggestion.Entry.Option option : mySuggestion.getOptions()) {
            String text = option.getText().string();
            System.out.println(text);
        }
    } catch (IOException e) {
        throw new RuntimeException(e);
    }
}
```

## 酒店搜索自动补全

之前的 hotel 索引库没设置拼音分词器，但索引库无法修改，所以需要删除重建

```json
# 删除重建
DELETE /hotel
PUT /hotel
{
  "settings": {
    "analysis": {
      "analyzer": {
        "text_analyzer": {
          "tokenizer": "ik_max_word",
          "filter": "py"
        },
        "completion_analyzer": {
          "tokenizer": "keyword",
          "filter": "py"
        }
      },
      "filter": {
        "py": {
          "type": "pinyin",
          "keep_full_pinyin": false,
          "keep_joined_full_pinyin": true,
          "keep_original": true,
          "limit_first_letter_length": 16,
          "remove_duplicated_term": true,
          "none_chinese_pinyin_tokenize": false
        }
      }
    }
  },
  "mappings": {
    "properties": {
      "id": {
        "type": "keyword"
      },
      "name": {
        "type": "text",
        "analyzer": "text_analyzer",
        "search_analyzer": "ik_smart",
        "copy_to": "all"
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
        "type": "keyword"
      },
      "starName": {
        "type": "keyword"
      },
      "business": {
        "type": "keyword",
        "copy_to": "all"
      },
      "location": {
        "type": "geo_point"
      },
      "pic": {
        "type": "keyword",
        "index": false
      },
      "all": {
        "type": "text",
        "analyzer": "text_analyzer",
        "search_analyzer": "ik_smart"
      },
      "suggestion": {
        "type": "completion",
        "analyzer": "completion_analyzer"
      }
    }
  }
}
```

修改 HotelDoc 实体类，添加 suggestion 字段

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
    private Object distance;
    // 广告
    private Boolean isAD;
    // 自动补全
    private List<String> suggestion;

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
        // 组装suggestion
        if(this.business.contains("/")){
            // bussiness有多个值，需要切割
            String[] arr = this.business.split("/");
            // 添加元素
            this.suggestion = new ArrayList<>();
            this.suggestion.add(this.brand);
            Collections.addAll(this.suggestion, arr);
        }else {
            this.suggestion = Arrays.asList(this.brand, this.business);
        }
    }
}
```

重新导入数据

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

查询测试

```json
GET /hotel/_search
{
  "query": {
    "match_all": {}
  }
}
```

可以看到查询结果 suggestion 字段，然后编写业务代码

Controller

```java
@GetMapping("/suggestion")
public List<String> getSuggestion(@RequestParam("key") String prefix){
    return hotelService.getSuggestion(prefix);
}
```

Service

```java
public List<String> getSuggestion(String prefix) {
    SearchRequest request = new SearchRequest("hotel");
    request.source().suggest(
            new SuggestBuilder().addSuggestion(
                    "mySuggestion",
                    SuggestBuilders
                            .completionSuggestion("suggestion")
                            .prefix(prefix)
                            .size(10)
                            .skipDuplicates(true)
            )
    );
    try {
        SearchResponse response = client.search(request, RequestOptions.DEFAULT);
        Suggest suggestions = response.getSuggest();
        CompletionSuggestion mySuggestion = suggestions.getSuggestion("mySuggestion");
        List<CompletionSuggestion.Entry.Option> options = mySuggestion.getOptions();
        ArrayList<String> list = new ArrayList<>(options.size());
        for (CompletionSuggestion.Entry.Option option : options) {
            String text = option.getText().string();
            list.add(text);
        }
        return list;
    } catch (IOException e) {
        throw new RuntimeException(e);
    }
}
```
