---
title: rabbitmq
date: 2020-03-21 22:44:46
tags: mq rabbitmq
---

# 什么是mq
消息队列，又叫做消息中间件。是指用高效可靠的消息传递机制进行与平台无关的 数据交流，并基于数据通信来进行分布式系统的集成。通过提供消息传递和消息队列模 型，可以在分布式环境下扩展进程的通信(维基百科)。

![](/img/rabbitmq/mq_jianjie.jpg)

## 为什么要用mq
1，实现异步通信。
2，实现系统解耦。
3，实现流量削峰。

总结起来:
1) 对于数据量大或者处理耗时长的操作，我们可以引入 MQ 实现异步通信，减少客户端的等待，提升响应速度。
2) 对于改动影响大的系统之间，可以引入 MQ 实现解耦，减少系统之间的直接依赖。
3) 对于会出现瞬间的流量峰值的系统，我们可以引入 MQ 实现流量削峰，达到保护应用和数据库的目的。

# RabbitMQ 简介

## 基本特性
官网 https://www.rabbitmq.com/getstarted.html


- 高可靠:RabbitMQ 提供了多种多样的特性让你在可靠性和性能之间做出权衡，包 括持久化、发送应答、发布确认以及高可用性。
- 灵活的路由:通过交换机(Exchange)实现消息的灵活路由。
- 支持多客户端:对主流开发语言(Python、Java、Ruby、PHP、C#、JavaScript、 Go、Elixir、Objective-C、Swift 等)都有客户端实现。
- 集群与扩展性:多个节点组成一个逻辑的服务器，支持负载。 
- 高可用队列:通过镜像队列实现队列中数据的复制。 
- 权限管理:通过用户与虚拟机实现权限管理。
- 插件系统:支持各种丰富的插件扩展，同时也支持自定义插件。 与 Spring 集成:Spring 对 AMQP 进行了封装。

## AMQP 协议

AMQP:高级消息队列协议，是一个工作于应用层的协议，最新的版本是 1.0 版本。
除了 RabbitMQ 之外，AMQP 的实现还有 OpenAMQ、Apache Qpid、Redhat Enterprise MRG、AMQP Infrastructure、ØMQ、Zyre。

除了 AMQP 之外，RabbitMQ 支持多种协议，STOMP、MQTT、HTTP and WebSockets。
可以使用 WireShark 等工具对 RabbitMQ 通信的 AMQP 协议进行抓包。


## 工作模型
由于 RabbitMQ 实现了 AMQP 协议，所以 RabbitMQ 的工作模型也是基于 AMQP 的。

![](/img/rabbitmq/rabbitmq_workmodel.jpg)


### Broker
我们要使用 RabbitMQ 来收发消息，必须要安装一个 RabbitMQ 的服务，可以安装 在 Windows 上面也可以安装在 Linux 上面，默认是 5672 的端口。这台 RabbitMQ 的 服务器我们把它叫做 Broker，中文翻译是代理/中介，因为 MQ 服务器帮助我们做的事 情就是存储、转发消息。


### Connection
无论是生产者发送消息，还是消费者接收消息，都必须要跟 Broker 之间建立一个连接，这个连接是一个 TCP 的长连接。

### Channel
如果所有的生产者发送消息和消费者接收消息，都直接创建和释放 TCP 长连接的话， 对于 Broker 来说肯定会造成很大的性能损耗，因为 TCP 连接是非常宝贵的资源，创建和 释放也要消耗时间。所以在 AMQP 里面引入了 Channel 的概念，它是一个虚拟的连接。我们把它翻译成通道，或者消息信道。这样我们就可以在保持的 TCP 长连接里面去创建和释放 Channel，大大了减少了资源消耗。另外一个需要注意的是，Channel 是 RabbitMQ 原 生 API 里面的最重要的编程接口，也就是说我们定义交换机、队列、绑定关系，发送消 息消费消息，调用的都是 Channel 接口上的方法。
https://stackoverflow.com/questions/18418936/rabbitmq-and-relationship- between-channel-and-connection

### Queue
现在我们已经连到 Broker 了，可以收发消息了。在其他一些 MQ 里面，比如 ActiveMQ 和 Kafka，我们的消息都是发送到队列上的。

队列是真正用来存储消息的，是一个独立运行的进程，有自己的数据库(Mnesia)。

消费者获取消息有两种模式，一种是 Push 模式，只要生产者发到服务器，就马上推 送给消费者。另一种是 Pull 模式，消息存放在服务端，只有消费者主动获取才能拿到消 息。消费者需要写一个 while 循环不断地从队列获取消息吗?不需要，我们可以基于事 件机制，实现消费者对队列的监听。

由于队列有 FIFO 的特性，只有确定前一条消息被消费者接收之后，才会把这条消息 从数据库删除，继续投递下一条消息。

### Excahnge
在 RabbitMQ 里面永远不会出现消息直接发送到队列的情况。因为在 AMQP 里面 引入了交换机(Exchange)的概念，用来实现消息的灵活路由。

交换机是一个绑定列表，用来查找匹配的绑定关系。

队列使用绑定键(Binding Key)跟交换机建立绑定关系。 生产者发送的消息需要携带路由键(Routing Key)，交换机收到消息时会根据它保存的绑定列表，决定将消息路由到哪些与它绑定的队列上。

注意:交换机与队列、队列与消费者都是多对多的关系。


### Vhost

我们每个需要实现基于 RabbitMQ 的异步通信的系统，都需要在服务器上创建自己要用到的交换机、队列和它们的绑定关系。如果某个业务系统不想跟别人混用一个系统， 怎么办?再采购一台硬件服务器单独安装一个 RabbitMQ 服务?这种方式成本太高了。 在同一个硬件服务器上安装多个 RabbitMQ 的服务呢?比如再运行一个 5673 的端口? 没有必要，因为 RabbitMQ 提供了虚拟主机 VHOST。


VHOST 除了可以提高硬件资源的利用率之外，还可以实现资源的隔离和权限的控 制。它的作用类似于编程语言中的 namespace 和 package，不同的 VHOST 中可以有 同名的 Exchange 和 Queue，它们是完全透明的。

这个时候，我们可以为不同的业务系统创建不同的用户(User)，然后给这些用户 分配 VHOST 的权限。比如给风控系统的用户分配风控系统的 VHOST 的权限，这个用户 可以访问里面的交换机和队列。给超级管理员分配所有 VHOST 的权限。


## 路由方式
RabbitMQ 引入 Exchange 是为了实现消息的灵活路由，到底有哪些路由方式?

### 直连 Direct
队列与直连类型的交换机绑定，需指定一个精确的绑定键。

生产者发送消息时会携带一个路由键。只有当路由键与其中的某个绑定键完全匹配 时，这条消息才会从交换机路由到满足路由关系的此队列上。

![](/img/rabbitmq/rabbitmq_router_direct.jpg)

### 主题 Topic
队列与主题类型的交换机绑定时，可以在绑定键中使用通配符。
两个通配符:
```
#  0个或者多个单词
*  不多不少一个单词
```
单词(word)指的是用英文的点“.”隔开的字符。例如 abc.def 是两个单词。

![](/img/rabbitmq/rabbitmq_router_topic.jpg)

解读:</br>
第一个队列支持路由键以 junior 开头的消息路由，后面可以有单词，也可以 没有。</br>
第二个队列支持路由键以 netty 开头，并且后面是一个单词的消息路由。</br>
第三个队列支持路由键以 jvm 结尾，并且前面是一个单词的消息路由。

### 广播 Fanout
主题类型的交换机与队列绑定时，不需要指定绑定键。因此生产者发送消息到广播类型的交换机上，也不需要携带路由键。消息达到交换机时，所有与之绑定了的队列， 都会收到相同的消息的副本。

![](/img/rabbitmq/rabbitmq_router_direct.jpg)

例如:
channel.basicPublish("MY_FANOUT_EXCHANGE", "", "msg 4"); 收到 msg 4。


# RabbitMQ 进阶知识

## 2.1. TTL(Time To Live)

有两种设置方式:

1) 通过队列属性设置消息过期时间 所有队列中的消息超过时间未被消费时，都会过期。
2) 设置单条消息的过期时间 在发送消息的时候指定消息属性。

