# 使用 Spring Data Redis 发布订阅消息

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-data-redis-pub-sub>

## 1。概述

在探索 Spring Data Redis 系列的第二篇文章中，我们将看看发布/订阅消息队列。

在 Redis 中，发布者没有被编程为将他们的消息发送给特定的订阅者。更确切地说，发布的消息被表征为通道，而不知道可能有什么(如果有的话)订阅者。

类似地，订阅者表达对一个或多个主题的兴趣，并且只接收感兴趣的消息，而不知道有什么(如果有的话)发布者。

发布者和订阅者的这种分离可以允许更大的可伸缩性和更动态的网络拓扑。

## 2 .Redis 配置

让我们开始添加消息队列所需的配置。

首先，我们将定义一个`MessageListenerAdapter` bean，它包含一个名为`RedisMessageSubscriber`的`MessageListener`接口的自定义实现。该 bean 充当发布-订阅消息传递模型中的订阅者:

```
@Bean
MessageListenerAdapter messageListener() { 
    return new MessageListenerAdapter(new RedisMessageSubscriber());
}
```

`RedisMessageListenerContainer`是 Spring Data Redis 提供的一个类，为 Redis 消息监听器提供异步行为。这是内部调用，根据 Spring Data Redis 文档–“处理监听、转换和消息分发的底层细节”

```
@Bean
RedisMessageListenerContainer redisContainer() {
    RedisMessageListenerContainer container 
      = new RedisMessageListenerContainer(); 
    container.setConnectionFactory(jedisConnectionFactory()); 
    container.addMessageListener(messageListener(), topic()); 
    return container; 
}
```

我们还将使用定制的`MessagePublisher`接口和`RedisMessagePublisher`实现创建一个 bean。这样，我们可以有一个通用的消息发布 API，并让 Redis 实现将一个`redisTemplate`和`topic`作为构造函数参数:

```
@Bean
MessagePublisher redisPublisher() { 
    return new RedisMessagePublisher(redisTemplate(), topic());
}
```

最后，我们将设置一个主题，发布者将向该主题发送消息，订阅者将收到消息:

```
@Bean
ChannelTopic topic() {
    return new ChannelTopic("messageQueue");
}
```

## 3。发布消息

### 3.1。定义`MessagePublisher` 界面

Spring Data Redis 不提供用于消息分发的`MessagePublisher`接口。我们可以定义一个自定义接口，它将在实现中使用`redisTemplate` :

```
public interface MessagePublisher {
    void publish(String message);
}
```

### 3.2。`RedisMessagePublisher`实施

我们的下一步是提供一个`MessagePublisher`接口的实现，添加消息发布细节并使用`redisTemplate.`中的函数

该模板包含一组非常丰富的函数，可用于各种操作——其中`convertAndSend` 能够通过主题向队列发送消息:

```
public class RedisMessagePublisher implements MessagePublisher {

    @Autowired
    private RedisTemplate<String, Object> redisTemplate;
    @Autowired
    private ChannelTopic topic;

    public RedisMessagePublisher() {
    }

    public RedisMessagePublisher(
      RedisTemplate<String, Object> redisTemplate, ChannelTopic topic) {
      this.redisTemplate = redisTemplate;
      this.topic = topic;
    }

    public void publish(String message) {
        redisTemplate.convertAndSend(topic.getTopic(), message);
    }
} 
```

如您所见，publisher 的实现非常简单。它使用`redisTemplate`的`convertAndSend()`方法将给定的消息格式化并发布到配置的主题。

主题实现了发布和订阅语义:当一条消息发布后，它将被发送给所有注册来监听该主题的订阅者。

## 4。订阅消息

`RedisMessageSubscriber` 实现 Spring Data Redis 提供的`MessageListener` 接口:

```
@Service
public class RedisMessageSubscriber implements MessageListener {

    public static List<String> messageList = new ArrayList<String>();

    public void onMessage(Message message, byte[] pattern) {
        messageList.add(message.toString());
        System.out.println("Message received: " + message.toString());
    }
}
```

注意，还有一个名为`pattern`的参数，我们在这个例子中没有用到。Spring Data Redis 文档指出，这个参数表示“匹配通道的模式(如果指定)”，但是它可以是`null`。

## 5。发送和接收消息

现在我们把它们放在一起。让我们创建一条消息，然后使用`RedisMessagePublisher`发布它:

```
String message = "Message " + UUID.randomUUID();
redisMessagePublisher.publish(message);
```

当我们调用`publish(message)`时，内容被发送到 Redis，在那里它被路由到我们的 publisher 中定义的消息队列主题。然后将它分发给该主题的订阅者。

您可能已经注意到`RedisMessageSubscriber` 是一个监听器，它将自己注册到队列中以检索消息。

在消息到达时，订阅者定义的`onMessage()`方法被触发。

在我们的例子中，我们可以通过检查我们的`RedisMessageSubscriber`中的`messageList`来验证我们已经收到了已经发布的消息:

```
RedisMessageSubscriber.messageList.get(0).contains(message) 
```

## 6。结论

在本文中，我们研究了使用 Spring Data Redis 的发布/订阅消息队列实现。

上面例子的实现可以在[一个 GitHub 项目](https://web.archive.org/web/20221013034414/https://github.com/eugenp/tutorials/tree/master/persistence-modules/spring-data-redis)中找到。