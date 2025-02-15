---
slug: 235
title: 'Elasticsearch 集群'
# draft: true
author: yexca
date: '2025-02-15T17:17:08+09:00'
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
> | Elasticsearch 自动补全 | <https://blog.yexca.net/archives/232> |
> | Elasticsearch 数据同步 | <https://blog.yexca.net/archives/234> |
> | Elasticsearch 集群 | 本文 |

单机的 es 做数据存储必然面临两个问题：海量数据存储与单点故障问题

* 海量数据存储问题：将索引库从逻辑上拆分为 N 个分片 (shard)，存储到多个节点
* 单点故障问题：将分片数据在不同节点备份 (replica)

ES 集群相关概念

* 集群 (cluster)：一组拥有共同的 cluster name 的节点
* 节点 (node)：集群中的一个 ES 实例
* 分片 (shard)：索引可以被拆分为不同的部分进行存储，称为分片。在集群环境下，一个索引的不同分片可以拆分到不同的节点中

解决问题：数据量太大，单点存储量有限的问题

> 分片类似 Hadoop 的 HDFS 的数据分成多份备份

* 主分片 (Primary shard)：相对于副本分片的定义
* 副本分片 (Replica shard)：每个主分片可以有一个或者多个副本，数据和主分片一样

## 搭建 ES 集群

可以通过 docker-compose 来完成

```yml
version: '2.2'
services:
  es01:
    image: elasticsearch:7.12.1
    container_name: es01
    environment:
      - node.name=es01
      - cluster.name=es-docker-cluster
      - discovery.seed_hosts=es02,es03
      - cluster.initial_master_nodes=es01,es02,es03
      - "ES_JAVA_OPTS=-Xms512m -Xmx512m"
    volumes:
      - data01:/usr/share/elasticsearch/data
    ports:
      - 9200:9200
    networks:
      - elastic
  es02:
    image: elasticsearch:7.12.1
    container_name: es02
    environment:
      - node.name=es02
      - cluster.name=es-docker-cluster
      - discovery.seed_hosts=es01,es03
      - cluster.initial_master_nodes=es01,es02,es03
      - "ES_JAVA_OPTS=-Xms512m -Xmx512m"
    volumes:
      - data02:/usr/share/elasticsearch/data
    ports:
      - 9201:9200
    networks:
      - elastic
  es03:
    image: elasticsearch:7.12.1
    container_name: es03
    environment:
      - node.name=es03
      - cluster.name=es-docker-cluster
      - discovery.seed_hosts=es01,es02
      - cluster.initial_master_nodes=es01,es02,es03
      - "ES_JAVA_OPTS=-Xms512m -Xmx512m"
    volumes:
      - data03:/usr/share/elasticsearch/data
    networks:
      - elastic
    ports:
      - 9202:9200
volumes:
  data01:
    driver: local
  data02:
    driver: local
  data03:
    driver: local

networks:
  elastic:
    driver: bridge
```

es 运行需要修改一些 linux 系统权限，修改 `/etc/sysctl.conf` 文件

```sh
vi /etc/sysctl.conf
```

添加下面的内容：

```sh
vm.max_map_count=262144
```

然后执行命令，让配置生效：

```sh
sysctl -p
```

通过docker-compose启动集群：

```sh
docker-compose up -d
```

## 监控集群状态

kibana 可以监控 es 集群，不过需要依赖 es 的 x-pack 功能，配置比较复杂

可以使用 cerebro 来监控 es 集群，Github：<https://github.com/lmenezes/cerebro>

运行 `bin/cerebro.bat` 后访问 <http://localhost:9000> 即可进入管理界面

使用 cerebro 可以可视化创建索引库，以下是 DSL 语句创建

```json
PUT /indexName
{
  "settings": {
    "number_of_shards": 3, // 分片数量
    "number_of_replicas": 1 // 副本数量
  },
  "mappings": {
    "properties": {
      // mapping映射定义 ...
    }
  }
}
```