如果同时指定了 Message TTL 和 Queue TTL，则小的那个时间生效。

## 死信队列

消息在某些情况下会变成死信(Dead Letter)。
队列在创建的时候可以指定一个死信交换机 DLX(Dead Letter Exchange)。 死信交换机绑定的队列被称为死信队列 DLQ(Dead Letter Queue)，DLX 实际上 也是普通的交换机，DLQ 也是普通的队列(例如替补球员也是普通球员)。

![](/img/rabbitmq/rabbitmq_addqueue_deadletter.jpg)

> 什么情况下消息会变成死信?
1. 消息被消费者拒绝并且未设置重回队列:(NACK || Reject ) && requeue == false
2. 消息过期
3. 队列达到最大长度，超过了 Max length(消息数)或者 Max length bytes (字节数)，最先入队的消息会被发送到 DLX。

> 死信队列如何使用?
1、声明原交换机(GP_ORI_USE_EXCHANGE)、原队列(GP_ORI_USE_QUEUE)，相互绑定。队列中的消息 10 秒钟过期，因为没有消费者，会变成死信。指定原队列的死信交换机(GP_DEAD_LETTER_EXCHANGE)。

``` java
@Bean("oriUseExchange")
public DirectExchange exchange() {
    return new DirectExchange("GP_ORI_USE_EXCHANGE", true, false, new HashMap<>());
}
​
@Bean("oriUseQueue") public Queue queue() {
    Map<String, Object> map = new HashMap<String, Object>();
    map.put("x-message-ttl", 10000); // 10 秒钟后成为死信
    map.put("x-dead-letter-exchange", "GP_DEAD_LETTER_EXCHANGE"); // 队列中的消息变成死信后，进入死信交换机
    return new Queue("GP_ORI_USE_QUEUE", true, false, false, map);
}
​
@Bean
public Binding binding(@Qualifier("oriUseQueue") Queue queue,@Qualifier("oriUseExchange") DirectExchange exchange) {
    return BindingBuilder.bind(queue).to(exchange).with("gupao.ori.use");
}
```

2 、 声 明 死 信 交 换 机 ( GP_DEAD_LETTER_EXCHANGE ) 、 死 信 队 列 (GP_DEAD_LETTER_QUEUE)，相互绑定

