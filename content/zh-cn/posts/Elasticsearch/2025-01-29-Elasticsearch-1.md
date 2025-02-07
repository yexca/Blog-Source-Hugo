---
slug: 226
title: 'Elasticsearch 入门'
# draft: true
author: yexca
date: '2025-01-29T23:38:51+09:00'
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
> | Elasticsearch 基础操作 | 本文                                  |
> | Elasticsearch 查询操作 | <https://blog.yexca.net/archives/227> |
> | RestClient 基础操作 | <https://blog.yexca.net/archives/228> |
> | RestClient 查询操作 | <https://blog.yexca.net/archives/229> |

Elasticsearch 是一款非常强大的开源搜索引擎，可以帮助我们从海量数据中快速找到需要的内容。结合 kibana、Logstash、Beats，也就是 elastic stack (ELK)。被广泛应用在日志数据分析、实时监控等领域

而 Elasticsearch 是 Elastic Stack 的核心，负责存储、搜索、分析数据

Elasticsearch 底层基于 Lucene 实现，Lucene 为 Java 的一个搜索引擎类库

## 正向索引

传统数据库 (例如 MySQL) 采用正向索引，例如下表

| id   | title          | price |
| ---- | -------------- | ----- |
| 1    | 小米手机       | 3499  |
| 2    | 华为手机       | 4999  |
| 3    | 华为小米充电器 | 49    |
| 4    | 小米手环       | 239   |

如果基于 id 精准查询，直接走索引会很快

但若基于 title 做模糊查询，只能逐行扫描数据，流程：

1. 用户搜索 *手机*，数据库条件 `%手机%`
2. 逐行获取数据，如 id 为 1 的数据
3. 判断数据中的 title 是否符合条件
4. 符合则放入，不符合则舍弃，下一行

随着数据量的增加，逐行扫描的效率越来越低

## 倒排索引

倒排索引的概念是基于 MySQL 这样的正向索引而言的

Elasticsearch 采用倒排索引，概念：

* 文档 (document)：每条数据就是一个文档
* 词条 (term)：文档按照语义分成的词语

创建倒排索引是对正向索引的一种特殊处理，流程：

1. 将每一个文档的数据利用算法分词，得到一个个词条
2. 创建表，每行包括词条、词条所在文档 id、位置等信息
3. 因为词条的唯一性，可以给词条创建索引，例如 hash 表结构索引

例如上例的表可以创建如下倒排索引

| 词条   | 文档 id |
| ------ | ------- |
| 小米   | 1，3，4 |
| 手机   | 1，2    |
| 华为   | 2，3    |
| 充电器 | 3       |
| 手环   | 4       |

倒排索引搜索流程：

1. 用户搜索 `小米手机`
2. 对搜索内容分词，得到 `小米`、`手机`
3. 使用词条在倒排索引查找，得到包含词条的文档 id：1、2、3、4
4. 使用文档 id 到正向索引中查找具体文档

### 文档

Elasticsearch 是面向文档存储的，可以是数据库中的一条商品数据，一个订单信息。文档数据会被序列化为 json 格式后存储在 Elasticsearch 中

上述正向索引表的 json 如下

```json
{
    "id": 1,
    "title": "小米手机",
    "price": 3499
}
{
    "id": 2,
    "title": "华为手机",
    "price": 4999
}
{
    "id": 3,
    "title": "华为小米充电器",
    "price": 49
}
{
    "id": 4,
    "title": "小米手环",
    "price": 299
}
```

在 Json 文档中包含很多字段，类似于数据库中的列

### 索引与映射

索引 (index) 为相同类型的文档的集合

映射 (mapping) 为索引中文档的字段约束信息，类似表的结构约束

可以把索引当做是数据库中的表，数据库的表会有约束信息，用来定义表的结构、字段的名称、类型等信息。因此，索引库中就有映射，是索引中文档的字段约束信息，类似表的结构约束

## MySQL 与 Elasticsearch

