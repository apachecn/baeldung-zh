# 春豆 vs . EJB-特征比较

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-bean-vs-ejb>

## 1.概观

这些年来，Java 生态系统有了巨大的发展和壮大。在此期间，Enterprise Java Beans 和 Spring 是两种技术，它们不仅相互竞争，还相互借鉴。

在本教程中，我们将看看它们的历史和差异。当然，我们会在 Spring world 中看到一些 EJB 及其对等物的代码示例。

## 2.这些技术的简史

首先，让我们快速浏览一下这两种技术的历史，以及它们是如何在这些年里稳步发展的。

### 2.1.企业 Java Beans

**EJB 规范是 Java EE(或 [J2EE，现在称为 Jakarta EE](/web/20220926183020/https://www.baeldung.com/java-enterprise-evolution) )规范**的子集。它的第一个版本出现在 1999 年，是第一个旨在使服务器端企业应用程序开发更容易的 Java 技术。

它承担了 Java 开发人员在并发性、安全性、持久性、事务处理等方面的负担。规范将这些和其他常见的企业问题交给了实现应用服务器的容器，容器无缝地处理它们。然而，由于需要大量的配置，照原样使用 EJB 有点麻烦。此外，事实证明这是一个性能瓶颈。

但是现在，随着注释的发明，以及来自 Spring 的激烈竞争，EJB 在其最新的 3.2 版本中比其首次发布的版本更易于使用。今天的企业 Java Beans 大量借鉴了 Spring 的依赖注入和 POJOs 的使用。

### 2.2.春天

当 EJB(以及一般的 Java EE)正在努力满足 Java 社区的需求时，Spring Framework 就像一股新鲜空气一样到来了。它的第一个里程碑式的版本出现在 2004 年，为 EJB 模型及其重量级容器提供了一个替代方案。

多亏了 Spring， **Java 企业应用程序现在可以在轻量级的 [IOC 容器](/web/20220926183020/https://www.baeldung.com/inversion-control-and-dependency-injection-in-spring#the-spring-ioc-container)T3 上运行。此外，它还提供了依赖反转、AOP 和 Hibernate 支持，以及许多其他有用的特性。在 Java 社区的大力支持下，Spring 现在已经成指数级增长，可以被称为一个完整的 Java/JEE 应用程序框架。**

在其最新的化身中，Spring 5.0 甚至支持反应式编程模型。另一个分支， [Spring Boot](/web/20220926183020/https://www.baeldung.com/spring-boot-start) ，凭借其嵌入式服务器和自动配置，完全改变了游戏规则。

## 3.功能比较的前奏

在使用代码示例进行特性比较之前，让我们先建立一些基础知识。

### 3.1.两者的基本区别

首先，基本的和明显的区别是 **EJB 是一个规范，而 Spring 是一个完整的框架**。

该规范由许多应用服务器实现，如 GlassFish、IBM WebSphere 和 JBoss/WildFly。这意味着我们选择使用 EJB 模型进行应用程序的后端开发是不够的。我们还需要选择使用哪个应用服务器。

理论上，企业 Java Beans 是跨应用服务器可移植的，尽管总是有一个先决条件，如果互操作性是一个选项，我们不应该使用任何特定于供应商的扩展。

第二， **Spring 就其广泛的产品组合而言，技术更接近 Java EE 而不是 EJB**。虽然 EJB 只指定后端操作，但是 Spring 和 Java EE 一样，也支持 UI 开发、RESTful APIs 和反应式编程等等。

### 3.2.有用信息

在接下来的小节中，我们将通过一些实际例子来比较这两种技术。由于 EJB 特征是更大的 Spring 生态系统的子集，我们将根据它们的类型来查看它们对应的 Spring 等价物。

为了更好地理解这些示例，可以考虑先阅读一下 [Java EE 会话 Bean](/web/20220926183020/https://www.baeldung.com/ejb-session-beans)、[消息驱动 Bean](/web/20220926183020/https://www.baeldung.com/ejb-message-driven-beans)、 [Spring Bean](/web/20220926183020/https://www.baeldung.com/spring-bean) 和 [Spring Bean 注释](/web/20220926183020/https://www.baeldung.com/spring-bean-annotations)。

我们将使用 [OpenJB](/web/20220926183020/https://www.baeldung.com/java-ee-singleton-session-bean#maven) 作为我们的嵌入式容器来运行 EJB 样本。对于运行大多数 Spring 示例，它的 IOC 容器就足够了；对于 Spring JMS，我们需要一个嵌入式 ApacheMQ 代理。

为了测试我们所有的样本，我们将使用 JUnit。

## 4.单身的 EJB ==春天`Component`

有时我们需要容器来创建一个 bean 的实例。例如，假设我们需要一个 bean 来统计 web 应用程序的访问者数量。**这个 bean 只需要在应用程序启动**时创建一次。

让我们看看如何使用[单例会话 EJB](/web/20220926183020/https://www.baeldung.com/java-ee-singleton-session-bean) 和[弹簧`Component`](/web/20220926183020/https://www.baeldung.com/spring-bean-annotations#component) 来实现这一点。

### 4.1.单身 EJB 的例子

我们首先需要一个接口来指定我们的 EJB 具有远程处理的能力:

```java
@Remote
public interface CounterEJBRemote {    
    int count();
    String getName();
    void setName(String name);
}
```

下一步是用注释`javax.ejb.Singleton` 和 viola 定义一个实现类。我们的独生子准备好了:

```java
@Singleton
public class CounterEJB implements CounterEJBRemote {
    private int count = 1;
    private String name;

    public int count() {
        return count++;
    }

    // getter and setter for name
} 
```

但是在我们测试 singleton(或任何其他 EJB 代码样本)之前，我们需要初始化`ejbContainer`并获取`context`:

```java
@BeforeClass
public void initializeContext() throws NamingException {
    ejbContainer = EJBContainer.createEJBContainer();
    context = ejbContainer.getContext();
    context.bind("inject", this);
} 
```

现在让我们来看看测试:

```java
@Test
public void givenSingletonBean_whenCounterInvoked_thenCountIsIncremented() throws NamingException {

    int count = 0;
    CounterEJBRemote firstCounter = (CounterEJBRemote) context.lookup("java:global/ejb-beans/CounterEJB");
    firstCounter.setName("first");

    for (int i = 0; i < 10; i++) {
        count = firstCounter.count();
    }

    assertEquals(10, count);
    assertEquals("first", firstCounter.getName());

    CounterEJBRemote secondCounter = (CounterEJBRemote) context.lookup("java:global/ejb-beans/CounterEJB");

    int count2 = 0;
    for (int i = 0; i < 10; i++) {
        count2 = secondCounter.count();
    }

    assertEquals(20, count2);
    assertEquals("first", secondCounter.getName());
} 
```

在上面的例子中需要注意一些事情:

*   我们使用 [JNDI](/web/20220926183020/https://www.baeldung.com/jndi) 查找从容器中获取`counterEJB`
*   `count2` 从`count` 离开单例的点开始，累加到`20`
*   `secondCounter` 保留我们为`firstCounter`设置的名称

最后两点证明了独生子女的重要性。因为每次查找时都使用相同的 bean 实例，所以总计数为 20，并且为一个实例设置的值对另一个实例保持不变。

### 4.2.单例 Spring Bean 示例

使用弹簧组件可以获得相同的功能。

这里我们不需要实现任何接口。相反，我们将添加`@Component`注释:

```java
@Component
public class CounterBean {
    // same content as in the EJB
}
```

**事实上，在 Spring** 中组件默认是单线态的。

我们还需要[配置 Spring 来扫描组件](/web/20220926183020/https://www.baeldung.com/spring-component-scanning#1-using-componentscan-in-aspring-application):

```java
@Configuration
@ComponentScan(basePackages = "com.baeldung.ejbspringcomparison.spring")
public class ApplicationConfig {} 
```

类似于我们初始化 EJB 上下文的方式，我们现在将设置 Spring 上下文:

```java
@BeforeClass
public static void init() {
    context = new AnnotationConfigApplicationContext(ApplicationConfig.class);
} 
```

现在让我们来看看我们的`Component`在运行:

```java
@Test
public void whenCounterInvoked_thenCountIsIncremented() throws NamingException {    
    CounterBean firstCounter = context.getBean(CounterBean.class);
    firstCounter.setName("first");
    int count = 0;
    for (int i = 0; i < 10; i++) {
        count = firstCounter.count();
    }

    assertEquals(10, count);
    assertEquals("first", firstCounter.getName());

    CounterBean secondCounter = context.getBean(CounterBean.class);
    int count2 = 0;
    for (int i = 0; i < 10; i++) {
        count2 = secondCounter.count();
    }

    assertEquals(20, count2);
    assertEquals("first", secondCounter.getName());
} 
```

正如我们所看到的，与 EJB 的唯一区别是我们如何从 Spring 容器的上下文中获取 bean，而不是 JNDI 查找。

## 5.范围为`prototype`的有状态 EJB == Spring `Component`

有时候，比方说当我们在构建购物车时，**我们需要 bean 在方法调用之间来回调用时记住它的状态**。

在这种情况下，我们需要容器为每次调用生成一个单独的 bean 并保存状态。让我们看看如何用我们的技术实现这一点。

### 5.1。有状态 EJB 示例

与我们的单例 EJB 示例类似，我们需要一个`javax.ejb.Remote`接口及其实现。只是这一次，它用`javax.ejb.Stateful`标注:

```java
@Stateful
public class ShoppingCartEJB implements ShoppingCartEJBRemote {
    private String name;
    private List<String> shoppingCart;

    public void addItem(String item) {
        shoppingCart.add(item);
    }
    // constructor, getters and setters
}
```

让我们编写一个简单的测试来设置一个`name`并将项目添加到一个`bathingCart`中。我们将检查它的大小并验证名称:

```java
@Test
public void givenStatefulBean_whenBathingCartWithThreeItemsAdded_thenItemsSizeIsThree()
  throws NamingException {
    ShoppingCartEJBRemote bathingCart = (ShoppingCartEJBRemote) context.lookup(
      "java:global/ejb-beans/ShoppingCartEJB");

    bathingCart.setName("bathingCart");
    bathingCart.addItem("soap");
    bathingCart.addItem("shampoo");
    bathingCart.addItem("oil");

    assertEquals(3, bathingCart.getItems().size());
    assertEquals("bathingCart", bathingCart.getName());
} 
```

现在，为了证明 bean 确实跨实例维护状态，让我们向该测试添加另一个 shoppingCartEJB:

```java
ShoppingCartEJBRemote fruitCart = 
  (ShoppingCartEJBRemote) context.lookup("java:global/ejb-beans/ShoppingCartEJB");

fruitCart.addItem("apples");
fruitCart.addItem("oranges");

assertEquals(2, fruitCart.getItems().size());
assertNull(fruitCart.getName()); 
```

这里我们没有设置`name`，因此它的值为空。回想一下单例测试，在一个实例中设置的名称被保留在另一个实例中。这表明我们从 bean 池中获得了具有不同实例状态的单独的`ShoppingCartEJB`实例。

### 5.2.有状态 Spring Bean 示例

为了用 Spring 得到同样的效果，我们需要一个带有[原型作用域](/web/20220926183020/https://www.baeldung.com/spring-bean-scopes#prototype)的`Component`:

```java
@Component
@Scope(value = ConfigurableBeanFactory.SCOPE_PROTOTYPE)
public class ShoppingCartBean {
   // same contents as in the EJB
} 
```

**就是这样，只是注释不同——代码的其余部分保持不变**。

为了测试我们的有状态 bean，我们可以使用与 EJB 相同的测试。唯一的区别还是我们如何从容器中获取 bean:

```java
ShoppingCartBean bathingCart = context.getBean(ShoppingCartBean.class); 
```

## 6.没有国家的 EJB！=春天的任何东西

有时，例如在搜索 API 中，**我们既不关心 bean 的实例状态，也不关心它是否是单例的**。我们只需要搜索结果，它可能来自我们关心的任何 bean 实例。

### 6.1.无国籍的 EJB 的例子

对于这种情况，EJB 有一个无状态的变体。**容器维护一个 beans 的实例池，任何一个都返回给调用方法**。

我们定义它的方式与其他 EJB 类型相同，使用远程接口，并使用`javax.ejb.Stateless`注释实现:

```java
@Stateless
public class FinderEJB implements FinderEJBRemote {

    private Map<String, String> alphabet;

    public FinderEJB() {
        alphabet = new HashMap<String, String>();
        alphabet.put("A", "Apple");
        // add more values in map here
    }

    public String search(String keyword) {
        return alphabet.get(keyword);
    }
} 
```

让我们添加另一个简单的测试来看看实际情况:

```java
@Test
public void givenStatelessBean_whenSearchForA_thenApple() throws NamingException {
    assertEquals("Apple", alphabetFinder.search("A"));        
} 
```

在上面的例子中，使用注释`javax.ejb.EJB`,`alphabetFinder`作为字段被注入到测试类中:

```java
@EJB
private FinderEJBRemote alphabetFinder; 
```

无状态 EJB 背后的核心思想是通过拥有相似 beans 的实例池来提高性能。

然而， **Spring 并不认同这种哲学，它只提供了作为无状态的单例。**

## 7.消息驱动 Beans = = Spring JMS

到目前为止讨论的所有 EJB 都是会话 beans。另一种是消息驱动的。顾名思义，**它们通常用于两个系统**之间的异步通信。

### 7.1.MDB 示例

为了创建消息驱动的企业 Java Bean，我们需要实现定义其`onMessage`方法的`javax.jms.MessageListener`接口，并将该类注释为`javax.ejb.MessageDriven`:

```java
@MessageDriven(activationConfig = { 
  @ActivationConfigProperty(propertyName = "destination", propertyValue = "myQueue"), 
  @ActivationConfigProperty(propertyName = "destinationType", propertyValue = "javax.jms.Queue") 
})
public class RecieverMDB implements MessageListener {

    @Resource
    private ConnectionFactory connectionFactory;

    @Resource(name = "ackQueue")
    private Queue ackQueue;

    public void onMessage(Message message) {
        try {
            TextMessage textMessage = (TextMessage) message;
            String producerPing = textMessage.getText();

            if (producerPing.equals("marco")) {
                acknowledge("polo");
            }
        } catch (JMSException e) {
            throw new IllegalStateException(e);
        }
    }
} 
```

请注意，我们还为我们的 MDB 提供了一些配置:

在这个例子中，我们的**接收者也产生一个确认，在这个意义上，它本身就是一个发送者**。它向另一个名为`ackQueue`的队列发送消息。

现在让我们通过一个测试来看看这一点:

```java
@Test
public void givenMDB_whenMessageSent_thenAcknowledgementReceived()
  throws InterruptedException, JMSException, NamingException {
    Connection connection = connectionFactory.createConnection();
    connection.start();
    Session session = connection.createSession(false, Session.AUTO_ACKNOWLEDGE);
    MessageProducer producer = session.createProducer(myQueue);
    producer.send(session.createTextMessage("marco"));
    MessageConsumer response = session.createConsumer(ackQueue);

    assertEquals("polo", ((TextMessage) response.receive(1000)).getText());
} 
```

这里**我们向`myQueue`发送了一条消息，这条消息由我们的`@MessageDriven`注释 POJO** 接收。然后这个 POJO 发送了一个确认，我们的测试收到了一个响应，名为`MessageConsumer`。

### 7.2.Spring JMS 示例

好了，现在是时候用 Spring 做同样的事情了！

首先，我们需要为此添加一些配置。我们需要用`@EnableJms`注释之前的`ApplicationConfig` 类，并添加一些 beans 来设置`JmsListenerContainerFactory`和`JmsTemplate`:

```java
@EnableJms
public class ApplicationConfig {

    @Bean
    public DefaultJmsListenerContainerFactory jmsListenerContainerFactory() {
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
        JmsTemplate template = new JmsTemplate(connectionFactory());
        template.setConnectionFactory(connectionFactory());
        return template;
    }
} 
```

接下来，我们需要一个`Producer`——一个简单的弹簧`Component`——它将向`myQueue`发送消息，并从`ackQueue`接收确认:

```java
@Component
public class Producer {
    @Autowired
    private JmsTemplate jmsTemplate;

    public void sendMessageToDefaultDestination(final String message) {
        jmsTemplate.convertAndSend("myQueue", message);
    }

    public String receiveAck() {
        return (String) jmsTemplate.receiveAndConvert("ackQueue");
    }
} 
```

然后，我们有一个带有注释为`@JmsListener`的方法的`Receiver` `Component`来异步接收来自`myQueue`的消息:

```java
@Component
public class Receiver {
    @Autowired
    private JmsTemplate jmsTemplate;

    @JmsListener(destination = "myQueue")
    public void receiveMessage(String msg) {
        sendAck();
    }

    private void sendAck() {
        jmsTemplate.convertAndSend("ackQueue", "polo");
    }
} 
```

它还作为发送者在`ackQueue`确认消息接收。

按照我们的惯例，让我们通过一个测试来验证这一点:

```java
@Test
public void givenJMSBean_whenMessageSent_thenAcknowledgementReceived() throws NamingException {
    Producer producer = context.getBean(Producer.class);
    producer.sendMessageToDefaultDestination("marco");

    assertEquals("polo", producer.receiveAck());
} 
```

在这个测试中，我们将`marco`发送到`myQueue`，并从`ackQueue`接收到`polo`作为确认，这与我们对 EJB 所做的一样。

这里需要注意的一点是 **Spring JMS 可以同步和异步发送/接收消息**。

## 8.结论

在本教程中，我们看到了 Spring 和 Enterprise Java Beans 的一对一对比。我们了解他们的历史和基本差异。

然后我们用简单的例子来演示 Spring Beans 和 EJB 的比较。不用说，**这仅仅是对这些技术能力的皮毛，还有很多东西需要进一步探索。**

此外，这些可能是相互竞争的技术，但这并不意味着它们不能共存。我们可以很容易地在 Spring 框架中集成 EJB。

和往常一样，源代码可以在 GitHub 上找到[。](https://web.archive.org/web/20220926183020/https://github.com/eugenp/tutorials/tree/master/spring-ejb-modules/ejb-beans)