```  java
@Bean("deatLetterExchange")
public TopicExchange deadLetterExchange() {
    return new TopicExchange("GP_DEAD_LETTER_EXCHANGE", true, false, new HashMap<>());
}
@Bean("deatLetterQueue")
public Queue deadLetterQueue() {
    return new Queue("GP_DEAD_LETTER_QUEUE", true, false, false, new HashMap<>());
}
@Bean
public Binding bindingDead(@Qualifier("deatLetterQueue") Queue queue,@Qualifier("deatLetterExchange") TopicExchange exchange) {
    return BindingBuilder.bind(queue).to(exchange).with("#");// 无条件路由 
}

```
3、最终消费者监听死信队列。 
4、生产者发送消息。

![](/img/rabbitmq/rabbitmq_addqueue_deadletter.jpg)

## 延迟队列
我们在实际业务中有一些需要延时发送消息的场景，例如:</br>
1、 家里有一台智能热水器，需要在 30 分钟后启动</br>
2、 未付款的订单，15 分钟后关闭</br>

RabbitMQ 本身不支持延迟队列，总的来说有三种实现方案:</br>
1、 先存储到数据库，用定时任务扫描</br>
2、 利用 RabbitMQ 的死信队列(Dead Letter Queue)实现</br>
3、 利用 rabbitmq-delayed-message-exchange 插件</br>


## 服务端流控 (Flow Control)
当 RabbitMQ 生产 MQ 消息的速度远大于消费消息的速度时，会产生大量的消息堆积，占用系统资源，导致机器的性能下降。我们想要控制服务端接收的消息的数量，应该怎么做呢?</br>
队列有两个控制长度的属性:
- x-max-length:队列中最大存储最大消息数，超过这个数量，队头的消息会被丢弃。
- x-max-length-bytes:队列中存储的最大消息容量(单位 bytes)，超过这个容量，队头的消息会被丢弃。
![](/img/rabbitmq/rabbitmq_server_flow_control.jpg)

需要注意的是，设置队列长度只在消息堆积的情况下有意义，而且会删除先入队的 消息，不能真正地实现服务端限流。

### 内存控制

RabbitMQ 会在启动时检测机器的物理内存数值。默认当 MQ 占用 40% 以上内 存时，MQ 会主动抛出一个内存警告并阻塞所有连接(Connections)。可以通过修改 rabbitmq.config 文件来调整内存阈值，默认值是 0.4，如下所示: 
```
[{rabbit, [{vm_memory_high_watermark, 0.4}]}].
```
也可以用命令动态设置，如果设置成 0，则所有的消息都不能发布。
```
rabbitmqctl set_vm_memory_high_watermark 0.3
```

### 磁盘控制
另一种方式是通过磁盘来控制消息的发布。当磁盘空间低于指定的值时(默认 50MB)，触发流控措施。
例如:指定为磁盘的 30%或者 2GB:
https://www.rabbitmq.com/configure.html
```
disk_free_limit.relative = 3.0 
disk_free_limit.absolute = 2GB
```

## 消费端限流
https://www.rabbitmq.com/consumer-prefetch.html</br>
默认情况下，如果不进行配置，RabbitMQ 会尽可能快速地把队列中的消息发送到 消费者。因为消费者会在本地缓存消息，如果消息数量过多，可能会导致 OOM 或者影 响其他进程的正常运行。</br>
在消费者处理消息的能力有限，例如消费者数量太少，或者单条消息的处理时间过长的情况下，如果我们希望在一定数量的消息消费完之前，不再推送消息过来，就要用到消费端的流量限制措施。</br>
可以基于 Consumer 或者 channel 设置 prefetch count 的值，含义为 Consumer端的最大的 unacked messages 数目。当超过这个数值的消息未被确认，RabbitMQ 会 停止投递新的消息给该消费者。</br>

``` java
channel.basicQos(2); // 如果超过 2 条消息没有发送 ACK，当前消费者不再接受队列消息
channel.basicConsume(QUEUE_NAME, false, consumer);
```

SimpleMessageListenerContainer
``` java
container.setPrefetchCount(2);
```

Spring Boot 配置:
``` java
spring.rabbitmq.listener.simple.prefetch=2
```


# Spring AMQP
## Spring AMQP 介绍
Spring 封装 RabbitMQ 的时候，它做了什么事情?
1. 管理对象(队列、交换机、绑定) 
2. 封装方法(发送消息、接收消息)

Spring AMQP 是对 Spring 基于 AMQP 的消息收发解决方案，它是一个抽象层， 不依赖于特定的 AMQP Broker 实现和客户端的抽象，所以可以很方便地替换。比如我 们可以使用 spring-rabbit 来实现。

``` maven
<dependency>
    <groupId>org.springframework.amqp</groupId>
    <artifactId>spring-rabbit</artifactId>
    <version>1.3.5.RELEASE</version>
</dependency>
```
包括 3 个 jar 包: Amqp-client-3.3.4.jar Spring-amqp.jar Spring.rabbit.jar



# rabbitmq消息可靠性投递

![](/img/rabbitmq/rabbitmq_work_model2.jpg)

