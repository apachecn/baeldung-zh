# 弹簧豆后处理器

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-beanpostprocessor>

## 1.概观

所以，在其他一些教程中，我们已经谈到了 [`BeanPostProcessor`](https://web.archive.org/web/20220627171921/https://www.baeldung.com/spring-annotation-bean-pre-processor) 。在本教程中，我们将使用 Guava 的`EventBus`在真实世界的例子中使用它们。

**Spring 的`BeanPostProcessor`给了我们 Spring bean 生命周期的钩子来修改它的配置。**

`BeanPostProcessor`允许直接修改 beans 本身。

在本教程中，我们将看一个这些类集成了 Guava 的`EventBus`的具体例子。

## 2.设置

首先，我们需要设置我们的环境。让我们将春天[上下文](https://web.archive.org/web/20220627171921/https://search.maven.org/search?q=g:org.springframework%20AND%20a:spring-context)，春天[表达式](https://web.archive.org/web/20220627171921/https://search.maven.org/search?q=g:org.springframework%20AND%20a:spring-expression)，以及[番石榴](https://web.archive.org/web/20220627171921/https://search.maven.org/search?q=g:com.google.guava%20AND%20a:guava)依赖项添加到我们的`pom.xml`:

```java
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-context</artifactId>
    <version>5.2.6.RELEASE</version>
</dependency>
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-expression</artifactId>
    <version>5.2.6.RELEASE</version>
</dependency>
<dependency>
    <groupId>com.google.guava</groupId>
    <artifactId>guava</artifactId>
    <version>31.0.1-jre</version>
</dependency>
```

接下来，我们来讨论一下我们的目标。

## 3.目标和实施

对于我们的第一个目标，我们希望**利用 Guava 的`EventBus`在系统的各个方面异步传递消息**。

接下来，我们希望在 bean 创建/销毁时自动注册和注销事件的对象，而不是使用由`EventBus`提供的手动方法。

所以，我们现在准备开始编码了！

我们的实现将包括 Guava 的`EventBus`的包装类、自定义标记注释、`BeanPostProcessor`、模型对象和从`EventBus`接收股票交易事件的 bean。此外，我们将创建一个测试用例来验证所需的功能。

### 3.1.`EventBus`包装纸

为了使用，我们将定义一个`EventBus`包装器来提供一些静态方法，以便为将由`BeanPostProcessor`使用的事件注册和注销 beans:

```java
public final class GlobalEventBus {

    public static final String GLOBAL_EVENT_BUS_EXPRESSION
      = "T(com.baeldung.postprocessor.GlobalEventBus).getEventBus()";

    private static final String IDENTIFIER = "global-event-bus";
    private static final GlobalEventBus GLOBAL_EVENT_BUS = new GlobalEventBus();
    private final EventBus eventBus = new AsyncEventBus(IDENTIFIER, Executors.newCachedThreadPool());

    private GlobalEventBus() {}

    public static GlobalEventBus getInstance() {
        return GlobalEventBus.GLOBAL_EVENT_BUS;
    }

    public static EventBus getEventBus() {
        return GlobalEventBus.GLOBAL_EVENT_BUS.eventBus;
    }

    public static void subscribe(Object obj) {
        getEventBus().register(obj);
    }
    public static void unsubscribe(Object obj) {
        getEventBus().unregister(obj);
    }
    public static void post(Object event) {
        getEventBus().post(event);
    }
}
```

这段代码提供了静态方法来访问`GlobalEventBus`和底层`EventBus`，以及注册和注销事件和发布事件。它还有一个 SpEL 表达式，在我们的自定义注释中用作默认表达式，以定义我们想要使用哪个`EventBus`。

### 3.2.自定义标记注释

接下来，让我们定义一个定制的标记注释，它将被`BeanPostProcessor`用来识别 beans 以自动注册/取消注册事件:

```java
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.TYPE)
@Inherited
public @interface Subscriber {
    String value() default GlobalEventBus.GLOBAL_EVENT_BUS_EXPRESSION;
}
```

### 3.3.`BeanPostProcessor`

现在，我们将定义`BeanPostProcessor`，它将检查每个 bean 的`Subscriber`注释。这个类也是一个`DestructionAwareBeanPostProcessor,` ，它是一个 Spring 接口，为`BeanPostProcessor`添加了一个销毁前回调。如果注释存在，我们将在 bean 初始化时用注释的 SpEL 表达式标识的`EventBus`注册它，并在 bean 销毁时注销它:

```java
public class GuavaEventBusBeanPostProcessor
  implements DestructionAwareBeanPostProcessor {

    Logger logger = LoggerFactory.getLogger(this.getClass());
    SpelExpressionParser expressionParser = new SpelExpressionParser();

    @Override
    public void postProcessBeforeDestruction(Object bean, String beanName)
      throws BeansException {
        this.process(bean, EventBus::unregister, "destruction");
    }

    @Override
    public boolean requiresDestruction(Object bean) {
        return true;
    }

    @Override
    public Object postProcessBeforeInitialization(Object bean, String beanName)
      throws BeansException {
        return bean;
    }

    @Override
    public Object postProcessAfterInitialization(Object bean, String beanName)
      throws BeansException {
        this.process(bean, EventBus::register, "initialization");
        return bean;
    }

    private void process(Object bean, BiConsumer<EventBus, Object> consumer, String action) {
       // See implementation below
    }
}
```

上面的代码获取每个 bean 并通过下面定义的`process`方法运行它。它在 bean 初始化之后、销毁之前对其进行处理。默认情况下，`requiresDestruction`方法返回 true，我们在这里保持这种行为，因为我们在`postProcessBeforeDestruction`回调中检查了`@Subscriber`注释的存在。

现在让我们来看看`process`方法:

```java
private void process(Object bean, BiConsumer<EventBus, Object> consumer, String action) {
    Object proxy = this.getTargetObject(bean);
    Subscriber annotation = AnnotationUtils.getAnnotation(proxy.getClass(), Subscriber.class);
    if (annotation == null)
        return;
    this.logger.info("{}: processing bean of type {} during {}",
      this.getClass().getSimpleName(), proxy.getClass().getName(), action);
    String annotationValue = annotation.value();
    try {
        Expression expression = this.expressionParser.parseExpression(annotationValue);
        Object value = expression.getValue();
        if (!(value instanceof EventBus)) {
            this.logger.error(
              "{}: expression {} did not evaluate to an instance of EventBus for bean of type {}",
              this.getClass().getSimpleName(), annotationValue, proxy.getClass().getSimpleName());
            return;
        }
        EventBus eventBus = (EventBus)value;
        consumer.accept(eventBus, proxy);
    } catch (ExpressionException ex) {
        this.logger.error("{}: unable to parse/evaluate expression {} for bean of type {}",
          this.getClass().getSimpleName(), annotationValue, proxy.getClass().getName());
    }
}
```

这段代码检查名为`Subscriber`的自定义标记注释是否存在，如果存在，则从其`value`属性中读取 SpEL 表达式。然后，表达式被计算为一个对象。如果它是`EventBus,`的实例，我们将`BiConsumer`函数参数应用于 bean。`BiConsumer`用于注册和注销`EventBus`中的 bean。

方法`getTargetObject`的实现如下:

```java
private Object getTargetObject(Object proxy) throws BeansException {
    if (AopUtils.isJdkDynamicProxy(proxy)) {
        try {
            return ((Advised)proxy).getTargetSource().getTarget();
        } catch (Exception e) {
            throw new FatalBeanException("Error getting target of JDK proxy", e);
        }
    }
    return proxy;
}
```

### 3.4.`StockTrade`模型对象

接下来，让我们定义我们的`StockTrade`模型对象:

```java
public class StockTrade {

    private String symbol;
    private int quantity;
    private double price;
    private Date tradeDate;

    // constructor
}
```

### 3.5.`StockTradePublisher`事件接收者

然后，让我们定义一个监听器类来通知我们收到了一个交易，这样我们就可以编写我们的测试了:

```java
@FunctionalInterface
public interface StockTradeListener {
    void stockTradePublished(StockTrade trade);
}
```

最后，我们将为新的`StockTrade`事件定义一个接收器:

```java
@Subscriber
public class StockTradePublisher {

    Set<StockTradeListener> stockTradeListeners = new HashSet<>();

    public void addStockTradeListener(StockTradeListener listener) {
        synchronized (this.stockTradeListeners) {
            this.stockTradeListeners.add(listener);
        }
    }

    public void removeStockTradeListener(StockTradeListener listener) {
        synchronized (this.stockTradeListeners) {
            this.stockTradeListeners.remove(listener);
        }
    }

    @Subscribe
    @AllowConcurrentEvents
    void handleNewStockTradeEvent(StockTrade trade) {
        // publish to DB, send to PubNub, ...
        Set<StockTradeListener> listeners;
        synchronized (this.stockTradeListeners) {
            listeners = new HashSet<>(this.stockTradeListeners);
        }
        listeners.forEach(li -> li.stockTradePublished(trade));
    }
}
```

上面的代码将这个类标记为 Guava `EventBus`事件的`Subscriber`，Guava 的`@Subscribe`注释将方法`handleNewStockTradeEvent`标记为事件的接收者。它将接收的事件类型基于该方法的单个参数的类；在这种情况下，我们将接收类型为`StockTrade`的事件。

`@AllowConcurrentEvents`注释允许并发调用这个方法。一旦我们收到交易，我们做任何我们想要的处理，然后通知任何听众。

### 3.6.测试

现在让我们用一个集成测试来结束我们的编码，以验证`BeanPostProcessor`工作正常。首先，我们需要一个 Spring 上下文:

```java
@Configuration
public class PostProcessorConfiguration {

    @Bean
    public GlobalEventBus eventBus() {
        return GlobalEventBus.getInstance();
    }

    @Bean
    public GuavaEventBusBeanPostProcessor eventBusBeanPostProcessor() {
        return new GuavaEventBusBeanPostProcessor();
    }

    @Bean
    public StockTradePublisher stockTradePublisher() {
        return new StockTradePublisher();
    }
}
```

现在我们可以实现我们的测试了:

```java
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(classes = PostProcessorConfiguration.class)
public class StockTradeIntegrationTest {

    @Autowired
    StockTradePublisher stockTradePublisher;

    @Test
    public void givenValidConfig_whenTradePublished_thenTradeReceived() {
        Date tradeDate = new Date();
        StockTrade stockTrade = new StockTrade("AMZN", 100, 2483.52d, tradeDate);
        AtomicBoolean assertionsPassed = new AtomicBoolean(false);
        StockTradeListener listener = trade -> assertionsPassed
          .set(this.verifyExact(stockTrade, trade));
        this.stockTradePublisher.addStockTradeListener(listener);
        try {
            GlobalEventBus.post(stockTrade);
            await().atMost(Duration.ofSeconds(2L))
              .untilAsserted(() -> assertThat(assertionsPassed.get()).isTrue());
        } finally {
            this.stockTradePublisher.removeStockTradeListener(listener);
        }
    }

    boolean verifyExact(StockTrade stockTrade, StockTrade trade) {
        return Objects.equals(stockTrade.getSymbol(), trade.getSymbol())
          && Objects.equals(stockTrade.getTradeDate(), trade.getTradeDate())
          && stockTrade.getQuantity() == trade.getQuantity()
          && stockTrade.getPrice() == trade.getPrice();
    }
}
```

上面的测试代码生成一个股票交易，并将其发送到`GlobalEventBus`。我们最多等待两秒钟，等待操作完成并被通知交易已被`stockTradePublisher`收到。此外，我们确认收到的交易在运输过程中没有被修改。

## 4.结论

总之，Spring 的`BeanPostProcessor`允许我们**定制 bean 本身**，为我们提供了一种自动化 bean 操作的方法，否则我们必须手动完成。

和往常一样，源代码可以在 GitHub 上找到[。](https://web.archive.org/web/20220627171921/https://github.com/eugenp/tutorials/tree/master/spring-core-4)