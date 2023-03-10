# 测试 Spring JMS

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-jms-testing>

## 1。概述

在本教程中，我们将创建一个简单的 Spring 应用程序，它连接到 ActiveMQ 来发送和接收消息。我们将重点测试这个应用程序以及测试 Spring JMS 的不同方法。

## 2。应用程序设置

首先，让我们创建一个可用于测试的基本应用程序。我们需要添加必要的依赖项并实现消息处理。

### 2.1。依赖性

让我们将所需的依赖项添加到项目的`pom.xml`中。**我们需要 [Spring JMS](https://web.archive.org/web/20220910150059/https://search.maven.org/artifact/org.springframework/spring-jms/4.3.4.RELEASE/jar) 来监听 JMS 消息。我们将使用 [ActiveMQ-Junit](https://web.archive.org/web/20220910150059/https://search.maven.org/artifact/org.apache.activemq.tooling/activemq-junit/5.17.1/jar) 为一部分测试启动一个嵌入式 ActiveMQ 实例，并使用 [TestContainers](https://web.archive.org/web/20220910150059/https://search.maven.org/artifact/org.testcontainers/testcontainers/1.17.3/jar) 在其他测试中运行一个 ActiveMQ Docker 容器:**

```java
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-jms</artifactId>
    <version>4.3.4.RELEASE</version>
</dependency>

<dependency>
    <groupId>org.apache.activemq.tooling</groupId>
    <artifactId>activemq-junit</artifactId>
    <version>5.16.5</version>
    <scope>test</scope>
</dependency>

<dependency>
    <groupId>org.testcontainers</groupId>
    <artifactId>testcontainers</artifactId>
    <version>1.17.3</version>
    <scope>test</scope>
</dependency>
```

### 2.2。申请代码

现在让我们创建一个可以监听消息的 Spring 应用程序:

```java
@ComponentScan
public class JmsApplication {
    public static void main(String[] args) {
        ApplicationContext context = new AnnotationConfigApplicationContext(JmsApplication.class);
    }
} 
```

我们需要创建一个配置类，**使用 [`@EnableJms`注释](/web/20220910150059/https://www.baeldung.com/spring-jms#Configuration)启用 JMS，并配置 [`ConnectionFactory`](/web/20220910150059/https://www.baeldung.com/spring-jms#Connection) 连接到我们的 ActiveMQ 实例**:

```java
@Configuration
@EnableJms
public class JmsConfig {

    @Bean
    public JmsListenerContainerFactory<?> jmsListenerContainerFactory() {
        DefaultJmsListenerContainerFactory factory = new DefaultJmsListenerContainerFactory();
        factory.setConnectionFactory(connectionFactory());
        return factory;
    }

    @Bean
    public ConnectionFactory connectionFactory() {
        return new ActiveMQConnectionFactory("tcp://localhost:61616");
    }

    @Bean
    public JmsTemplate jmsTemplate() {
        return new JmsTemplate(connectionFactory());
    }
}
```

之后，让我们创建可以接收和处理消息的侦听器:

```java
@Component
public class MessageListener {

    private static final Logger logger = LoggerFactory.getLogger(MessageListener.class);

    @JmsListener(destination = "queue-1")
    public void sampleJmsListenerMethod(TextMessage message) throws JMSException {
        logger.info("JMS listener received text message: {}", message.getText());
    }
} 
```

我们还需要一个可以发送消息的类:

```java
@Component
public class MessageSender {

    @Autowired
    private JmsTemplate jmsTemplate;

    private static final Logger logger = LoggerFactory.getLogger(MessageSender.class);

    public void sendTextMessage(String destination, String message) {
        logger.info("Sending message to {} destination with text {}", destination, message);
        jmsTemplate.send(destination, s -> s.createTextMessage(message));
    }
} 
```

## 3。使用嵌入式 ActiveMQ 进行测试

让我们测试我们的应用程序。我们将首先使用一个嵌入式 ActiveMQ 实例。让我们创建测试类，并添加一个管理 ActiveMQ 实例的 JUnit 规则:

```java
@RunWith(SpringRunner.class)
public class EmbeddedActiveMqTests4 {

    @ClassRule
    public static EmbeddedActiveMQBroker embeddedBroker = new EmbeddedActiveMQBroker();

    @Test
    public void test() {
    }

    // ...
}
```

让我们运行这个空测试并检查日志。**我们可以看到，嵌入式代理是从我们的测试**开始的:

```java
INFO | Starting embedded ActiveMQ broker: embedded-broker
INFO | Using Persistence Adapter: MemoryPersistenceAdapter
INFO | Apache ActiveMQ 5.14.1 (embedded-broker, ID:DESKTOP-52539-254421135-0:1) is starting
INFO | Apache ActiveMQ 5.14.1 (embedded-broker, ID:DESKTOP-52539-254421135-0:1) started
INFO | For help or more information please see: http://activemq.apache.org
INFO | Connector vm://embedded-broker started
INFO | Successfully connected to vm://embedded-broker?create=false
```

在我们的测试类中执行完所有测试之后，代理停止。

我们需要配置我们的应用程序来连接到这个 ActiveMQ 实例，以便我们可以正确地测试我们的`MessageListener`和`MessageSender`类:

```java
@Configuration
@EnableJms
static class TestConfiguration {
    @Bean
    public JmsListenerContainerFactory<?> jmsListenerContainerFactory() {
        DefaultJmsListenerContainerFactory factory = new DefaultJmsListenerContainerFactory();
        factory.setConnectionFactory(connectionFactory());
        return factory;
    }

    @Bean
    public ConnectionFactory connectionFactory() {
        return new ActiveMQConnectionFactory(embeddedBroker.getVmURL());
    }

    @Bean
    public JmsTemplate jmsTemplate() {
        return new JmsTemplate(connectionFactory());
    }
}
```

这个类使用一个特殊的`ConnectionFactory`,它从我们的嵌入式代理获取 URL。现在我们需要通过向包含我们的测试的类添加`@ContextConfiguration`注释来使用这个配置:

```java
@ContextConfiguration(classes = { TestConfiguration.class, MessageSender.class }) public class EmbeddedActiveMqTests {
```

### 3.1。发送消息

让我们编写第一个测试并检查我们的`MessageSender`类的功能。首先，我们需要通过简单地将它作为一个字段注入来获得对这个类的一个实例的引用:

```java
@Autowired
private MessageSender messageSender;
```

让我们向 ActiveMQ 发送一条简单的消息，并添加一些[断言](/web/20220910150059/https://www.baeldung.com/junit-assertions#assertions)来检查功能:

```java
@Test
public void whenSendingMessage_thenCorrectQueueAndMessageText() throws JMSException {
    String queueName = "queue-2";
    String messageText = "Test message";

    messageSender.sendTextMessage(queueName, messageText);

    assertEquals(1, embeddedBroker.getMessageCount(queueName));
    TextMessage sentMessage = embeddedBroker.peekTextMessage(queueName);
    assertEquals(messageText, sentMessage.getText());
}
```

**现在我们确信我们的`MessageSender`工作正常，因为在我们发送消息后，队列只包含一个具有正确文本的条目。**

### 3.2。接收消息

让我们也检查一下我们的监听器类。**让我们从创建一个新的测试方法并使用嵌入式代理发送消息开始。**我们的侦听器被设置为使用“queue-1”作为其目的地，因此我们需要确保我们在这里使用相同的名称。

让我们用[模仿](/web/20220910150059/https://www.baeldung.com/mockito-series)来检查听者的行为。我们将使用 [`@SpyBean`](/web/20220910150059/https://www.baeldung.com/mockito-series) 注释来获得`MessageListener:`的一个实例

```java
@SpyBean
private MessageListener messageListener;
```

然后，我们将[检查该方法是否被调用](/web/20220910150059/https://www.baeldung.com/mockito-verify)，并用 [`ArgumentCaptor`](/web/20220910150059/https://www.baeldung.com/mockito-argumentcaptor) 捕获接收到的方法参数:

```java
@Test
public void whenListening_thenReceivingCorrectMessage() throws JMSException {
    String queueName = "queue-1";
    String messageText = "Test message";

    embeddedBroker.pushMessage(queueName, messageText);
    assertEquals(1, embeddedBroker.getMessageCount(queueName));

    ArgumentCaptor<TextMessage> messageCaptor = ArgumentCaptor.forClass(TextMessage.class);

    Mockito.verify(messageListener, Mockito.timeout(100)).sampleJmsListenerMethod(messageCaptor.capture());

    TextMessage receivedMessage = messageCaptor.getValue();
    assertEquals(messageText, receivedMessage.getText());
}
```

我们现在可以运行测试了，它们都通过了。

## 4。用测试容器进行测试

让我们看看在 Spring 应用程序中测试 JMS 的另一种方法。我们可以使用 [TestContainers](/web/20220910150059/https://www.baeldung.com/docker-test-containers) 来运行一个 ActiveMQ Docker 容器，并在测试中连接到它。

让我们创建一个新的测试类，并将 Docker 容器作为一个 JUnit 规则包含进来:

```java
@RunWith(SpringRunner.class)
public class TestContainersActiveMqTests {

    @ClassRule
    public static GenericContainer<?> activeMqContainer 
      = new GenericContainer<>(DockerImageName.parse("rmohr/activemq:5.14.3")).withExposedPorts(61616);

    @Test
    public void test() throws JMSException {
    }
}
```

让我们运行这个测试并检查日志。**我们可以看到一些与 TestContainers 相关的信息，因为它正在提取指定的 docker 图像，并且启动了容器:**

```java
INFO | Creating container for image: rmohr/activemq:5.14.3
INFO | Container rmohr/activemq:5.14.3 is starting: e9b0ddcd45c54fc9994aff99d734d84b5fae14b55fdc70887c4a2c2309b229a7
INFO | Container rmohr/activemq:5.14.3 started in PT2.635S 
```

让我们创建一个类似于我们用 ActiveMQ 实现的配置类。唯一不同的是`ConnectionFactory`的配置:

```java
@Bean
public ConnectionFactory connectionFactory() {
    String brokerUrlFormat = "tcp://%s:%d";
    String brokerUrl = String.format(brokerUrlFormat, activeMqContainer.getHost(), activeMqContainer.getFirstMappedPort());
    return new ActiveMQConnectionFactory(brokerUrl);
}
```

### 4.1。发送消息

让我们测试一下我们的`MessageSender`类，看看它是否适用于这个 Docker 容器。**这次我们不能使用`EmbeddedBroker,`上的方法，但是弹簧`JmsTemplate`也很容易使用:**

```java
@Autowired
private MessageSender messageSender;

@Autowired
private JmsTemplate jmsTemplate;

@Test
public void whenSendingMessage_thenCorrectQueueAndMessageText() throws JMSException {
    String queueName = "queue-2";
    String messageText = "Test message";

    messageSender.sendTextMessage(queueName, messageText);

    Message sentMessage = jmsTemplate.receive(queueName);
    Assertions.assertThat(sentMessage).isInstanceOf(TextMessage.class);

    assertEquals(messageText, ((TextMessage) sentMessage).getText());
}
```

我们可以使用`JmsTemplate`来读取队列的内容，并检查我们的类是否发送了正确的消息。

### 4.2。接收消息

测试我们的监听器类也没什么不同。**让我们使用`JmsTemplate`发送一条消息，并验证我们的监听器是否收到了正确的文本:**

```java
@SpyBean
private MessageListener messageListener;

@Test
public void whenListening_thenReceivingCorrectMessage() throws JMSException {
    String queueName = "queue-1";
    String messageText = "Test message";

    jmsTemplate.send(queueName, s -> s.createTextMessage(messageText));

    ArgumentCaptor<TextMessage> messageCaptor = ArgumentCaptor.forClass(TextMessage.class);

    Mockito.verify(messageListener, Mockito.timeout(100)).sampleJmsListenerMethod(messageCaptor.capture());

    TextMessage receivedMessage = messageCaptor.getValue();
    assertEquals(messageText, receivedMessage.getText());
}
```

## 5。结论

在本文中，我们创建了一个可以用 Spring JMS 发送和接收消息的基本应用程序。然后，我们讨论了两种测试方法。

首先，我们使用了一个嵌入式 ActiveMQ 实例，它甚至提供了一些方便的方法来与代理交互。其次，我们使用 TestContainers 通过 docker 容器来测试我们的代码，docker 容器可以更好地模拟真实世界的场景。

和往常一样，这些例子的源代码可以在 GitHub 上的[处获得。](https://web.archive.org/web/20220910150059/https://github.com/eugenp/tutorials/tree/master/spring-jms)