在我们使用 RabbitMQ 收发消息的时候，有几个主要环节: 
1. 代表消息从生产者发送到 Broker.
生产者把消息发到 Broker 之后，怎么知道自己的消息有没有被 Broker 成功接 收?
2. 代表消息从 Exchange 路由到 Queue.
Exchange 是一个绑定列表，如果消息没有办法路由到正确的队列，会发生什么事情?应该怎么处理?
3. 代表消息在 Queue 中存储
队列是一个独立运行的服务，有自己的数据库(Mnesia)，它是真正用来存储消 息的。如果还没有消费者来消费，那么消息要一直存储在队列里面。如果队列出了问 题，消息肯定会丢失。怎么保证消息在队列稳定地存储呢?
4. 代表消费者订阅 Queue 并消费消息
队列的特性是什么?FIFO。队列里面的消息是一条一条的投递的，也就是说，只有上一条消息被消费者接收以后，才能把这一条消息从数据库删掉，继续投递下一条 消息。那么问题来了，Broker 怎么知道消费者已经接收了消息呢?


## 消息发送到 RabbitMQ 服务器
第一个环节是生产者发送消息到 Broker。可能因为网络或者 Broker 的问题导致消息 发送失败，生产者不能确定 Broker 有没有正确的接收。

在 RabbitMQ 里面提供了两种机制服务端确认机制，也就是在生产者发送消息给 RabbitMQ 的服务端的时候，服务端会通过某种方式返回一个应答，只要生产者收到了 这个应答，就知道消息发送成功了。

第一种是 Transaction(事务)模式，第二种 Confirm(确认)模式。

### Transaction(事务)模式
事务模式怎么使用呢?

我们通过一个 channel.txSelect()的方法把信道设置成事务模式，然后就可以发布消 息给 RabbitMQ 了，如果 channel.txCommit();的方法调用成功，就说明事务提交成功， 则消息一定到达了 RabbitMQ 中。

如果在事务提交执行之前由于 RabbitMQ 异常崩溃或者其他原因抛出异常，这个时 候我们便可以将其捕获，进而通过执行 channel.txRollback()方法来实现事务回滚。
流程图如下：
![](/img/rabbitmq/rabbitmq_broker_transaction.jpg)

在事务模式里面，只有收到了服务端的 Commit-OK 的指令，才能提交成功。所以
可以解决生产者和服务端确认的问题。但是事务模式有一个缺点，它是阻塞的，一条消息没有发送完毕，不能发送下一条消息，它会榨干 RabbitMQ 服务器的性能。所以不建 议大家在生产环境使用。

SpringBoot中的设置：
``` java
rabbitTemplate.setChannelTransacted(true);
```
### Confirm(确认)模式
确认模式有三种，一种是普通确认模式。

在生产者这边通过调用 channel.confirmSelect()方法将信道设置为 Confirm 模式， 然后发送消息。一旦消息被投递到所有匹配的队列之后，RabbitMQ 就会发送一个确认 (Basic.Ack)给生产者，也就是调用 channel.waitForConfirms()返回 true，这样生产 者就知道消息被服务端接收了。

这种发送 1 条确认 1 条的方式消息还不是太高，所以我们还有一种批量确认的方式。 批量确认，就是在开启 Confirm 模式后，先发送一批消息。只要 channel.waitForConfirmsOrDie();方法没有抛出异常，就代表消息都被服务端接收了。

批量确认的方式比单条确认的方式效率要高，但是也有两个问题，第一个就是批量 的数量的确定。对于不同的业务，到底发送多少条消息确认一次?数量太少，效率提升 不上去。数量多的话，又会带来另一个问题，比如我们发 1000 条消息才确认一次，如果 前面 999 条消息都被服务端接收了，如果第 1000 条消息被拒绝了，那么前面所有的消 息都要重发。


异步确认模式。

异步确认模式需要添加一个 ConfirmListener，并且用一个 SortedSet 来维护没有 被确认的消息。
Confirm 模式是在 Channel 上开启的，因为 RabbitTemplate 对 Channel 进行了封 装，叫做 ConfimrCallback。


``` java
rabbitTemplate.setConfirmCallback(new RabbitTemplate.ConfirmCallback() { 
    @Override
    public void confirm(CorrelationData correlationData, boolean ack, String cause) {
        if (!ack) {
            System.out.println("发送消息失败:" + cause);
            throw new RuntimeException("发送异常:" + cause);
        }
    }
});
```

## 消息从 Exchange 路由到 Queue
第二个环节就是消息从交换机路由到队列。在什么情况下，消息会无法路由到正确 的队列?可能因为路由键错误，或者队列不存在。

我们有两种方式处理无法路由的消息，一种就是让服务端重发给生产者，一种是让 交换机路由到另一个备份的交换机。

消息回发的方式:使用 mandatory 参数和 ReturnListener(在 Spring AMQP 中是 ReturnCallback)。


``` java
rabbitTemplate.setMandatory(true);
rabbitTemplate.setReturnCallback(new RabbitTemplate.ReturnCallback(){
    public void returnedMessage(Message message, int replyCode,
    String replyText, String exchange, String routingKey){
        System.out.println("回发的消息:");
        System.out.println("replyCode: "+replyCode);
        System.out.println("replyText: "+replyText);
        System.out.println("exchange: "+exchange);
        System.out.println("routingKey: "+routingKey);
    }
});
```

消息路由到备份交换机的方式:在创建交换机的时候，从属性中指定备份交换机。
``` java
Map<String,Object> arguments = new HashMap<String,Object>(); 
arguments.put("alternate-exchange","ALTERNATE_EXCHANGE"); // 指定交换机的备份交换机 ​
channel.exchangeDeclare("TEST_EXCHANGE","topic", false, false, false, arguments);
```

(注意区别，队列可以指定死信交换机;交换机可以指定备份交换机)