| MySQL  | Elasticsearch | 说明                                                         |
| ------ | ------------- | ------------------------------------------------------------ |
| Table  | Index         | 索引 (index)，就是文档的集合，类似数据库的表 (table)         |
| Row    | Document      | 文档 (Document)，就是一条条的数据，类似数据库中的行 (Row)，文档都是 JSON 格式 |
| Column | Field         | 字段 (Field)，就是 JSON 文档中的字段，类似数据库中的列 (Column) |
| Schema | Mapping       | Mapping (映射) 是索引中文档的约束，例如字段类型约束。类似数据库的表结构 (Schema) |
| SQL    | DSL           | DSL 是 elasticsearch 提供的 JSON 风格的请求语句，用来操作 elasticsearch，实现 CRUD |

在企业中，往往是两者结合使用：

* 对安全性要求较高的写操作，使用 mysql 实现
* 对查询性能要求较高的搜索需求，使用 elasticsearch 实现
* 两者再基于某种方式，实现数据的同步，保证一致性

### 优缺点

**正向索引**：

* 优点：
  * 可以给多个字段创建索引
  * 根据索引字段搜索、排序速度非常快
* 缺点：
  * 根据非索引字段，或者索引字段中的部分词条查找时，只能全表扫描

**倒排索引**：

* 优点：
  * 根据词条搜索、模糊搜索时，速度非常快
* 缺点：
  * 只能给词条创建索引，而不是字段
  * 无法根据字段做排序

## 安装

一般只用 Elasticsearch 即可，使用 kibana 可以提供一个 elasticsearch 的可视化界面，方便学习书写 DSL 语句

### Elasticsearch

为了使 Elasticsearch 与 kibana 容器互联，可以先创建一个网络

```bash
docker network create es-net
```

> 有多种方式可以实现互联，如 docker-compose、172.17.0.1

拉取 Elasticsearch

```bash
docker pull elasticsearch:7.12.1
```

单点部署

```bash
docker run -d \
    --name es \
    -e "ES_JAVA_OPTS=-Xms512m -Xmx512m" \
    -e "discovery.type=single-node" \
    -v es-data:/usr/share/elasticsearch/data \
    -v es-plugins:/usr/share/elasticsearch/plugins \
    --privileged \
    --network es-net \
    -p 9200:9200 \
    -p 9300:9300 \
elasticsearch:7.12.1
```

注意修改映射目录，上述使用数据卷，部分解释

* `-e "cluster.name=es-docker-cluster"`：设置集群名称
* `-e "http.host=0.0.0.0"`：监听的地址，可以外网访问
* `-e "ES_JAVA_OPTS=-Xms512m -Xmx512m"`：内存大小
* `-e "discovery.type=single-node"`：非集群模式
* `-v es-data:/usr/share/elasticsearch/data`：挂载逻辑卷，绑定es的数据目录
* `-v es-logs:/usr/share/elasticsearch/logs`：挂载逻辑卷，绑定es的日志目录
* `-v es-plugins:/usr/share/elasticsearch/plugins`：挂载逻辑卷，绑定es的插件目录
* `--privileged`：授予逻辑卷访问权
* `--network es-net` ：加入一个名为es-net的网络中

访问 <localhost:9200> 查看返回类似下述即启动成功

```json
{
  "name" : "6747e3f712ba",
  "cluster_name" : "docker-cluster",
  "cluster_uuid" : "GSLtjxiMSlyRRRW-pSzvWQ",
  "version" : {
    "number" : "7.12.1",
    "build_flavor" : "default",
    "build_type" : "docker",
    "build_hash" : "3186837139b9c6b6d23c3200870651f10d3343b7",
    "build_date" : "2021-04-20T20:56:39.040728659Z",
    "build_snapshot" : false,
    "lucene_version" : "8.8.0",
    "minimum_wire_compatibility_version" : "6.8.0",
    "minimum_index_compatibility_version" : "6.0.0-beta1"
  },
  "tagline" : "You Know, for Search"
}
```

### kibana

拉取同版本镜像

```bash
docker pull kibana:7.12.1
```

运行

```bash
docker run -d \
--name kibana \
-e ELASTICSEARCH_HOSTS=http://es:9200 \
--network=es-net \
-p 5601:5601  \
kibana:7.12.1
```

其中 `-e ELASTICSEARCH_HOSTS=http://es:9200"`：设置 elasticsearch 的地址，因为 kibana 已经与 elasticsearch 在一个网络，因此可以用容器名直接访问 elasticsearch

