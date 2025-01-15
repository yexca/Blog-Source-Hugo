---
slug: 271
title: 'SpringAMQP'
# draft: true
author: yexca
date: '2025-01-15T17:03:32+09:00'
categories:
    - 后端
    - Spring
tags:
    - SpringCloud
---

## 初识 MQ

### 同步调用

微服务间基于 Feign 的调用属于同步方式，存在一些问题

例如要开发一个支付服务，需要加入订单服务和仓储服务的代码，后期若要加入短信服务、积分服务等都需要修改支付代码，违反了[开放-封闭原则](https://blog.yexca.net/archives/93#%E9%9D%A2%E5%90%91%E5%AF%B9%E8%B1%A1%E8%AE%BE%E8%AE%A1%E5%8E%9F%E5%88%99)，并且在请求返回前无法做其他事情也会造成性能的浪费

问题：耦合度高、性能下降、资源浪费、级联失败 (若提供者出现问题，所有调用方也会跟着出问题，如同多米诺骨牌，迅速导致整个微服务群故障)

### 异步调用方案

异步调用常见实现就是事件驱动模式

用户支付请求 -> 支付服务 -> Broker，之后支付服务完成并响应，然后由 Broker 通知订单服务、仓储服务和短信服务

优点：服务解耦、性能提升，吞吐量提高、服务没有强依赖，故障隔离、流量削峰

缺点：依赖于 Broker 的可靠性、安全性、吞吐能力，架构复杂了，业务没有明显的流程线，不好追踪管理

### MQ

MessageQueue，消息队列，字面意思为存放消息的队列，也就是事件驱动架构中的 Broker

|            | **RabbitMQ**            | **ActiveMQ**                   | **RocketMQ** | **Kafka**  |
| ---------- | ----------------------- | ------------------------------ | ------------ | ---------- |
| 公司/社区  | Rabbit                  | Apache                         | 阿里         | Apache     |
| 开发语言   | Erlang                  | Java                           | Java         | Scala&Java |
| 协议支持   | AMQP，XMPP，SMTP，STOMP | OpenWire,STOMP，REST,XMPP,AMQP | 自定义协议   | 自定义协议 |
| 可用性     | 高                      | 一般                           | 高           | 高         |
| 单机吞吐量 | 一般                    | 差                             | 高           | 非常高     |
| 消息延迟   | 微秒级                  | 毫秒级                         | 毫秒级       | 毫秒以内   |
| 消息可靠性 | 高                      | 一般                           | 高           | 一般       |

追求可用性：Kafka、 RocketMQ 、RabbitMQ

追求可靠性：RabbitMQ、RocketMQ

追求吞吐能力：RocketMQ、Kafka

追求消息低延迟：RabbitMQ、Kafka

## 安装 RabbitMQ

可以从[官网](https://www.rabbitmq.com/download.html)看到多种安装方式，我使用 Docker 在线拉取

```bash
docker pull rabbitmq:3-management
```

运行命令

```bash
docker run \
 -e RABBITMQ_DEFAULT_USER=admin \
 -e RABBITMQ_DEFAULT_PASS=admin \
 --name mq \
 --hostname mq1 \
 -p 15672:15672 \
 -p 5672:5672 \
 -d \
 rabbitmq:3-management
```

访问 <localhost:15672> 即可打开管理，RabbitMQ 中的一些概念：

- channel：操作 MQ 的工具
- exchange：交换机，路由消息到队列中
- queue：队列，存储消息
- virtualHost：虚拟主机，是对 queue、exchange 等资源的逻辑分组

## 消息模型

在[官网提供了多种 Demo](https://www.rabbitmq.com/getstarted.html)，对应了不同的消息模型

- 基本消息队列 (BasicQueue)：["Hello World!"](https://www.rabbitmq.com/tutorials/tutorial-one-python.html)
- 工作消息队列 (WorkQueue)：[Work Queues](https://www.rabbitmq.com/tutorials/tutorial-two-python.html)
- 发布订阅模型
  - Fanout Exchange：广播 [Publish/Subscribe](https://www.rabbitmq.com/tutorials/tutorial-three-python.html)
  - Direct Exchange：路由 [Routing](https://www.rabbitmq.com/tutorials/tutorial-four-python.html)
  - Topic Exchange：主题 [Topics](https://www.rabbitmq.com/tutorials/tutorial-five-python.html)

### Hello World

Publisher -> Queue -> Consumer

- publisher：消息发布者，将消息发送到队列 queue
- queue：消息队列，负责接受并缓存消息
- consumer：订阅队列，处理队列中的消息

发布者

```java
public class PublisherTest {
    @Test
    public void testSendMessage() throws IOException, TimeoutException {
        // 1.建立连接
        ConnectionFactory factory = new ConnectionFactory();
        // 1.1.设置连接参数，分别是：主机名、端口号、vhost、用户名、密码
        factory.setHost("localhost");
        factory.setPort(5672);
        factory.setVirtualHost("/");
        factory.setUsername("admin");
        factory.setPassword("admin");
        // 1.2.建立连接
        Connection connection = factory.newConnection();

        // 2.创建通道Channel
        Channel channel = connection.createChannel();

        // 3.创建队列
        String queueName = "hello.queue";
        channel.queueDeclare(queueName, false, false, false, null);

        // 4.发送消息
        String message = "hello, rabbitmq!";
        channel.basicPublish("", queueName, null, message.getBytes());
        System.out.println("发送消息成功：【" + message + "】");

        // 5.关闭通道和连接
        channel.close();
        connection.close();

    }
}
```

接收者

```java
public class ConsumerTest {
    public static void main(String[] args) throws IOException, TimeoutException {
        // 1.建立连接
        ConnectionFactory factory = new ConnectionFactory();
        // 1.1.设置连接参数，分别是：主机名、端口号、vhost、用户名、密码
        factory.setHost("localhost");
        factory.setPort(5672);
        factory.setVirtualHost("/");
        factory.setUsername("admin");
        factory.setPassword("admin");
        // 1.2.建立连接
        Connection connection = factory.newConnection();

        // 2.创建通道Channel
        Channel channel = connection.createChannel();

        // 3.创建队列
        String queueName = "hello.queue";
        channel.queueDeclare(queueName, false, false, false, null);

        // 4.订阅消息
        channel.basicConsume(queueName, true, new DefaultConsumer(channel){
            @Override
            public void handleDelivery(String consumerTag, Envelope envelope,
                                       AMQP.BasicProperties properties, byte[] body) throws IOException {
                // 5.处理消息
                String message = new String(body);
                System.out.println("接收到消息：【" + message + "】");
            }
        });
        System.out.println("等待接收消息。。。。");
    }
}
```

控制台输出：

```markdown
等待接收消息。。。。
接收到消息：【hello, rabbitmq!】
```

显然这种方式略显繁琐

## SpringAMQP

[SpringAMQP](https://spring.io/projects/spring-amqp) 是基于 RabbitMQ 封装的一套模板，并且还利用 SpringBoot 对其实现了自动装配，使用起来非常方便

- AMQP

Advanced Message Queuing Protocol，是用于在应用程序之间传递业务消息的开放标准。该协议与语言和平台无关，更符合微服务中独立性的要求

- Spring AMQP

Spring AMQP 是基于 AMQP 协议定义的一套 API 规范，提供了模板来发送和接收消息。包含两部分，其中 spring-amqp 是基础抽象，spring-rabbit 是底层的默认实现

它可以自动声明队列、交换机及其绑定关系，基于注解的监听器模式，异步接收消息

### Basic Queue 简单队列模型

首先在父工程中引入依赖

```xml
<!--AMQP依赖，包含RabbitMQ-->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-amqp</artifactId>
</dependency>
```

#### publisher 消息发送

配置 application.yml

```yml
spring:
  rabbitmq:
    host: localhost # 主机名
    port: 5672 # 端口
    virtual-host: / # 虚拟主机
    username: admin # 用户名
    password: admin # 密码
```

利用 RabbitTemplate 实现消息发送

```java
@RunWith(SpringRunner.class)
@SpringBootTest
public class SpringamqpTest {
    @Autowired
    private RabbitTemplate rabbitTemplate;
    @Test
    public void testSimpleQueue(){
        // 队列名
        String queueName = "hello.queue";
        // 消息
        String msg = "Hello Spring ampq";
        // 发送
        rabbitTemplate.convertAndSend(queueName,msg);
    }
}
```

#### consumer 消息接收

配置 application.yml 同上

```yml
spring:
  rabbitmq:
    host: localhost # 主机名
    port: 5672 # 端口
    virtual-host: / # 虚拟主机
    username: admin # 用户名
    password: admin # 密码
```

创建一个新类 SpringRabbitListener

```java
@Component
public class SpringRabbitListener {
    @RabbitListener(queues = "hello.queue")
    public void listenSimpleQueue(String msg){
        System.out.println("接收的消息为：" + msg);
    }
}
```

### WorkQueue 工作消息队列

也称 TaskQueue，任务模型，可以提高消息处理速度，避免队列消息堆积

publisher -> queue -> consumer1 and consumer2 and ...

#### publisher 消息发送

定义一个方法，每秒发送 50 条消息

```java
public class SpringamqpTest {
    @Test
    public void testWorkQueue() throws InterruptedException {
        String queueName = "hello.queue";
        String msg = "Hello Spring ampq...";
        for (int i = 0; i < 50; i++) {
            rabbitTemplate.convertAndSend(queueName, msg + i);
            // 睡眠20毫秒，1秒发50条消息
            Thread.sleep(20);
        }
    }
}
```

#### consumer 消息接收

创建两个消费者绑定同一队列

```java
@Component
public class SpringRabbitListener {
    @RabbitListener(queues = "hello.queue")
    public void listenSimpleQueue1(String msg) throws InterruptedException {
        System.out.println("1接收的消息为：" + msg);
        // 每秒处理40条消息
        Thread.sleep(25);
    }

    @RabbitListener(queues = "hello.queue")
    public void listenSimpleQueue2(String msg) throws InterruptedException {
        // 使用err输出红色消息
        System.err.println("2接收的消息为：" + msg);
        // 每秒处理5条消息
        Thread.sleep(200);
    }
}
```

#### 测试

先运行接收者，而后运行发送者发送消息

从输出结果可以看到，俩接收者各接收一半的消息，也就是说消息是平均分配给每个消费者，并没有考虑到消费者的处理能力。这样显然是有问题的

#### prefetch

修改 application.yml 文件，设置 prefetch 这个值，可以控制预取消息的上限 (默认无限)

```yml
spring:
  rabbitmq:
    host: localhost # 主机名
    port: 5672 # 端口
    virtual-host: / # 虚拟主机
    username: admin # 用户名
    password: admin # 密码
    listener:
      simple:
        # 每次只能获取一条消息，处理完成才能获取下一个消息
        prefetch: 1
```

再次测试可以发现执行效率提高

### 发布订阅模型

发布订阅模式加入了交换机，允许将同一个消息发送给全部接收者

publisher -> exchange -> queue1 and queue2
queue1 -> consumer1 and consumer2
queue2 -> consumer3

常见的 exchange 有

- Fanout：广播
- Direct：路由
- Topic：话题

> exchange 负责路由，并不存储，一旦路由失败则消息丢失

#### Fanout(扇出) 广播

Fanout exchange 会将接收到的消息广播到每一个和其绑定的 queue

在接收者创建一个配置类，声明队列与交换机

```java
@Configuration
public class FanoutConfig {
    /**
     * 声明交换机
     * @return
     */
    @Bean
    public FanoutExchange fanoutExchange(){
        return new FanoutExchange("hello.fanout");
    }

    /**
     * 第一个队列
     * @return
     */
    @Bean
    public Queue fanoutQueue1(){
        return new Queue("fanout.queue1");
    }

    /**
     * 绑定第一个队列与交换机
     * @param fanoutQueue1
     * @param fanoutExchange
     * @return
     */
    @Bean
    public Binding bindingQueue1(Queue fanoutQueue1, FanoutExchange fanoutExchange){
        return BindingBuilder.bind(fanoutQueue1).to(fanoutExchange);
    }

    /**
     * 第二个队列
     * @return
     */
    @Bean
    public Queue fanoutQueue2(){
        return new Queue("fanout.queue2");
    }

    /**
     * 绑定第二个队列与交换机
     * @param fanoutQueue2
     * @param fanoutExchange
     * @return
     */
    @Bean
    public Binding bindingQueue2(Queue fanoutQueue2, FanoutExchange fanoutExchange){
        return BindingBuilder.bind(fanoutQueue2).to(fanoutExchange);
    }
}
```

消息发送

```java
public class SpringamqpTest {    
    @Test
    public void testFanoutExchange(){
        // 交换机
        String exchangeName = "hello.fanout";
        // 消息
        String msg = "hello, everyone";
        rabbitTemplate.convertAndSend(exchangeName,"",msg);
    }
}
```

其中中间空着的 `routingkey` 在下两个模型使用

消息的接收

```java
@Component
public class SpringRabbitListener {
    @RabbitListener(queues = "fanout.queue1")
    public void listenFanoutQueue1(String msg){
        System.out.println("Fanout1 接收消息：" + msg);
    }
    @RabbitListener(queues = "fanout.queue2")
    public void listenFanoutQueue2(String msg){
        System.out.println("Fanout2 接收消息：" + msg);
    }
}
```

#### Direct 路由

Direct Exchange 会将接收到的消息根据规则路由到指定的 Queue，因此称为路由模式

队列与交换机的绑定要指定一个 `Routingkey`，发送方发消息时也必须指定消息的 `Routingkey`，只有队列的 `Routingkey` 和消息的 `Routingkey` 完全一致，才会接收到消息

在此使用基于注解声明队列和交换机，不需要配置类

```java
@Component
public class SpringRabbitListener {
    @RabbitListener(bindings = @QueueBinding(
            value = @Queue(name = "direct.queue1"),
//            exchange = @Exchange(name = "hello.direct", type = ExchangeTypes.DIRECT),
//            因为默认为Direct类型，可以不用指定
            exchange = @Exchange(name = "hello.direct"),
            key = {"red","warma"}
    ))
    public void listenDirectQueue1(String msg){
        System.out.println("1 接收消息：" + msg);
    }

    @RabbitListener(bindings = @QueueBinding(
            value = @Queue(name = "direct.queue2"),
            exchange = @Exchange(name = "hello.direct"),
            key = {"red","aqua"}
    ))
    public void listenDirectQueue2(String msg){
        System.out.println("2 接收消息：" + msg);
    }
}
```

发送者

```java
public class SpringamqpTest {    
    @Test
    public void testDirectExchange(){
        // 交换机
        String exchangeName = "hello.direct";
        // 消息
        String msg = "hello, aqua";
        rabbitTemplate.convertAndSend(exchangeName,"aqua",msg);
    }
}
```

上例只有接收者 2 可以接收到消息，若 routingkey 为 red 则两个都能接收到消息

#### Topic 话题

TopicExchange 与 DirectExchange 类似，区别在于 routingKey 必须是多个单词的列表，并且以 `.` 分割。

Queue 与Exchange 指定 BindingKey 时可以使用通配符

- `#`：匹配一个或多个单词
- `*`：只匹配一个单词

接收者

```java
@Component
public class SpringRabbitListener {
    @RabbitListener(bindings = @QueueBinding(
            value = @Queue("topic.queue1"),
            exchange = @Exchange(name = "hello.topic", type = ExchangeTypes.TOPIC),
            key = "#.news"
    ))
    public void listenTopicQueue1(String msg){
        System.out.println("1 接受消息：" + msg);
    }
    @RabbitListener(bindings = @QueueBinding(
            value = @Queue("topic.queue2"),
            exchange = @Exchange(name = "hello.topic", type = ExchangeTypes.TOPIC),
            key = "china.#"
    ))
    public void listenTopicQueue2(String msg){
        System.out.println("2 接受消息：" + msg);
    }
}
```

消息发送

```java
public class SpringamqpTest {    
    @Test
    public void testTopicExchange(){
        // 交换机
        String exchangeName = "hello.topic";
        // 消息
        String msg = "news for China";
        rabbitTemplate.convertAndSend(exchangeName,"china.news",msg);
    }
}
```

上例 1 和 2 都可以接收

```java
public class SpringamqpTest {    
    @Test
    public void testTopicExchange(){
        // 交换机
        String exchangeName = "hello.topic";
        // 消息
        String msg = "news for Japan";
        rabbitTemplate.convertAndSend(exchangeName,"japan.news",msg);
    }
}
```

只有 1 可以接收

## 消息转换器

Spring 会把发送的消息序列化为字节发送给 MQ，接收消息时，把字节反序列化为 Java 对象，只不过默认情况下 Spring 采用的序列化方式是 JDK 序列化，其数据体积过大、有安全漏洞、可读性差

可以使用 JSON 方式来做序列化和反序列化

首先在父工程引入依赖

```xml
<!-- JSON转换器 -->
<dependency>
    <groupId>com.fasterxml.jackson.dataformat</groupId>
    <artifactId>jackson-dataformat-xml</artifactId>
    <version>2.9.10</version>
</dependency>
```

在消费者与接收者声明一个 bean 即可

```java
@Bean
public MessageConverter jsonMessageConverter(){
    return new Jackson2JsonMessageConverter();
}
```