## 消息在 Queue 中存储
第三个环节是消息在队列存储，如果没有消费者的话，队列一直存在在数据库中。

如果 RabbitMQ 的服务或者硬件发生故障，比如系统宕机、重启、关闭等等，可能 会导致内存中的消息丢失，所以我们要把消息本身和元数据(队列、交换机、绑定)都 保存到磁盘。

解决方案：
### 队列持久化
``` java
@Bean("GpQueue")
public Queue GpQueue() {
    // queueName, durable, exclusive, autoDelete, Properties
    return new Queue("GP_TEST_QUEUE", true, false, false, new HashMap<>());
}
```
### 交换机持久化
``` java

@Bean("GpExchange")
public DirectExchange exchange() {
    // exchangeName, durable, exclusive, autoDelete, Properties
    return new DirectExchange("GP_TEST_EXCHANGE", true, false, new HashMap<>());
}
```

### 消息持久化
``` java
MessageProperties messageProperties = new MessageProperties();
messageProperties.setDeliveryMode(MessageDeliveryMode.PERSISTENT);
Message message = new Message("持久化消息".getBytes(), messageProperties);
rabbitTemplate.send("GP_TEST_EXCHANGE", "gupao.test", message);
```
### 集群
如果只有一个 RabbitMQ 的节点，即使交换机、队列、消息做了持久化，如果服务 崩溃或者硬件发生故障，RabbitMQ 的服务一样是不可用的，所以为了提高 MQ 服务的 可用性，保障消息的传输，我们需要有多个 RabbitMQ 的节点



## 消费者订阅 Queue 并消费消息
如果消费者收到消息后没来得及处理即发生异常，或者处理过程中发生异常，会导 致4失败。服务端应该以某种方式得知消费者对消息的接收情况，并决定是否重新投递 这条消息给其他消费者。

RabbitMQ 提供了消费者的消息确认机制(message acknowledgement)，消费 者可以自动或者手动地发送 ACK 给服务端。

没有收到 ACK 的消息，消费者断开连接后，RabbitMQ 会把这条消息发送给其他消 费者。如果没有其他消费者，消费者重启后会重新消费这条消息，重复执行业务逻辑。

消费者在订阅队列时，可以指定 autoAck 参数，当 autoAck 等于 false 时，RabbitMQ 会等待消费者显式地回复确认信号后才从队列中移去消息。

如何设置手动 ACK?

SimpleRabbitListenerContainer 或者 SimpleRabbitListenerContainerFactory
```java
factory.setAcknowledgeMode(AcknowledgeMode.MANUAL);
```

application.properties
``` java
spring.rabbitmq.listener.direct.acknowledge-mode=manual 
spring.rabbitmq.listener.simple.acknowledge-mode=manual
```

注意这三个值的区别: </br>
NONE:自动 ACK</br>
MANUAL: 手动 ACK</br>
AUTO:如果方法未抛出异常，则发送 ack。</br>

当抛出 AmqpRejectAndDontRequeueException 异常的时候，则消息会被拒绝，
且不重新入队。当抛出 ImmediateAcknowledgeAmqpException 异常，则消费者会 发送 ACK。其他的异常，则消息会被拒绝，且 requeue = true 会重新入队。

在springboot中：消费者又怎么调用 ACK，或者说怎么获得 Channel 参数呢?

```java
public class SecondConsumer {
    @RabbitHandler
    public void process(String msgContent,Channel channel, Message message) throws IOException {
        System.out.println("Second Queue received msg : " + msgContent );
        channel.basicAck(message.getMessageProperties().getDeliveryTag(), false); 
    }
}
```
如果消息无法处理或者消费失败，也有两种拒绝的方式，Basic.Reject()拒绝单条， Basic.Nack()批量拒绝。如果 requeue 参数设置为 true，可以把这条消息重新存入队列， 以便发给下一个消费者(当然，只有一个消费者的时候，这种方式可能会出现无限循环 重复消费的情况。可以投递到新的队列中，或者只打印异常日志)。

思考:服务端收到了 ACK 或者 NACK，生产者会知道吗?即使消费者没有接收到消息，或者消费时出现异常，生产者也是完全不知情的。

例如，我们寄出去一个快递，是怎么知道收件人有没有收到的?因为有物流跟踪和 签收反馈，所以寄件人可以知道。

在没有用上电话的年代，我们寄出去一封信，是怎么知道收信人有没有收到信件? 只有收到回信，才知道寄出的信被收到了。

所以，这个是生产者最终确定消费者有没有消费成功的两种方式:
1. 消费者收到消息，处理完毕后，调用生产者的API(思考:是否破坏解耦?) 
2. 消费者收到消息，处理完毕后，发送一条响应消息给生产者。

### 消费者回调
1) 调用生产者 API 例如:提单系统给其他系统发送了碎屏保消息后，其他系统必须在处理完消息后调用提单系统提供的 API，来修改提单系统中数据的状态。只要 API 没有被调用， 数据状态没有被修改，提单系统就认为下游系统没有收到这条消息。

2) 发送响应消息给生产者

例如:商业银行与人民银行二代支付通信，无论是人行收到了商业银行的消息， 还是商业银行收到了人行的消息，都必须发送一条响应消息(叫做回执报文)。
![](/img/rabbitmq/rabbitmq_customer_callback.jpg)


### 补偿机制