kibana 启动一般比较慢，需要多等一会，可以查看日志，若出现端口号则启动成功

```bash
docker logs -f kibana
```

访问 <localhost:5601> 查看结果

### IK 分词器

es 在创建倒排索引时需要对文档分词；在搜索时，需要对用户输入内容分词。但默认的分词规则对中文处理并不友好，例如测试

```json
# 测试分词
POST /_analyze
{
  "analyzer": "standard",
  "text": "初次使用 Elasticsearch"
}
```

语法说明

* POST：请求方式
* /_analyze：请求路径。这里省略了 <http://localhost:9200>，由 kibana 补充
* 请求参数使用 JSON
  * analyzer：分词器类型，默认为 standard
  * text：需要分词的内容

结果为

```json
{
  "tokens" : [
    {
      "token" : "初",
      "start_offset" : 0,
      "end_offset" : 1,
      "type" : "<IDEOGRAPHIC>",
      "position" : 0
    },
    {
      "token" : "次",
      "start_offset" : 1,
      "end_offset" : 2,
      "type" : "<IDEOGRAPHIC>",
      "position" : 1
    },
    {
      "token" : "使",
      "start_offset" : 2,
      "end_offset" : 3,
      "type" : "<IDEOGRAPHIC>",
      "position" : 2
    },
    {
      "token" : "用",
      "start_offset" : 3,
      "end_offset" : 4,
      "type" : "<IDEOGRAPHIC>",
      "position" : 3
    },
    {
      "token" : "elasticsearch",
      "start_offset" : 5,
      "end_offset" : 18,
      "type" : "<ALPHANUM>",
      "position" : 4
    }
  ]
}
```

可以看到分词效果非常不好，处理中文分词，一般会使用IK分词器

IK 分词器 Github：<https://github.com/medcl/elasticsearch-analysis-ik>

#### 在线安装

注意安装版本与 es 对应

```bash
# 进入容器内部
docker exec -it elasticsearch /bin/bash

# 在线下载并安装
./bin/elasticsearch-plugin  install https://github.com/medcl/elasticsearch-analysis-ik/releases/download/v7.12.1/elasticsearch-analysis-ik-7.12.1.zip

#退出
exit
#重启容器
docker restart elasticsearch
```

#### 离线安装

安装插件需要知道 elasticsearch 的 plugins 目录位置，上述使用数据卷挂载到本地，可使用下面命令查看

```bash
docker volume inspect es-plugins
```

输出的 JSON 的 `Mountpoint` 即为目录

将从 Github 下载的压缩包解压后文件夹重命名为 `ik`，放到 plugins 目录下

重启容器

```bash
docker restart es
```

#### 测试效果

IK 分词器有两种模式

* ik_smart：最少切分
* ik_max_word：最细切分

还是上例

```json
# 测试IK分词
POST /_analyze
{
  "analyzer": "ik_smart",
  "text": "初次使用 Elasticsearch"
}
```

结果

```json
{
  "tokens" : [
    {
      "token" : "初次",
      "start_offset" : 0,
      "end_offset" : 2,
      "type" : "CN_WORD",
      "position" : 0
    },
    {
      "token" : "使用",
      "start_offset" : 2,
      "end_offset" : 4,
      "type" : "CN_WORD",
      "position" : 1
    },
    {
      "token" : "elasticsearch",
      "start_offset" : 5,
      "end_offset" : 18,
      "type" : "ENGLISH",
      "position" : 2
    }
  ]
}
```

> 该例两种分词模式结果相同，可使用其他更长语句测试结果

#### 拓展词库

随着互联网发展，会不断涌现新词语，在原有的词汇列表中并不存在，所以词汇列表也需要不断更新。若拓展 IK 词库，只需要修改 ik 目录 config 目录中的 `IKAnalyzer.cfg.xml` 文件即可

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE properties SYSTEM "http://java.sun.com/dtd/properties.dtd">
<properties>
    <comment>IK Analyzer 扩展配置</comment>
    <!--用户可以在这里配置自己的扩展字典 -->
    <entry key="ext_dict">ext.dic</entry>
     <!--用户可以在这里配置自己的扩展停止词字典-->
    <entry key="ext_stopwords">stopwords.dic</entry>
    <!--用户可以在这里配置远程扩展字典 -->
    <!-- <entry key="remote_ext_dict">words_location</entry> -->
    <!--用户可以在这里配置远程扩展停止词字典-->
    <!-- <entry key="remote_ext_stopwords">words_location</entry> -->
