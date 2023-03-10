# 解决 Spring 的“没有资格自动代理”警告

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-not-eligible-for-auto-proxying>

## 1.概观

在这个简短的教程中，我们将看到如何找到 Spring 的“`not eligible for auto-proxying`”消息的原因以及如何修复它。

首先，我们将创建一个简单的真实代码示例，它会在应用程序启动时显示消息。然后，我们来解释一下为什么会出现这种情况。

最后，我们将通过展示一个工作代码示例来给出问题的解决方案。

## 2.`“not eligible for auto proxying”`消息的原因

### 2.1.示例配置

在解释该消息的原因之前，让我们构建一个导致该消息在应用程序启动期间出现的示例。

首先，我们将创建一个定制的`RandomInt`注释。我们将使用它来注释应该插入指定范围内的随机整数的字段:

```java
@Retention(RetentionPolicy.RUNTIME)
public @interface RandomInt {
    int min();

    int max();
}
```

其次，让我们创建一个简单的 Spring 组件`DataCache`类。我们希望为缓存分配一个随机组，例如，该组可能用于支持分片。为此，我们将使用自定义注释对该字段进行注释:

```java
@Component
public class DataCache {
    @RandomInt(min = 2, max = 10)
    private int group;
    private String name;
}
```

现在，我们来看看`RandomIntGenerator`类。这是一个 Spring 组件，我们将使用它将随机的`int`值插入到由`RandomInt`注释标注的字段中:

```java
@Component
public class RandomIntGenerator {
    private Random random = new Random();
    private DataCache dataCache;

    public RandomIntGenerator(DataCache dataCache) {
        this.dataCache = dataCache;
    }

    public int generate(int min, int max) {
        return random.nextInt(max - min) + min;
    }
}
```

需要注意的是，我们通过[构造函数注入](/web/20220810012722/https://www.baeldung.com/constructor-injection-in-spring)将`DataCache`类自动连接到`RandomIntGenerator `中。

最后，让我们创建一个`RandomIntProcessor`类，它将负责查找用`RandomInt`注释标注的字段，并将随机值插入其中:

```java
public class RandomIntProcessor implements BeanPostProcessor {
    private final RandomIntGenerator randomIntGenerator;

    public RandomIntProcessor(RandomIntGenerator randomIntGenerator) {
        this.randomIntGenerator = randomIntGenerator;
    }

    @Override
    public Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException {
        Field[] fields = bean.getClass().getDeclaredFields();
        for (Field field : fields) {
            RandomInt injectRandomInt = field.getAnnotation(RandomInt.class);
            if (injectRandomInt != null) {
                int min = injectRandomInt.min();
                int max = injectRandomInt.max();
                int randomValue = randomIntGenerator.generate(min, max);
                field.setAccessible(true);
                ReflectionUtils.setField(field, bean, randomValue);
            }
        }
        return bean;
    }
}
```

在类初始化之前，它使用了一个`org.springframework.beans.factory.config.BeanPostProcessor` 接口的实现来访问带注释的字段。

### 2.2.测试我们的示例

即使一切编译正确，当我们运行我们的 Spring 应用程序并查看它的日志时，我们会看到一条由 Spring 的`BeanPostProcessorChecker`类生成的“`not eligible for auto proxying`”消息:

```java
INFO org.springframework.context.support.PostProcessorRegistrationDelegate$BeanPostProcessorChecker - Bean 'randomIntGenerator' of type [com.baeldung.autoproxying.RandomIntGenerator] is not eligible for getting processed by all BeanPostProcessors (for example: not eligible for auto-proxying)
```

此外，我们看到依赖于这种机制的`DataCache` bean 并没有按照我们的意图进行初始化:

```java
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(classes = {RandomIntProcessor.class, DataCache.class, RandomIntGenerator.class})
public class NotEligibleForAutoProxyingIntegrationTest {

    private RandomIntProcessor randomIntProcessor;

    @Autowired
    private DataCache dataCache;

    @Test
    public void givenAutowireInBeanPostProcessor_whenSpringContextInitialize_thenNotEligibleLogShouldShow() {
        assertEquals(0, dataCache.getGroup());
    }
}
```

然而，值得一提的是，即使消息出现，应用程序也不会崩溃。

### 2.3.分析原因

该警告是由`RandomIntProcessor` 类及其自动关联的依赖项引起的。实现`BeanPostProcessor `接口**的类在启动时被实例化，这是`ApplicationContext,` 在任何其他 beans 之前的特殊启动阶段的一部分。**

此外，AOP 自动代理机制也是一个`BeanPostProcessor` 接口`.`的实现。因此，无论是`BeanPostProcessor `实现还是它们直接引用的 beans 都不符合自动代理的条件。这意味着 Spring 使用 AOP 的特性，比如自动连接、安全性或事务性注释，在这些类中不会像预期的那样工作。

在我们的例子中，我们能够将`DataCache`实例自动连接到`RandomIntGenerator` 类中，没有任何问题`.` ，但是`,` group 字段没有填充随机整数。

## 3.如何修复错误

为了摆脱"`not eligible for auto proxying”` 消息，我们需要**打破`BeanPostProcessor`实现和它的 bean 依赖**之间的循环。在我们的例子中，我们需要告诉 IoC 容器惰性地初始化`RandomIntGenerator` bean。我们可以用弹簧的 [`Lazy`](/web/20220810012722/https://www.baeldung.com/spring-lazy-annotation) 标注:

```java
public class RandomIntProcessor implements BeanPostProcessor {
    private final RandomIntGenerator randomIntGenerator;

    @Lazy
    public RandomIntProcessor(RandomIntGenerator randomIntGenerator) {
        this.randomIntGenerator = randomIntGenerator;
    }

    @Override
    public Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException {
        //...
    }
}
```

当`RandomIntProcessor`在`postProcessBeforeInitialization`方法中请求时，Spring 初始化`RandomIntGenerator` bean。**此时，Spring 的 IoC 容器实例化了所有现有的也适合自动代理的 beans。**

事实上，如果我们运行我们的应用程序，我们不会在日志中看到“`not eligible for auto proxying”` 消息。此外，`DataCache` bean 将有一个用随机整数填充的组字段:

```java
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(classes = {RandomIntProcessor.class, DataCache.class, RandomIntGenerator.class})
public class NotEligibleForAutoProxyingIntegrationTest {

    private RandomIntProcessor randomIntProcessor;

    @Autowired
    private DataCache dataCache;

    @Test
    public void givenAutowireInBeanPostProcessor_whenSpringContextInitialize_thenGroupFieldShouldBePopulated() {
        assertNotEquals(0, dataCache.getGroup());
    }
}
```

## 4.结论

在本文中，我们学习了如何跟踪和修复 Spring 的`“not eligible for auto-proxying”`消息的原因。惰性初始化打破了 bean 构造过程中的依赖循环。

与往常一样，GitHub 上的示例代码[可用。](https://web.archive.org/web/20220810012722/https://github.com/eugenp/tutorials/tree/master/spring-core-5)