如果生产者的 API 就是没有被调用，也没有收到消费者的响应消息，怎么办? </br>
不要着急，可能是消费者处理时间太长或者网络超时。 </br>
生产者与消费者之间应该约定一个超时时间，比如 5 分钟，对于超出这个时间没有得到响应的消息，可以设置一个定时重发的机制，但要发送间隔和控制次数，比如每隔 2 分钟发送一次，最多重发 3 次，否则会造成消息堆积。</br>
重发可以通过消息落库+定时任务来实现。
重发，是否发送一模一样的消息?

参考:
ATM 机上运行的系统叫 C 端(ATMC)，前置系统叫 P 端(ATMC)，它接收 ATMC的消息，再转发给卡系统或者核心系统。</br>
1)如果客户存款，没有收到核心系统的应答，不知道有没有记账成功，最多发次存款确认报文，因为已经吞钞了，所以要保证成功;</br>
2)如果客户取款，ATMC 未得到应答时，最多发送 5 次存款冲正报文。因为没有吐钞，所以要保证失败。


### 消息幂等性
如果消费者每一次接收生产者的消息都成功了，只是在响应或者调用 API 的时候出 了问题，会不会出现消息的重复处理?例如:存款 100 元，ATM 重发了 5 次，核心系统 一共处理了 6 次，余额增加了 600 元。

所以，为了避免相同消息的重复处理，必须要采取一定的措施。RabbitMQ 服务端 是没有这种控制的(同一批的消息有个递增的 DeliveryTag)，它不知道你是不是就要把 一条消息发送两次，只能在消费端控制。

如何避免消息的重复消费?

消息出现重复可能会有两个原因:
1. 生产者的问题，环节1重复发送消息，比如在开启了 Confirm 模式但未收到 确认，消费者重复投递。
2. 环节4出了问题，由于消费者未发送 ACK 或者其他原因，消息重复投递。
3. 生产者代码或者网络问题。 对于重复发送的消息，可以对每一条消息生成一个唯一的业务 ID，通过日志或者消
息落库来做重复控制。

### 最终一致

如果确实是消费者宕机了，或者代码出现了 BUG 导致无法正常消费，在我们尝试多 次重发以后，消息最终也没有得到处理，怎么办?</br>

例如存款的场景，客户的钱已经被吞了，但是余额没有增加，这个时候银行出现了 长款，应该怎么处理?如果客户没有主动通知银行，这个问题是怎么发现的?银行最终 怎么把这个账务做平?</br>

在我们的金融系统中，都会有双方对账或者多方对账的操作，通常是在一天的业务 结束之后，第二天营业之前。我们会约定一个标准，比如 ATM 跟核心系统对账，肯定是 以核心系统为准。ATMC 获取到核心的对账文件，然后解析，登记成数据，然后跟自己 记录的流水比较，找出核心有 ATM 没有，或者 ATM 有核心没有，或者两边都有但是金 额不一致的数据。</br>

对账之后，我们再手工平账。比如取款记了账但是没吐钞的，做一笔冲正。存款吞 了钞没记账的，要么把钱退给客户，要么补一笔账。</br>

### 消息的顺序性

消息的顺序性指的是消费者消费消息的顺序跟生产者生产消息的顺序是一致的。
例如:商户信息同步到其他系统，有三个业务操作:1、新增门店 2、绑定产品 3、 激活门店，这种情况下消息消费顺序不能颠倒(门店不存在时无法绑定产品和激活)。
又比如:1、发表微博;2、发表评论;3、删除微博。顺序不能颠倒。
在 RabbitMQ 中，一个队列有多个消费者时，由于不同的消费者消费消息的速度是不一样的，顺序无法保证。只有一个队列仅有一个消费者的情况才能保证顺序消费(不同的业务消息发送到不同的专用的队列)。
![](/img/rabbitmq/rabbitmq_message_orderwork.jpg)

# 集群与高可用

## 为什么要做集群
集群主要用于实现高可用与负载均衡。
- 高可用:如果集群中的某些 MQ 服务器不可用，客户端还可以连接到其他 MQ 服务 器。
- 负载均衡:在高并发的场景下，单台 MQ 服务器能处理的消息有限，可以分发给多 台 MQ 服务器。

RabbitMQ 有两种集群模式:普通集群模式和镜像队列模式。

## RabbitMQ 如何支持集群
应用做集群，需要面对数据同步和通信的问题。因为 Erlang 天生具备分布式的特性， 所以 RabbitMQ 天然支持集群，不需要通过引入 ZK 或者数据库来实现数据同步。

RabbitMQ 通过/var/lib/rabbitmq/.erlang.cookie 来验证身份，需要在所有节点上 保持一致。


## RabbitMQ 的节点类型
集群有两种节点类型，一种是磁盘节点(Disc Node)，一种是内存节点(RAM Node)。</br>
磁盘节点:将元数据(包括队列名字属性、交换机的类型名字属性、绑定、vhost) 放在磁盘中。</br>
内存节点:将元数据放在内存中。</br>
PS:内存节点会将磁盘节点的地址存放在磁盘(不然重启后就没有办法同步数据了)。 如果是持久化的消息，会同时存放在内存和磁盘。</br>
集群中至少需要一个磁盘节点用来持久化元数据，否则全部内存节点崩溃时，就无 从同步元数据。未指定类型的情况下，默认为磁盘节点。</br>
我们一般把应用连接到内存节点(读写快)，磁盘节点用来备份。</br>
集群通过 25672 端口两两通信，需要开放防火墙的端口。</br>
需要注意的是，RabbitMQ 集群无法搭建在广域网上，除非使用 federation 或者 shovel 等插件(没这个必要，在同一个机房做集群)。</br>

