# Spring JMS 入门

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-jms>

## 1。概述

Spring 提供了一个 JMS 集成框架，简化了 JMS API 的使用。本文介绍了这种集成的基本概念。

## 2。Maven 依赖关系

为了在我们的应用程序中使用 Spring JMS，我们需要在`pom.xml`中添加必要的构件:

```
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-jms</artifactId>
    <version>4.3.3.RELEASE</version>
</dependency> 
```

神器的最新版本可以在这里找到[。](https://web.archive.org/web/20220813071133/https://search.maven.org/classic/#search%7Cga%7C1%7Cg%3A%22org.springframework%22%20AND%20a%3A%22spring-jms%22)

## 3。`JmsTemplate`

**`JmsTemplate`类处理发送或同步接收消息时资源的创建和释放。**

因此，使用这个`JmsTemplate`的类只需要实现方法定义中指定的回调接口。

从 Spring 4.1 开始，`JmsMessagingTemplate`构建在`JmsTemplate`之上，它提供了与消息传递抽象的集成，即`org.springframework.messaging.Message.` ，这反过来允许我们创建一个以通用方式发送的消息。

## 4。连接管理

为了连接并能够发送/接收消息，我们需要配置一个`ConnectionFactory`。

**A `ConnectionFactory`是由管理员**预先配置的 JMS 管理对象之一。在配置的帮助下，客户端将与 JMS 提供者建立连接。

Spring 提供了两种类型的`ConnectionFactory`:

*   **`SingleConnectionFactory –`** 是`ConnectionFactory`接口的一个实现，它将在所有`createConnection`()调用上返回相同的连接，并忽略对`close`()的调用
*   **`CachingConnectionFactory`–**扩展了`SingleConnectionFactory`的功能，并添加了`Sessions`、`MessageProducers`和`MessageConsumers`的缓存来增强它

## 5。目的地管理

如上所述，和`ConnectionFactory`一样，目的地也是 JMS 管理的对象，可以从 JNDI 中存储和检索。

Spring 提供了像`DynamicDestinationResolver`这样的通用解析器和像 `JndiDestinationResolver`这样的特定解析器。

`JmsTemplate`将根据我们的选择将目的地名称的解析委托给其中一个实现。

它还将提供一个名为`defaultDestination`的属性，该属性将用于不指向特定目的地的`send`和`receive`操作。

## 6。消息转换

如果没有消息转换器的支持，Spring JMS 将是不完整的。

对于`ConvertAndSend()`和`ReceiveAndConvert()`操作，`JmsTemplate`使用的默认转换策略是`SimpleMessageConverter`类。

**`The SimpleMessageConverter`能够处理`TextMessages`、`BytesMessages`、`MapMessages`、`ObjectMessages`。这个类实现了`MessageConverter` 接口。**

除了`SimpleMessageConverter`，Spring JMS 还提供了一些其他的`MessageConverter`类，比如`MappingJackson2MessageConverter`、`MarshallingMessageConverter`、`MessagingMessageConverter`。

此外，我们可以简单地通过实现`MessageConverter`接口的`toMessage()`和`FromMessage()`方法来创建定制的消息转换功能。

让我们看看实现定制`MessageConverter`的示例代码片段，

```
public class SampleMessageConverter implements MessageConverter {
    public Object fromMessage(Message message) 
      throws JMSException, MessageConversionException {
        //...
    }

    public Message toMessage(Object object, Session session)
      throws JMSException, MessageConversionException { 
        //...
    }
}
```

## 7。样品弹簧 JMS

在这一节中，我们将看到如何使用`JmsTemplate` 来发送和接收消息。

发送消息的默认方法是`JmsTemplate.send()`。它有两个关键参数，第一个参数是 JMS 目的地，第二个参数是使用`MessageCreator`的回调方法`createMessage()`构造消息的`MessageCreator. The JmsTemplate`的实现。

**`JmsTemplate.send()`适用于发送纯文本消息，但是为了发送自定义消息，`JmsTemplate`有另一个方法叫做 c `onvertAndSend()`。**

下面我们可以看到这些方法的实现:

```
public class SampleJmsMessageSender {

    private JmsTemplate jmsTemplate;
    private Queue queue;

    // setters for jmsTemplate & queue

    public void simpleSend() {
        jmsTemplate.send(queue, s -> s.createTextMessage("hello queue world"));
    }
```

```
 public void sendMessage(Employee employee) { 
        System.out.println("Jms Message Sender : " + employee); 
        Map<String, Object> map = new HashMap<>(); 
        map.put("name", employee.getName()); map.put("age", employee.getAge()); 
        jmsTemplate.convertAndSend(map); 
    }
}
```

下面是消息接收者类，我们称之为消息驱动 POJO (MDP)。**我们可以看到类`SampleListener`正在实现`MessageListener`接口，并为接口方法`onMessage().`** 提供了文本具体实现

除了`onMessage()`方法，我们的`SampleListener`类还调用了一个方法`receiveAndConvert()`来接收自定义消息:

```
public class SampleListener implements MessageListener {

    public JmsTemplate getJmsTemplate() {
        return getJmsTemplate();
    }

    public void onMessage(Message message) {
        if (message instanceof TextMessage) {
            try {
                String msg = ((TextMessage) message).getText();
                System.out.println("Message has been consumed : " + msg);
            } catch (JMSException ex) {
                throw new RuntimeException(ex);
            }
        } else {
            throw new IllegalArgumentException("Message Error");
        }
    }

    public Employee receiveMessage() throws JMSException {
        Map map = (Map) getJmsTemplate().receiveAndConvert();
        return new Employee((String) map.get("name"), (Integer) map.get("age"));
    }
}
```

我们看到了如何实现`MessageListener`，下面我们看到了 Spring 应用程序上下文中的配置:

```
<bean id="messageListener" class="com.baeldung.spring.jms.SampleListener" /> 

<bean id="jmsContainer" 
  class="org.springframework.jms.listener.DefaultMessageListenerContainer"> 
    <property name="connectionFactory" ref="connectionFactory"/> 
    <property name="destinationName" ref="IN_QUEUE"/> 
    <property name="messageListener" ref="messageListener" /> 
</bean>
```

**`DefaultMessageListenerContainer`是 Spring 提供的默认消息监听器容器以及许多其他专用容器。**

## 8。带 Java 注释的基本配置

**`@JmsListener`是将普通 bean 的方法转换成 JMS 侦听器端点所需的唯一注释。** Spring JMS 提供了更多的注释来简化 JMS 的实现。

我们可以看到下面注释的一些示例类:

```
@JmsListener(destination = "myDestination")
public void SampleJmsListenerMethod(Message<Order> order) { ... }
```

为了给一个方法添加多个监听器，我们只需要添加多个`@JmsListener`注释。

**我们需要将`@EnableJms`注释添加到我们的一个配置类中，以支持`@JmsListener`注释方法:**

```
@Configuration
@EnableJms
public class AppConfig {

    @Bean
    public DefaultJmsListenerContainerFactory jmsListenerContainerFactory() {
        DefaultJmsListenerContainerFactory factory 
          = new DefaultJmsListenerContainerFactory();
        factory.setConnectionFactory(connectionFactory());
        return factory;
    }
}
```

## 9.错误处理程序

我们还可以为我们的消息监听器容器配置一个定制的错误处理程序。

让我们首先实现`org.springframework.util.ErrorHandler` 接口:

```
@Service
public class SampleJmsErrorHandler implements ErrorHandler {

    // ... logger

    @Override
    public void handleError(Throwable t) {
        LOG.warn("In default jms error handler...");
        LOG.error("Error Message : {}", t.getMessage());
    }

}
```

注意，我们已经覆盖了 `handleError()`方法，它只是记录错误消息。

然后，我们需要使用`setErrorHandler()` 方法在`DefaultJmsListenerConnectionFactory` 中引用我们的错误处理服务:

```
@Bean
public DefaultJmsListenerContainerFactorybjmsListenerContainerFactory() {
    DefaultJmsListenerContainerFactory factory 
      = new DefaultJmsListenerContainerFactory();
    factory.setConnectionFactory(connectionFactory());
    factory.setErrorHandler(sampleJmsErrorHandler);
    return factory;
}
```

这样，我们配置的错误处理程序现在将捕捉任何未处理的异常并记录消息。

可选地，我们也可以通过更新我们的 `appContext.xml:`,使用普通的 XML 配置来配置错误处理程序

```
<bean id="sampleJmsErrorHandler"
  class="com.baeldung.spring.jms.SampleJmsErrorHandler" />

<bean id="jmsContainer"
  class="org.springframework.jms.listener.DefaultMessageListenerContainer">
    <property name="connectionFactory" ref="connectionFactory" />
    <property name="destinationName" value="IN_QUEUE" />
    <property name="messageListener" ref="messageListener" />
    <property name="errorHandler" ref="sampleJmsErrorHandler" />
</bean>
```

## 10。结论

在本教程中，我们讨论了 Spring JMS 的配置和基本概念。我们还简要了解了用于发送和接收消息的特定于 Spring 的`JmsTemplate`类。

你可以在 [GitHub 项目](https://web.archive.org/web/20220813071133/https://github.com/eugenp/tutorials/tree/master/spring-jms)中找到代码实现。