## 集群职责划分

ES 中集群节点有不同的职责划分

| 节点类型        | 配置参数                                       | 默认值 | 节点职责                                                     |
| --------------- | ---------------------------------------------- | ------ | ------------------------------------------------------------ |
| master eligible | node.master                                    | true   | 备选主节点：主节点可以管理和记录集群状态、决定分片在哪个节点、处理创建和删除索引库的请求 |
| data            | node.data                                      | true   | 数据节点：存储数据、搜索、聚合、CRUD                         |
| ingest          | node.ingest                                    | true   | 数据存储之前的预处理                                         |
| coordinating    | 上面 3 个参数都为 false 则为 coordinating 节点 | 无     | 路由请求到其它节点，合并其它节点处理的结果，返回给用户       |

默认情况下，集群的任何一个节点都同时具备上述四种角色

但真实的集群一定要将集群职责分离

* master 节点：对 CPU 要求高，但是内存要求低
* data 节点：对 CPU 和内存要求都高
* coordinating 节点：对网络带宽、CPU 要求高

职责分离可以让我们根据不同节点的需求分配不同的硬件去部署。而且避免业务之间的相互干扰

## 集群脑裂问题

脑裂是因为集群中的节点失联导致的

假设有三个节点，现在主节点 (node1) 与其他节点失联 (网络阻塞)，node2 和 node3 会认为 node1 宕机，就会重新选主。假设 node3 当选，集群会继续对外提供服务，node2 和 node3 自成集群，node1 自成集群，俩集群数据不同步，出现数据差异

当网络阻塞恢复正常，因为集群中有两个 master 节点，集群状态不一致，出现脑裂的情况

解决方案：要求选票超过 `(eligible 节点数量 + 1) / 2` 才能当选为 master 节点，因此 eligible 节点数量最好是奇数。对应的配置项为 `discovery.zen.minimum_master_nodes`，在 es7.0 以后，已经成为默认配置，因此一般不会发送脑裂问题

例如上述 3 个节点形成的集群，选票必须超过 (3+1)/2 = 2 票。node3 得到 node2 和 node3 的选票，当选为主，而 node1 只有自己的一票，没有当选。集群中依然只有 1 个主节点，没有出现脑裂

## 集群分布式存储

当新增文档时，应该保存到不同分片，保证数据均衡，那么 coordinating node 如何确定数据该存储到哪个分片呢

es 会通过 hash 算法来计算文档应该存储到哪个分片

公式：shard = hash(_routing) % number_of_shards

说明：

* _routing 默认是文档的 id
* 算法与分片数量有关，因此索引库一旦创建，分片数量不能修改

新增文档流程：

![image](https://raw.githubusercontent.com/yexca/picx-images-hosting/master/2024/01-SpringCloud/image-20210723225436084.5mtf1uqklgc0.webp)

流程：

1. 新增一个 id = 1 的文档
2. 对 id 做 hash 运算，假如得到的是 2，则对应存储到 shard-2
3. shard-2 的主分片在 node3 节点，将数据路由到 node3
4. 保存文档
5. 同步给 shard-2 的副本 replica-2，在 node2 节点
6. 返回结果给 coordinating-node 节点

## 集群分布式查询

es 的查询分为两个阶段：

* scatter phase：分散阶段，coordinating node 会把请求分发到每一个分片
* gather phase：聚集阶段，coordinating node 汇总 data node 的搜索结果，并处理为最终结果集返回给用户

## 集群故障转移

集群的 master 节点会监控集群中的节点状态，如果发现有节点宕机，会立即将宕机节点的分片数据迁移到其它节点，确保数据安全，这个叫做故障转移

假设一个集群三个节点，node1 是主节点

1. node1 发生了宕机
2. 需要重新选主，假设选中了 node2
3. node2 成为主节点后，会检测集群状态，发现 node1 的分片没有副本节点，需要将 node1 上的数据迁移到 node2、node3