集群的配置步骤:</br>
1、配置 hosts</br>
2、同步 erlang.cookie</br>
3、加入集群(join cluster)</br>

## 普通集群
普通集群模式下，不同的节点之间只会相互同步元数据。

![](/img/rabbitmq/rabbitmq_nomal_jiqun.jpg)

疑问:为什么不直接把队列的内容(消息)在所有节点上复制一份?</br>
主要是出于存储和同步数据的网络开销的考虑，如果所有节点都存储相同的数据， 就无法达到线性地增加性能和存储容量的目的(堆机器)。</br>
假如生产者连接的是节点 3，要将消息通过交换机 A 路由到队列 1，最终消息还是会 转发到节点 1 上存储，因为队列 1 的内容只在节点 1 上。</br>
同理，如果消费者连接是节点 2，要从队列 1 上拉取消息，消息会从节点 1 转发到 节点 2。其它节点起到一个路由的作用，类似于指针。</br>

普通集群模式不能保证队列的高可用性，因为队列内容不会复制。如果节点失效将 导致相关队列不可用，因此我们需要第二种集群模式。

## 镜像集群
第二种集群模式叫做镜像队列。</br>
镜像队列模式下，消息内容会在镜像节点间同步，可用性更高。不过也有一定的副 作用，系统性能会降低，节点过多的情况下同步的代价比较大。

![](/img/rabbitmq/rabbitmq_jingxiang_jiqun.jpg)

## 高可用
集群搭建成功后，如果有多个内存节点，那么生产者和消费者应该连接到哪个内存节点?如果在我们的代码中根据一定的策略来选择要使用的服务器，那每个地方都要修改，客户端的代码就会出现很多的重复，修改起来也比较麻烦。

所以需要一个负载均衡的组件(例如 HAProxy，LVS，Nignx)，由负载的组件来做路由。这个时候，只需要连接到负载组件的 IP 地址就可以了。
![](/img/rabbitmq/rabbitmq_gaokeyong_broke.jpg)

但是，如果这个负载的组件也挂了呢?客户端就无法连接到任意一台 MQ 的服务器 了。所以负载软件本身也需要做一个集群。新的问题又来了，如果有两台负载的软件， 客户端应该连哪个?

![](/img/rabbitmq/rabbitmq_loadbalance.jpg)

负载之上再负载?陷入死循环了。这个时候我们就要换个思路了。


我们应该需要这样一个组件:
1、 它本身有路由(负载)功能，可以监控集群中节点的状态(比如监控 HAProxy)，如果某个节点出现异常或者发生故障，就把它剔除掉。</br>
2、 为了提高可用性，它也可以部署多个服务，但是只有一个自动选举出 来的 MASTER 服务器(叫做主路由器)，通过广播心跳消息实现。</br>
3、 MASTER 服务器对外提供一个虚拟 IP，提供各种网络功能。也就是 谁抢占到 VIP，就由谁对外提供网络服务。应用端只需要连接到这一 个 IP 就行了。</br>

这个协议叫做 VRRP 协议(虚拟路由冗余协议 Virtual Router Redundancy Protocol)，这个组件就是 Keepalived，它具有 Load Balance 和 High Availability 的功能。


下面我们看下用 HAProxy 和 Keepalived 如何实现 RabbitMQ 的高可用 (MySQL、Mycat、Redis 类似)。

## 基于 Docker 安装 HAproxy 负载+Keepalived 高可用
![](/img/rabbitmq/rabbitmq_HAproxy_Keepalived.jpg)

规划:
内存节点 1:192.168.8.40 </br>
内存节点 2:192.168.8.45 </br>
磁盘节点:192.168.8.150 </br>
VIP:192.168.8.220 </br>

1、我们规划了两个内存节点，一个磁盘节点。所有的节点之间通过镜像队列的 方式同步数据。内存节点用来给应用访问，磁盘节点用来持久化数据。</br>
2、为了实现对两个内存节点的负载，我们安装了两个 HAProxy，监听两个 5672 和 15672 的端口。</br>
3、安装两个 Keepalived，一主一备。两个 Keepalived 抢占一个 VIP192.168.8.220。谁抢占到这个 VIP，应用就连接到谁，来执行对 MQ 的负载。</br>
这种情况下，我们的 Keepalived 挂了一个节点，没有影响，因为 BACKUP 会变 成 MASTER，抢占 VIP。HAProxy 挂了一个节点，没有影响，我们的 VIP 会自动路 由的可用的 HAProxy 服务。RabbitMQ 挂了一个节点，没有影响， 因为 HAProxy 会自动负载到可用的节点。</br>




# 实践经验总结

## 资源管理
到底在消费者创建还是在生产者创建?</br>
如果 A 项目和 B 项目有相互发送和接收消息，应该创建几个 vhost，几个 Exchange?</br>
交换机和队列，实际上是作为资源，由运维管理员创建的。 为什么仍然需要在代码中定义?重复创建不报错吗?</br>

![](/img/rabbitmq/rabbitmq_resource_apply.jpg)


