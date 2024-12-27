---
slug: 158
# layout: post
title: 'Hi SpringCloud'
author: yexca
date: 2024-02-25T17:56:54+08:00
# permalink: /archives/158
categories:
    - Back-end
    - Spring
tags:
    - SpringCloud
--- 

## 服务架构

### 单体架构

将业务的所有功能集中在一个项目中开发，打包一个包部署

优点：架构简单、扩展性差、部署成本低，适合小型项目

缺点：耦合度高

### 分布式架构

根据业务功能对系统进行拆分，每个业务模块作为独立项目开发，称为一个服务

优点：降低服务耦合、有利于服务升级拓展

缺点：架构复杂、难度大，适合大型互联网项目

### 微服务

微服务是一种经过良好架构设计的分布式架构方案，微服务架构特征：

* [单一职责](https://blog.yexca.net/archives/93#%E9%9D%A2%E5%90%91%E5%AF%B9%E8%B1%A1%E8%AE%BE%E8%AE%A1%E5%8E%9F%E5%88%99)：微服务拆分粒度更小，每一个服务都对应唯一的业务能力，做到单一职责，避免重复业务开发
* 面向服务：微服务对外暴露业务接口
* 自治：团队独立、技术独立、数据独立、部署独立
* 隔离性强：服务调用做好隔离、容错、降级，避免出现级联问题

## 微服务技术

微服务方案需要技术框架落地，常见的：

|                | Dubbo               | SpringCloud               | SpringCloudAlibaba        |
| -------------- | ------------------- | ------------------------- | ------------------------- |
| 注册中心       | zookeeper、 Redis   | Eureka、 Consul           | Nacos、 Eureka            |
| 服务远程调用   | Dubbo协议           | Feign (http协议)          | Dubbo、Feign              |
| 配置中心       | 无                  | SpringCloudConfig         | SpringCloudConfig、 Nacos |
| 服务网关       | 无                  | SpringCloudGateway、 Zuul | SpringCloudGateway、 Zuul |
| 服务监控和保护 | dubbo-admin，功能弱 | Hystix                    | Sentinel                  |

微服务需要根据业务模块拆分

* 单一职责：不同微服务，不要重复开发相同业务
* 数据独立：不要访问其它微服务的数据库
* 面向服务：将自己的业务暴露为接口，供其它微服务调用

## SpringCloud

SpringCloud 是目前使用最广泛的微服务框架。集成了各种微服务功能组件，并基于 SpringBoot 实现了这些组件的自动装配

官网：<https://spring.io/projects/spring-cloud/>

* 服务注册发现：Eureka、Nacos、Consul
* 统一配置管理：SpringCloudConfig、Nacos
* 服务远程调用：OpenFeign、Dubbo
* 统一网关路由： SpringCloudGateway、Zuul
* 服务链路监控：Zipkin、Sleuth
* 流控、降级、保护：Hystix、Sentinel

## 微服务调用

需求：根据订单id查询订单的同时，把订单所属的用户信息一起返回

### 注册 RestTemplate

```java
@Bean
public RestTemplate restTemplate(){
    return new RestTemplate();
}
```

### 服务远程调用 RestTemplate

```java
@Service
public class OrderService {

    @Autowired
    private OrderMapper orderMapper;
    @Autowired
    private RestTemplate restTemplate;

    public Order queryOrderById(Long orderId) {
        // 1.查询订单
        Order order = orderMapper.findById(orderId);
        // 2.查询用户
        String url = "http://localhost:8081/user/" + order.getUserId();
        	// RestTemplate的GET方法
        User user = restTemplate.getForObject(url, User.class);
        // 3.封装用户信息
        order.setUser(user);
        // 4.返回
        return order;
    }
}
```