</properties>
```

如上将拓展词放在 `./ext.dic`，禁止词放在 `./stopwords.dic`

禁止词可以放一些无意义的词，如 *的*、*啊* 等

配置好后重启 es

## DSL 索引库操作

索引库就类似数据库表，要向 es 中存储数据，必须先创建“库”和“表”

### mapping 映射属性

mapping 是对索引库中文档的约束，常见的 mapping 属性包括：

* type：字段数据类型，常见的简单类型有：
  * 字符串：text（可分词的文本）、keyword（精确值，例如：品牌、国家、ip地址）
  * 数值：long、integer、short、byte、double、float、
  * 布尔：boolean
  * 日期：date
  * 对象：object
* index：是否创建索引，默认为true
* analyzer：使用哪种分词器
* properties：该字段的子字段

### 创建索引库

* 请求方式：PUT
* 请求路径：/索引库名，可以自定义
* 请求参数：mapping 映射

```json
PUT /索引库名称
{
  "mappings": {
    "properties": {
      "字段名":{
        "type": "text",
        "analyzer": "ik_smart"
      },
      "字段名2":{
        "type": "keyword",
        "index": "false"
      },
      "字段名3":{
        "properties": {
          "子字段": {
            "type": "keyword"
          }
        }
      },
      // code
    }
  }
}
```

例如

```json
# 创建索引库
PUT /hello
{
  "mappings": {
    "properties": {
      "info": {
        "type": "text",
        "analyzer": "ik_smart"
      },
      "email": {
        "type": "keyword",
        "index": false
      },
      "name": {
        "properties": {
          "firstName": {
            "type": "keyword"
          },
          "lastName": {
            "type": "keyword"
          }
        }
      }
    }
  }
}
```

运行后返回类似即成功

```json
{
  "acknowledged" : true,
  "shards_acknowledged" : true,
  "index" : "hello"
}
```

### 查询索引库

* 请求方式：GET

* 请求路径：/索引库名

* 请求参数：无

格式

```json
GET /索引库名
```

例如

```json
# 查看索引库
GET /hello
```

结果

```json
{
  "hello" : {
    "aliases" : { },
    "mappings" : {
      "properties" : {
        "email" : {
          "type" : "keyword",
          "index" : false
        },
        "info" : {
          "type" : "text",
          "analyzer" : "ik_smart"
        },
        "name" : {
          "properties" : {
            "firstName" : {
              "type" : "keyword"
            },
            "lastName" : {
              "type" : "keyword"
            }
          }
        }
      }
    },
    "settings" : {
      "index" : {
        "routing" : {
          "allocation" : {
            "include" : {
              "_tier_preference" : "data_content"
            }
          }
        },
        "number_of_shards" : "1",
        "blocks" : {
          "read_only_allow_delete" : "true"
        },
        "provided_name" : "hello",
        "creation_date" : "1703683379263",
        "number_of_replicas" : "1",
        "uuid" : "zn-kPdsETZeFcB0nXK79hg",
        "version" : {
          "created" : "7120199"
        }
      }
    }
  }
}
```

### 修改索引库

索引库和 mapping 一旦创建，不允许修改，但可以添加字段

```json
PUT /索引库名/_mapping
{
  "properties": {
    "新字段名":{
      "type": "integer"
    }
  }
}
```

例如

```json
# 新加字段
PUT /hello/_mapping
{
  "properties": {
    "age": {
      "type": "integer",
      "index": false
    }
  }
}
```

---

如果遇到 `read-only-allow-delete` 类似错误，产生原因为磁盘剩余空间不足 5%，可用以下请求解决

```json
PUT _settings
{
  "index": {
    "blocks": {
      "read_only_allow_delete": "false"
    }
  }
}
```

### 删除索引库

* 请求方式：DELETE

* 请求路径：/索引库名

* 请求参数：无

格式

```json
DELETE /索引库名
```

例如

```json
DELETE /hello
```

结果

```json
{
  "acknowledged" : true
}
```

### 索引库操作总结

* 创建索引库：PUT /索引库名
* 查询索引库：GET /索引库名
* 删除索引库：DELETE /索引库名
* 添加字段：PUT /索引库名/_mapping

## DSL 文档操作

### 新增文档

```json
POST /索引库名/_doc/文档id
{
    "字段1": "值1",
    "字段2": "值2",
    "字段3": {
        "子属性1": "值3",
        "子属性2": "值4"
    },
    // code
}
```

例子

```json
# 添加文档
PUT /hello/_doc/1
{
  "info": "hello es",
  "email": "blog@yexca.net",
  "name": {
    "firstName": "yexca",
    "lastName": "Dale"
  }
}
```

结果

```json
{
  "_index" : "hello",
  "_type" : "_doc",
  "_id" : "1",
  "_version" : 1,
  "result" : "created",
  "_shards" : {
    "total" : 2,
    "successful" : 1,
    "failed" : 0
  },
  "_seq_no" : 0,
  "_primary_term" : 1
}
```

### 查询文档

```json
GET /{索引库名称}/_doc/{id}
```

例子

```json
# 查询文档
GEt /hello/_doc/1
```

结果

```json
{
  "_index" : "hello",
  "_type" : "_doc",
  "_id" : "1",
  "_version" : 1,
  "_seq_no" : 0,
  "_primary_term" : 1,
  "found" : true,
  "_source" : {
    "info" : "hello es",
    "email" : "blog@yexca.net",
    "name" : {
      "firstName" : "yexca",
      "lastName" : "Dale"
    }
  }
}
```

### 修改文档

修改有两种方式，全量修改与增量修改

#### 全量修改

全量修改是覆盖原来的文档，其本质是：

1. 根据指定的 id 删除文档
2. 新增一个相同 id 的文档

如果 id 不存在，也会执行第二步，也就从修改变成新增了 (覆盖写)

```json
PUT /{索引库名}/_doc/文档id
{
    "字段1": "值1",
    "字段2": "值2",
    // code
}
```

例如

```json
# 修改-全量修改
PUT /hello/_doc/1
{
  "info": "hello es",
  "email": "es@yexca.net",
  "name": {
    "firstName": "yexca",
    "lastName": "Dale"
  }
}
```

结果

```json
{
  "_index" : "hello",
  "_type" : "_doc",
  "_id" : "1",
  "_version" : 2,
  "result" : "updated",
  "_shards" : {
    "total" : 2,
    "successful" : 1,
    "failed" : 0
  },
  "_seq_no" : 1,
  "_primary_term" : 1
}
```

查询可发现邮箱已经修改

#### 增量修改

增量修改是只修改指定 id 匹配的文档中的部分字段

```json
POST /{索引库名}/_update/文档id
{
    "doc": {
         "字段名": "新的值",
    }
}
```

例如

```json
# 修改-增量修改
POST /hello/_update/1
{
  "doc": {
    "email": "blog@yexca.net"
  }
}
```

结果

```json
{
  "_index" : "hello",
  "_type" : "_doc",
  "_id" : "1",
  "_version" : 3,
  "result" : "updated",
  "_shards" : {
    "total" : 2,
    "successful" : 1,
    "failed" : 0
  },
  "_seq_no" : 2,
  "_primary_term" : 1
}
```

查询可发现邮箱已经修改

### 删除文档

```json
DELETE /{索引库名}/_doc/id值
```

例如

```json
# 删除文档
DELETE /hello/_doc/1
```

结果

```json
{
  "_index" : "hello",
  "_type" : "_doc",
  "_id" : "1",
    // 因为中间我修改了其他的，版本号变高了
  "_version" : 8,
  "result" : "deleted",
  "_shards" : {
    "total" : 2,
    "successful" : 1,
    "failed" : 0
  },
  "_seq_no" : 7,
  "_primary_term" : 1
}
```

### 文档操作总结

* 创建文档：POST /{索引库名}/_doc/文档id   { json文档 }
* 查询文档：GET /{索引库名}/_doc/文档id
* 删除文档：DELETE /{索引库名}/_doc/文档id
* 修改文档：
  * 全量修改：PUT /{索引库名}/_doc/文档id { json文档 }
  * 增量修改：POST /{索引库名}/_update/文档id { "doc": {字段}}