## 配置文件与命名规范
1、元数据的命名集中放在 properties 文件中，不要用硬编码。如果有多个系统， 可以配置多个 xxx_mq.properties。

2、命名体现元数据的类型 
- 虚拟机命名: XXX_VHOST 
- 交换机命名:XXX_EXCHANGE
- 队列命名:_QUEUE

3、命名体现数据来源和去向
例如:销售系统发往产品系统的交换机:SALE_TO_PRODUCT_EXCHANGE。做到
见名知义，不用去查文档(当然注释是必不可少的)。

## 调用封装
在项目中可以对 Template 做进一步封装，简化消息的发送。

例如:如果交换机、路由键是固定的，封装之后就只需要一个参数:消息内容。

另外，如果想要平滑地迁移不同的 MQ(如果有这种需求的话)，也可以再做一层简单的封装。

```java

GpSendMsg(){
    JmsTemplate.send(destination,msg); 
}
这时，如果要把 ActiveMQ 替换为 RabbitMQ，只需要修改:

GpSendMsg(){
    RabbitTemplate.send(exchange,routingKey,msg); 
}
```

## 信息落库+定时任务
将需要发送的消息保存在数据库中，可以实现消息的可追溯和重复控制，需要配合定时任务来实现。
1) 将需要发送的消息登记在消息表中。
2) 定时任务一分钟或半分钟扫描一次，将未发送的消息发送到 MQ 服务器，并且修改状态为已发送。
3) 如果需要重发消息，将指定消息的状态修改为未发送即可。

## 生产环境运维监控

虽然 RabbitMQ 提供了一个简单的管理界面，但是如果对于系统性能、高可用和其他参数有一些定制化的监控需求的话，我们就需要通过其他方式来实现监控了。
主要关注:磁盘、内存、连接数。
zabbix

## 日志追踪

RabbitMQ 可以通过 Firehose 功能来记录消息流入流出的情况，用于调试，排错。

它是通过创建一个 TOPIC 类型的交换机(amq.rabbitmq.trace)，把生产者发送给 Broker 的消息或者 Broker 发送给消费者的消息发到这个默认的交换机上面来实现的。

另外 RabbitMQ 也提供了一个 Firehose 的 GUI 版本，就是 Tracing 插件。

启用 Tracing 插件后管理界面右侧选项卡会多一个 Tracing，可以添加相应的策略。 RabbitMQ 还提供了其他的插件来增强功能。
https://www.rabbitmq.com/firehose.html

## 如何减少连接数
在发送大批量消息的情况下，创建和释放连接依然有不小的开销。我们可以跟接收方约定批量消息的格式，比如支持 JSON 数组的格式，通过合并消息内容，可以减少生产者/消费者与 Broker 的连接。

比如:活动过后，要全范围下线产品，通过 Excel 导入模板，通常有几万到几十万条 解绑数据，合并发送的效率更高。

建议单条消息不要超过 4M(4096KB)，一次发送的消息数需要合理地控制。

# 面试题

1、 消息队列的作用与使用场景?

2、 Channel 和 vhost 的作用是什么?
Channel:减少 TCP 资源的消耗。也是最重要的编程接口。 Vhost:提高硬件资源利用率，实现资源隔离。

3、 RabbitMQ 的消息有哪些路由方式?适合在什么业务场景使用?
Direct、Topic、Fanout

4、 交换机与队列、队列与消费者的绑定关系是什么样的? 多个消费者监听一个队列时(比如一个服务部署多个实例)，消息会重复消费吗?
多对多;
轮询(平均分发)


5、 无法被路由的消息，去了哪里? 直接丢弃。可用备份交换机(alternate-exchange)接收。

6、 消息在什么时候会变成 Dead Letter(死信)? 消息过期;消息超过队列长度或容量;消息被拒绝并且未设置重回队列

7、 如果一个项目要从多个服务器接收消息，怎么做? 如果一个项目要发送消息到多个服务器，怎么做?
定义多个 ConnectionFactory，注入到消费者监听类/Temaplate。

8、 RabbitMQ 如何实现延迟队列? 基于数据库+定时任务; 或者消息过期+死信队列; 或者延迟队列插件。

9、 哪些情况会导致消息丢失?怎么解决? 哪些情况会导致消息重复?怎么解决? 从消息发送的整个流程来分析。


10、 一个队列最多可以存放多少条消息? 由硬件决定。

11、 可以用队列的 x-max-length 最大消息数来实现限流吗?例如秒杀场景。 不能，因为会删除先入队的消息，不公平。

12、 如何提高消息的消费速率? 创建多个消费者。

13、 AmqpTemplate 和 RabbitTemplate 的区别?
Spring AMQP 是 Spring 整合 AMQP 的一个抽象。Spring-Rabbit 是一个实现。


14、 如何动态地创建消费者监听队列? 通过 ListenerContainer
``` java
container.setQueues(getSecondQueue(), getThirdQueue()); //监听的队列
```

15、 Spring AMQP 中消息怎么封装?用什么转换? Message，MessageConvertor

16、 如何保证消息的顺序性? 一个队列只有一个消费者

17、 RabbitMQ 的集群节点类型? 磁盘节点和内存节点

18、 如何保证 RabbitMQ 的高可用?
HAProxy(LVS)+Keepalived

19、 大量消息堆积怎么办?
1) 重启(不是开玩笑的)
2) 多创建几个消费者同时消费
3) 直接清空队列，重发消息
