# Spring Data Redis Reactive 简介

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-data-redis-reactive>

## 1。简介

在本教程中，**我们将学习如何使用 Spring Data 的`ReactiveRedisTemplate. `** 来配置和实现 Redis 操作

我们将回顾一下`ReactiveRedisTemplate` 的基本用法，比如如何在 Redis 中存储和检索对象。我们将看看如何使用`ReactiveRedisConnection`执行 Redis 命令。

为了涵盖基础知识，请查看我们的[对 Spring Data Redis](/web/20220625221934/https://www.baeldung.com/spring-data-redis-tutorial) 的介绍。

## 2。设置

要在我们的代码中使用`ReactiveRedisTemplate `，首先，我们需要为 Spring Boot 的 Redis Reactive 模块添加[依赖项:](https://web.archive.org/web/20220625221934/https://search.maven.org/classic/#search%7Cga%7C1%7Ca%3A%22spring-boot-starter-data-redis-reactive%22)

```java
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-redis-reactive</artifactId>
</dependency>
```

## 3。配置

然后我们需要建立与 Redis 服务器的连接。如果想在`localhost:6379`连接到 Redis 服务器，我们不需要添加任何配置代码。

但是，**如果我们的服务器是远程的或者在不同的端口上，我们可以在`LettuceConnectionFactory `构造函数:**中提供主机名和端口

```java
@Bean
public ReactiveRedisConnectionFactory reactiveRedisConnectionFactory() {
    return new LettuceConnectionFactory(host, port);
}
```

## 4。列表操作

Redis 列表是按插入顺序排序的字符串列表。我们可以通过从左侧或右侧推动或弹出来添加或删除列表中的元素。

### 4.1。字符串模板

为了处理列表，我们需要一个 **`ReactiveStringRedisTemplate`的实例来获取对*redist operations***的引用:

```java
@Autowired
private ReactiveStringRedisTemplate redisTemplate;
private ReactiveListOperations<String, String> reactiveListOps;
@Before
public void setup() {
    reactiveListOps = redisTemplate.opsForList();
}
```

### 4.2。LPUSH 和 LPOP

现在我们有了一个`ReactiveListOperations,` 的实例，让我们用`demo_list`作为列表的标识符对列表进行 LPUSH 操作。

之后，我们将对列表进行 LPOP，然后验证弹出的元素:

```java
@Test
public void givenListAndValues_whenLeftPushAndLeftPop_thenLeftPushAndLeftPop() {
    Mono<Long> lPush = reactiveListOps.leftPushAll(LIST_NAME, "first", "second")
      .log("Pushed");
    StepVerifier.create(lPush)
      .expectNext(2L)
      .verifyComplete();
    Mono<String> lPop = reactiveListOps.leftPop(LIST_NAME)
      .log("Popped");
    StepVerifier.create(lPop)
      .expectNext("second")
      .verifyComplete();
}
```

注意，在测试电抗元件时，我们可以使用`StepVerifier` 来阻止任务的完成。

## 5。数值运算

我们可能也想使用自定义对象，而不仅仅是字符串。

因此，让我们在一个`Employee`对象上做一些类似的操作来演示我们在 POJO 上的操作:

```java
public class Employee implements Serializable {
    private String id;
    private String name;
    private String department;
    // ... getters and setters
    // ... hashCode and equals
}
```

### 5.1。员工模板

我们需要创建`ReactiveRedisTemplate.` 的第二个实例，我们仍然使用`String `作为我们的键，但是这一次值将是`Employee`:

```java
@Bean
public ReactiveRedisTemplate<String, Employee> reactiveRedisTemplate(
  ReactiveRedisConnectionFactory factory) {
    StringRedisSerializer keySerializer = new StringRedisSerializer();
    Jackson2JsonRedisSerializer<Employee> valueSerializer =
      new Jackson2JsonRedisSerializer<>(Employee.class);
    RedisSerializationContext.RedisSerializationContextBuilder<String, Employee> builder =
      RedisSerializationContext.newSerializationContext(keySerializer);
    RedisSerializationContext<String, Employee> context = 
      builder.value(valueSerializer).build();
    return new ReactiveRedisTemplate<>(factory, context);
}
```

为了正确地序列化定制对象，我们需要指导 Spring 如何做。这里，我们告诉模板**通过为值**配置一个 `**Jackson2JsonRedisSerializer**` **来使用 Jackson 库。因为密钥只是一个字符串，所以我们可以使用`StringRedisSerializer `来实现它。**

然后，我们像以前一样利用这个序列化上下文和我们的连接工厂来创建一个模板。

接下来，我们将创建一个 `ReactiveValueOperations` 的实例，就像我们之前创建`ReactiveListOperations`一样:

```java
@Autowired
private ReactiveRedisTemplate<String, Employee> redisTemplate;
private ReactiveValueOperations<String, Employee> reactiveValueOps;
@Before
public void setup() {
    reactiveValueOps = redisTemplate.opsForValue();
}
```

### 5.2。保存和检索操作

现在我们有了一个`ReactiveValueOperations, `的实例，让我们用它来存储一个`Employee`的实例:

```java
@Test
public void givenEmployee_whenSet_thenSet() {
    Mono<Boolean> result = reactiveValueOps.set("123", 
      new Employee("123", "Bill", "Accounts"));
    StepVerifier.create(result)
      .expectNext(true)
      .verifyComplete();
}
```

然后我们可以从 Redis 获得相同的对象:

```java
@Test
public void givenEmployeeId_whenGet_thenReturnsEmployee() {
    Mono<Employee> fetchedEmployee = reactiveValueOps.get("123");
    StepVerifier.create(fetchedEmployee)
      .expectNext(new Employee("123", "Bill", "Accounts"))
      .verifyComplete();
}
```

### 5.3。有到期时间的操作

我们经常希望**将值放入一个将自然过期的缓存中**，我们可以通过相同的`set `操作来实现这一点:

```java
@Test
public void givenEmployee_whenSetWithExpiry_thenSetsWithExpiryTime() 
  throws InterruptedException {
    Mono<Boolean> result = reactiveValueOps.set("129", 
      new Employee("129", "John", "Programming"), 
      Duration.ofSeconds(1));
    StepVerifier.create(result)
      .expectNext(true)
      .verifyComplete();
    Thread.sleep(2000L); 
    Mono<Employee> fetchedEmployee = reactiveValueOps.get("129");
    StepVerifier.create(fetchedEmployee)
      .expectNextCount(0L)
      .verifyComplete();
}
```

注意，这个测试自己做一些阻塞来等待缓存键过期。

## 6。再命令〔t1〕

Redis 命令基本上是 Redis 客户机可以在服务器上调用的方法。Redis 支持几十个命令，其中一些我们已经见过了，比如 LPUSH 和 LPOP。

**`Operations `API 是围绕 Redis 的命令集的高级抽象。**

但是，**如果我们想更直接的使用 Redis 命令原语，那么 Spring Data Redis Reactive 也给了我们一个`Commands ` API。**

所以，让我们通过`Commands` API 的镜头来看看字符串和键盘命令。

### 6.1。字符串和键盘命令

为了执行 Redis 命令操作，我们将获得`ReactiveKeyCommands`和`ReactiveStringCommands.`的实例

我们可以从我们的`ReactiveRedisConnectionFactory`实例中获得它们:

```java
@Bean
public ReactiveKeyCommands keyCommands(ReactiveRedisConnectionFactory 
  reactiveRedisConnectionFactory) {
    return reactiveRedisConnectionFactory.getReactiveConnection().keyCommands();
}
@Bean
public ReactiveStringCommands stringCommands(ReactiveRedisConnectionFactory 
  reactiveRedisConnectionFactory) {
    return reactiveRedisConnectionFactory.getReactiveConnection().stringCommands();
}
```

### 6.2。设置和获取操作

我们可以使用`ReactiveStringCommands`通过一次调用来存储多个键，**基本上是多次调用 SET 命令**。

然后，我们可以通过`ReactiveKeyCommands`、**调用按键命令**来检索这些按键:

```java
@Test
public void givenFluxOfKeys_whenPerformOperations_thenPerformOperations() {
    Flux<SetCommand> keys = Flux.just("key1", "key2", "key3", "key4");
      .map(String::getBytes)
      .map(ByteBuffer::wrap)
      .map(key -> SetCommand.set(key).value(key));
    StepVerifier.create(stringCommands.set(keys))
      .expectNextCount(4L)
      .verifyComplete();
    Mono<Long> keyCount = keyCommands.keys(ByteBuffer.wrap("key*".getBytes()))
      .flatMapMany(Flux::fromIterable)
      .count();
    StepVerifier.create(keyCount)
      .expectNext(4L)
      .verifyComplete();
}
```

注意，如前所述，这个 API 要低级得多。例如，**不是处理高层对象，而是发送一个字节流，使用`ByteBuffer`** 。此外，我们使用更多的 Redis 原语，如 SET 和 SCAN。

最后，String 和 Key 命令只是 Spring Data Redis 被动公开的众多命令接口中的两个。

## 7。结论

在本教程中，我们介绍了使用 Spring Data 的 Reactive Redis 模板的基础知识，以及将它与应用程序集成的各种方法。

GitHub 上的[提供了示例的完整源代码。](https://web.archive.org/web/20220625221934/https://github.com/eugenp/tutorials/tree/master/persistence-modules/spring-data-redis)