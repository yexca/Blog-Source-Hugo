---
slug: 234
title: 'Elasticsearch 数据同步'
# draft: true
author: yexca
date: '2025-02-14T20:36:55+09:00'
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
> | Elasticsearch 数据聚合 | <https://blog.yexca.net/archives/231> |
> | Elasticsearch 自动补全 | <https://blog.yexca.net/archives/232> |
> | Elasticsearch 数据同步 | 本文 |
> | Elasticsearch 集群 | <https://blog.yexca.net/archives/235> |

es 的数据来自 MySQL 数据库，因此 MySQL 数据发送改变时，es 也必须跟着改变，此为 es 与 MySQL 之间的数据同步

常见的数据同步方案有三种：

* 同步调用
* 异步通知
* 监听 binlog

## 同步调用

![image](https://raw.githubusercontent.com/yexca/picx-images-hosting/master/2024/01-SpringCloud/image.1lj3wtigvg1s.webp)

步骤：

* hotel-demo 对外提供接口，用于修改 es 中的数据
* 后台管理系统 (hotel-admin) 在完成数据库操作后，之间调用 hotel-demo 提供的接口

## 异步通知

![image](https://raw.githubusercontent.com/yexca/picx-images-hosting/master/2024/01-SpringCloud/image.7639v9kp7l0.webp)

步骤：

* hotel-admin 对 mysql 数据完成 CRUD 后发送 MQ 消息
* hotel-demo 监听 MQ，接收消息后完成对 es 数据修改

## 监听 binlog

![image](https://raw.githubusercontent.com/yexca/picx-images-hosting/master/2024/01-SpringCloud/image.6t9hbchmlm00.webp)

流程：

* 开启 MySQL 的 binlog 功能
* MySQL 完成 CRUD 操作都会记录在 binlog 中
* hotel-demo 基于 canal 监听 binlog 变化，实时更新 es 中的内容

## 方案对比

方式一：同步调用

* 优点：实现简单，粗暴
* 缺点：业务耦合度高

方式二：异步通知

* 优点：低耦合，实现难度一般
* 缺点：依赖 mq 的可靠性

方式三：监听binlog

* 优点：完全解除服务间耦合
* 缺点：开启 binlog 增加数据库负担、实现复杂度高

## 数据同步实现

说明：使用 MQ 异步通知，在 hotel-admin 实现对 MySQL 的 CRUD 操作

在 hotel-admin、hotel-demo 引入 rabbitmq 依赖

```xml
<!--amqp-->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-amqp</artifactId>
</dependency>
```

声明队列、交换机名称

```java
public class MqConstants {
    // 交换机
    public final static String HOTEL_EXCHANGE = "hotel.topic";
    // 监听新增和修改队列
    public final static String HOTEL_INSERT_QUEUE = "hotel.insert.queue";
    // 监听删除的队列
    public final static String HOTEL_DELETE_QUEUE = "hotel.delete.queue";
    // 新增或修改的RoutingKey
    public final static String HOTEL_INSERT_KEY = "hotel.insert";
    // 删除的RoutingKey
    public final static String HOTEL_DELETE_KEY = "hotel.delete";
}
```

在 hotel-demo 声明交换机配置

```java
@Configuration
public class MqConfig {
    @Bean
    public TopicExchange topicExchange(){
        return new TopicExchange(MqConstants.HOTEL_EXCHANGE,true,false);
    }
    @Bean
    public Queue insertQueue(){
        return new Queue(MqConstants.HOTEL_INSERT_QUEUE,true);
    }
    @Bean
    public Queue deleteQueue(){
        return new Queue(MqConstants.HOTEL_DELETE_QUEUE, true);
    }
    @Bean
    public Binding insertQueueBinding(){
        return BindingBuilder.bind(insertQueue()).to(topicExchange()).with(MqConstants.HOTEL_INSERT_KEY);
    }
    @Bean
    public Binding deleteQueueBinding(){
        return BindingBuilder.bind(deleteQueue()).to(topicExchange()).with(MqConstants.HOTEL_DELETE_KEY);
    }
}
```

在 hotel-admin 发送 MQ 消息

```java
public class HotelController {
    @Autowired
    private RabbitTemplate rabbitTemplate;
    
    @PostMapping
    public void saveHotel(@RequestBody Hotel hotel){
        hotelService.save(hotel);
        rabbitTemplate.convertAndSend(MqConstants.HOTEL_EXCHANGE, MqConstants.HOTEL_INSERT_KEY, hotel.getId());
    }

    @PutMapping()
    public void updateById(@RequestBody Hotel hotel){
        if (hotel.getId() == null) {
            throw new InvalidParameterException("id不能为空");
        }
        hotelService.updateById(hotel);
        rabbitTemplate.convertAndSend(MqConstants.HOTEL_EXCHANGE, MqConstants.HOTEL_INSERT_KEY,hotel.getId());
    }

    @DeleteMapping("/{id}")
    public void deleteById(@PathVariable("id") Long id) {
        hotelService.removeById(id);
        rabbitTemplate.convertAndSend(MqConstants.HOTEL_EXCHANGE, MqConstants.HOTEL_DELETE_KEY, id);
    }
}
```

在 hotel-demo 接收 MQ 消息，业务逻辑：

* 新增消息：根据传递的 hotel.id 查询信息，然后新增一条数据到索引库
* 删除消息：根据传递的 hotel.id 删除索引库中的一条数据

```java
@Override
public void deleteById(Long id) {
    try {
        // 准备request
        DeleteRequest request = new DeleteRequest("hotel", id.toString());
        // 发送请求
        client.delete(request, RequestOptions.DEFAULT);
    } catch (IOException e) {
        throw new RuntimeException(e);
    }
}

@Override
public void insertById(Long id) {
    try {
        // 根据id查询数据
        Hotel hotel = getById(id);
        // 转换文档类型
        HotelDoc hotelDoc = new HotelDoc(hotel);

        // 准备request
        IndexRequest request = new IndexRequest("hotel").id(id.toString());
        // 准备json
        request.source(JSON.toJSONString(hotelDoc), XContentType.JSON);
        // 发送请求
        client.index(request, RequestOptions.DEFAULT);
    } catch (IOException e) {
        throw new RuntimeException(e);
    }
}
```

在 hotel-demo 编写监听器

```java
@Component
public class HotelLinster {
    @Autowired
    private IHotelService hotelService;

    /**
     * 监听修改或新增
     * @param id
     */
    @RabbitListener(queues = MqConstants.HOTEL_INSERT_QUEUE)
    public void listenHotelInsertOrUpdate(Long id){
        hotelService.insertById(id);
    }

    /**
     * 监听删除
     * @param id
     */
    @RabbitListener(queues = MqConstants.HOTEL_DELETE_QUEUE)
    public void listenHotelDelete(Long id){
        hotelService.deleteById(id);
    }
}
```
