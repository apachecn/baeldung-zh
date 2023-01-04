# Dubbo 简介

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/dubbo>

## 1。简介

[Dubbo](https://web.archive.org/web/20220628152926/http://dubbo.io/) 是阿里巴巴的开源 RPC 和微服务框架。

除此之外，它有助于增强服务治理，并使传统的整体应用程序可以平滑地重构为可伸缩的分布式架构。

在本文中，我们将介绍 Dubbo 及其最重要的特性。

## 2。架构

Dubbo 区分了几个角色:

1.  提供者——公开服务的地方；提供者将向注册中心注册其服务
2.  容器——启动、加载和运行服务的地方
3.  消费者——谁调用远程服务；消费者将订阅注册中心所需的服务
4.  注册中心——注册和发现服务的地方
5.  monitor——记录服务的统计数据，例如，给定时间间隔内服务调用的频率

[![Dubbo Architecture](img/44d10942deb397bb43c2313d4c1edea6.png)](/web/20220628152926/https://www.baeldung.com/wp-content/uploads/2017/12/dubbo-architecture-1024x639.png)

`(source: http://dubbo.img/dubbo-architecture.png)`

提供者、消费者和注册中心之间的连接是持久的，因此每当服务提供者停机时，注册中心可以检测到故障并通知消费者。

注册表和监视器是可选的。消费者可以直接连接到服务提供商，但整个系统的稳定性会受到影响。

## 3。Maven 依赖关系

在我们开始之前，让我们将下面的依赖项添加到我们的`pom.xml`中:

```java
<dependency>
    <groupId>com.alibaba</groupId>
    <artifactId>dubbo</artifactId>
    <version>2.5.7</version>
</dependency>
```

最新版本可以在这里找到[。](https://web.archive.org/web/20220628152926/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22com.alibaba%22%20AND%20a%3A%22dubbo%22)

## 4。自举

现在让我们尝试一下 Dubbo 的基本特性。

这是一个微创框架，它的许多特性依赖于外部配置或注释。

官方建议我们应该使用 XML 配置文件，因为它依赖于 Spring 容器(目前是 Spring 4.3.10)。

我们将使用 XML 配置演示它的大部分特性。

### 4.1。多播注册中心-服务提供商

首先，我们只需要一个服务提供商、一个消费者和一个“看不见的”注册中心。注册表是不可见的，因为我们使用的是多播网络。

在下面的例子中，提供者只对其消费者说“嗨”:

```java
public interface GreetingsService {
    String sayHi(String name);
}

public class GreetingsServiceImpl implements GreetingsService {

    @Override
    public String sayHi(String name) {
        return "hi, " + name;
    }
}
```

为了进行远程过程调用，消费者必须与服务提供者共享一个公共接口，因此接口`GreetingsService`必须与消费者共享。

### 4.2。多播注册-服务注册

现在让我们将`GreetingsService`注册到注册表中。如果提供者和消费者都在同一个本地网络上，一种非常方便的方法是使用多播注册表:

```java
<dubbo:application name="demo-provider" version="1.0"/>
<dubbo:registry address="multicast://224.1.1.1:9090"/>
<dubbo:protocol name="dubbo" port="20880"/>
<bean id="greetingsService" class="com.baeldung.dubbo.remote.GreetingsServiceImpl"/>
<dubbo:service interface="com.baeldung.dubbo.remote.GreetingsService"
  ref="greetingsService"/>
```

使用上面的 beans 配置，我们刚刚将我们的`GreetingsService`暴露给了一个位于`dubbo://127.0.0.1:20880`下的 url，并将服务注册到了一个在`<dubbo:registry />`中指定的组播地址。

在提供者的配置中，我们还分别通过`<dubbo:application />`、`<dubbo:service />`和`<beans />`声明了我们的应用元数据、要发布的接口及其实现。

`dubbo`协议是该框架支持的众多协议之一。它建立在 Java NIO 非阻塞特性之上，并且是使用的默认协议。

我们将在本文后面更详细地讨论它。

### 4.3。多播注册——服务消费者

通常，消费者需要指定要调用的接口和远程服务的地址，这正是消费者所需要的:

```java
<dubbo:application name="demo-consumer" version="1.0"/>
<dubbo:registry address="multicast://224.1.1.1:9090"/>
<dubbo:reference interface="com.baeldung.dubbo.remote.GreetingsService"
  id="greetingsService"/>
```

现在一切都设置好了，让我们看看它们是如何工作的:

```java
public class MulticastRegistryTest {

    @Before
    public void initRemote() {
        ClassPathXmlApplicationContext remoteContext
          = new ClassPathXmlApplicationContext("multicast/provider-app.xml");
        remoteContext.start();
    }

    @Test
    public void givenProvider_whenConsumerSaysHi_thenGotResponse(){
        ClassPathXmlApplicationContext localContext 
          = new ClassPathXmlApplicationContext("multicast/consumer-app.xml");
        localContext.start();
        GreetingsService greetingsService
          = (GreetingsService) localContext.getBean("greetingsService");
        String hiMessage = greetingsService.sayHi("baeldung");

        assertNotNull(hiMessage);
        assertEquals("hi, baeldung", hiMessage);
    }
}
```

当提供者的`remoteContext`启动时，Dubbo 会自动加载`GreetingsService`并将其注册到给定的注册表中。在这种情况下，它是一个多播注册表。

消费者订阅多播注册中心，并在上下文中创建一个代理`GreetingsService`。当我们的本地客户机调用`sayHi`方法时，它透明地调用远程服务。

我们提到注册中心是可选的，这意味着消费者可以通过公开的端口直接连接到提供者:

```java
<dubbo:reference interface="com.baeldung.dubbo.remote.GreetingsService"
  id="greetingsService" url="dubbo://127.0.0.1:20880"/>
```

基本上，这个过程类似于传统的 web 服务，但是 Dubbo 只是把它变得简单明了。

### 4.4。简单注册表

注意，当使用“不可见的”多播注册表时，注册表服务不是独立的。但是，它只适用于受限的本地网络。

为了明确地设置一个可管理的注册表，我们可以使用一个 [`SimpleRegistryService`](https://web.archive.org/web/20220628152926/https://github.com/apache/incubator-dubbo/blob/master/dubbo-registry/dubbo-registry-default/src/test/java/org/apache/dubbo/registry/dubbo/SimpleRegistryService.java) 。

将以下 beans 配置加载到 Spring context 后，将启动一个简单的注册服务:

```java
<dubbo:application name="simple-registry" />
<dubbo:protocol port="9090" />
<dubbo:service interface="com.alibaba.dubbo.registry.RegistryService"
  ref="registryService" registry="N/A" ondisconnect="disconnect">
    <dubbo:method name="subscribe">
        <dubbo:argument index="1" callback="true" />
    </dubbo:method>
    <dubbo:method name="unsubscribe">
        <dubbo:argument index="1" callback="true" />
    </dubbo:method>
</dubbo:service>

<bean class="com.alibaba.dubbo.registry.simple.SimpleRegistryService"
  id="registryService" />
```

注意，`SimpleRegistryService`类不包含在工件中，所以我们直接从 Github 库中复制了[源代码](https://web.archive.org/web/20220628152926/https://github.com/apache/incubator-dubbo/blob/master/dubbo-registry/dubbo-registry-default/src/test/java/org/apache/dubbo/registry/dubbo/SimpleRegistryService.java)。

然后，我们将调整提供者和消费者的注册中心配置:

```java
<dubbo:registry address="127.0.0.1:9090"/>
```

`SimpleRegistryService`测试时可作为独立注册表使用，但不建议在生产环境中使用。

### 4.5。Java 配置

还支持通过 Java API、属性文件和注释进行配置。然而，属性文件和注释只有在我们架构不是很复杂的情况下才适用。

让我们看看如何将我们以前的多播注册中心的 XML 配置转换成 API 配置。首先，提供程序的设置如下:

```java
ApplicationConfig application = new ApplicationConfig();
application.setName("demo-provider");
application.setVersion("1.0");

RegistryConfig registryConfig = new RegistryConfig();
registryConfig.setAddress("multicast://224.1.1.1:9090");

ServiceConfig<GreetingsService> service = new ServiceConfig<>();
service.setApplication(application);
service.setRegistry(registryConfig);
service.setInterface(GreetingsService.class);
service.setRef(new GreetingsServiceImpl());

service.export();
```

现在服务已经通过多播注册中心公开了，让我们在本地客户机中使用它:

```java
ApplicationConfig application = new ApplicationConfig();
application.setName("demo-consumer");
application.setVersion("1.0");

RegistryConfig registryConfig = new RegistryConfig();
registryConfig.setAddress("multicast://224.1.1.1:9090");

ReferenceConfig<GreetingsService> reference = new ReferenceConfig<>();
reference.setApplication(application);
reference.setRegistry(registryConfig);
reference.setInterface(GreetingsService.class);

GreetingsService greetingsService = reference.get();
String hiMessage = greetingsService.sayHi("baeldung");
```

虽然上面的代码片段像前面的 XML 配置示例一样有魅力，但是它稍微有点琐碎。就目前而言，如果我们打算充分利用 Dubbo，XML 配置应该是首选。

## 5。协议支持

该框架支持多种协议，包括`dubbo`、`RMI`、`hessian`、`HTTP`、`web service`、`thrift`、`memcached`和`redis`。大多数协议看起来都很熟悉，除了`dubbo`。让我们看看这个协议有什么新内容。

`dubbo`协议保持了提供者和消费者之间的持久连接。长连接和 NIO 无阻塞网络通信使得在传输小规模数据包(< 100K)时性能相当出色。

有几个可配置的属性，如端口、每个用户的连接数、最大接受连接数等。

```java
<dubbo:protocol name="dubbo" port="20880"
  connections="2" accepts="1000" />
```

Dubbo 还支持同时通过不同的协议公开服务:

```java
<dubbo:protocol name="dubbo" port="20880" />
<dubbo:protocol name="rmi" port="1099" />

<dubbo:service interface="com.baeldung.dubbo.remote.GreetingsService"
  version="1.0.0" ref="greetingsService" protocol="dubbo" />
<dubbo:service interface="com.bealdung.dubbo.remote.AnotherService"
  version="1.0.0" ref="anotherService" protocol="rmi" />
```

是的，我们可以使用不同的协议公开不同的服务，如上面的代码片段所示。底层传输器、序列化实现和其他与网络相关的公共属性也是可配置的。

## 6。结果缓存

本机支持远程结果缓存，以加快对热数据的访问。这就像向 bean 引用添加一个缓存属性一样简单:

```java
<dubbo:reference interface="com.baeldung.dubbo.remote.GreetingsService"
  id="greetingsService" cache="lru" />
```

这里我们配置了一个最近最少使用的缓存。为了验证缓存行为，我们将对前面的标准实现稍作修改(我们称之为“特殊实现”):

```java
public class GreetingsServiceSpecialImpl implements GreetingsService {
    @Override
    public String sayHi(String name) {
        try {
            SECONDS.sleep(5);
        } catch (Exception ignored) { }
        return "hi, " + name;
    }
}
```

启动 provider 后，我们可以在使用者端验证在多次调用时结果是否被缓存:

```java
@Test
public void givenProvider_whenConsumerSaysHi_thenGotResponse() {
    ClassPathXmlApplicationContext localContext
      = new ClassPathXmlApplicationContext("multicast/consumer-app.xml");
    localContext.start();
    GreetingsService greetingsService
      = (GreetingsService) localContext.getBean("greetingsService");

    long before = System.currentTimeMillis();
    String hiMessage = greetingsService.sayHi("baeldung");

    long timeElapsed = System.currentTimeMillis() - before;
    assertTrue(timeElapsed > 5000);
    assertNotNull(hiMessage);
    assertEquals("hi, baeldung", hiMessage);

    before = System.currentTimeMillis();
    hiMessage = greetingsService.sayHi("baeldung");
    timeElapsed = System.currentTimeMillis() - before;

    assertTrue(timeElapsed < 1000);
    assertNotNull(hiMessage);
    assertEquals("hi, baeldung", hiMessage);
}
```

这里，消费者调用特殊的服务实现，所以第一次完成调用花费了 5 秒多的时间。当我们再次调用时，`sayHi`方法几乎立即完成，因为结果是从缓存中返回的。

注意，也支持线程本地缓存和 JCache。

## 7 .**。集群支持**

Dubbo 通过其负载平衡能力和多种容错策略帮助我们自由扩展服务。这里，让我们假设我们用 Zookeeper 作为注册中心来管理集群中的服务。提供商可以像这样在 Zookeeper 中注册他们的服务:

```java
<dubbo:registry address="zookeeper://127.0.0.1:2181"/>
```

请注意，我们在`POM`中需要这些额外的依赖项:

```java
<dependency>
    <groupId>org.apache.zookeeper</groupId>
    <artifactId>zookeeper</artifactId>
    <version>3.4.11</version>
</dependency>
<dependency>
    <groupId>com.101tec</groupId>
    <artifactId>zkclient</artifactId>
    <version>0.10</version>
</dependency>
```

最新版本的`zookeeper`依赖和`zkclient`可以在[这里](https://web.archive.org/web/20220628152926/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22org.apache.zookeeper%22%20AND%20a%3A%22zookeeper%22)和[这里](https://web.archive.org/web/20220628152926/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22com.101tec%22%20AND%20a%3A%22zkclient%22)找到。

### 7.1。负载平衡

目前，该框架支持一些负载平衡策略:

*   随意
*   循环赛
*   最不活跃的
*   一致哈希。

在下面的例子中，我们有两个服务实现作为集群中的提供者。使用循环法路由请求。

首先，让我们设置服务提供商:

```java
@Before
public void initRemote() {
    ExecutorService executorService = Executors.newFixedThreadPool(2);
    executorService.submit(() -> {
        ClassPathXmlApplicationContext remoteContext 
          = new ClassPathXmlApplicationContext("cluster/provider-app-default.xml");
        remoteContext.start();
    });
    executorService.submit(() -> {
        ClassPathXmlApplicationContext backupRemoteContext
          = new ClassPathXmlApplicationContext("cluster/provider-app-special.xml");
        backupRemoteContext.start();
    });
}
```

现在，我们有了一个标准的“快速提供者”来立即响应，还有一个特殊的“慢速提供者”，它对每个请求都会休眠 5 秒钟。

使用循环策略运行 6 次后，我们预计平均响应时间至少为 2.5 秒:

```java
@Test
public void givenProviderCluster_whenConsumerSaysHi_thenResponseBalanced() {
    ClassPathXmlApplicationContext localContext
      = new ClassPathXmlApplicationContext("cluster/consumer-app-lb.xml");
    localContext.start();
    GreetingsService greetingsService
      = (GreetingsService) localContext.getBean("greetingsService");

    List<Long> elapseList = new ArrayList<>(6);
    for (int i = 0; i < 6; i++) {
        long current = System.currentTimeMillis();
        String hiMessage = greetingsService.sayHi("baeldung");
        assertNotNull(hiMessage);
        elapseList.add(System.currentTimeMillis() - current);
    }

    OptionalDouble avgElapse = elapseList
      .stream()
      .mapToLong(e -> e)
      .average();
    assertTrue(avgElapse.isPresent());
    assertTrue(avgElapse.getAsDouble() > 2500.0);
}
```

此外，还采用了动态负载平衡。下一个例子演示了使用循环策略，当新的服务提供者上线时，消费者自动选择新的服务提供者作为候选者。

“慢速提供者”在系统启动 2 秒钟后注册:

```java
@Before
public void initRemote() {
    ExecutorService executorService = Executors.newFixedThreadPool(2);
    executorService.submit(() -> {
        ClassPathXmlApplicationContext remoteContext
          = new ClassPathXmlApplicationContext("cluster/provider-app-default.xml");
        remoteContext.start();
    });
    executorService.submit(() -> {
        SECONDS.sleep(2);
        ClassPathXmlApplicationContext backupRemoteContext
          = new ClassPathXmlApplicationContext("cluster/provider-app-special.xml");
        backupRemoteContext.start();
        return null;
    });
}
```

消费者每秒钟调用一次远程服务。运行 6 次后，我们预计平均响应时间将大于 1.6 秒:

```java
@Test
public void givenProviderCluster_whenConsumerSaysHi_thenResponseBalanced()
  throws InterruptedException {
    ClassPathXmlApplicationContext localContext
      = new ClassPathXmlApplicationContext("cluster/consumer-app-lb.xml");
    localContext.start();
    GreetingsService greetingsService
      = (GreetingsService) localContext.getBean("greetingsService");
    List<Long> elapseList = new ArrayList<>(6);
    for (int i = 0; i < 6; i++) {
        long current = System.currentTimeMillis();
        String hiMessage = greetingsService.sayHi("baeldung");
        assertNotNull(hiMessage);
        elapseList.add(System.currentTimeMillis() - current);
        SECONDS.sleep(1);
    }

    OptionalDouble avgElapse = elapseList
      .stream()
      .mapToLong(e -> e)
      .average();

    assertTrue(avgElapse.isPresent());
    assertTrue(avgElapse.getAsDouble() > 1666.0);
}
```

请注意，负载平衡器既可以在使用者端配置，也可以在提供者端配置。下面是一个消费者端配置的例子:

```java
<dubbo:reference interface="com.baeldung.dubbo.remote.GreetingsService"
  id="greetingsService" loadbalance="roundrobin" />
```

### 7.2。容错能力

Dubbo 支持多种容错策略，包括:

*   故障转移
*   自动防故障装置
*   快速失效
*   故障恢复
*   分叉。

在故障转移的情况下，当一个提供者失败时，消费者可以尝试集群中的其他服务提供者。

服务提供商的容错策略配置如下:

```java
<dubbo:service interface="com.baeldung.dubbo.remote.GreetingsService"
  ref="greetingsService" cluster="failover"/>
```

为了实际演示服务故障转移，让我们创建一个`GreetingsService`的故障转移实现:

```java
public class GreetingsFailoverServiceImpl implements GreetingsService {

    @Override
    public String sayHi(String name) {
        return "hi, failover " + name;
    }
}
```

我们可以回忆一下，我们的特殊服务实现`GreetingsServiceSpecialImpl`为每个请求休眠 5 秒钟。

当任何超过 2 秒的响应被视为消费者的请求失败时，我们有一个故障转移场景:

```java
<dubbo:reference interface="com.baeldung.dubbo.remote.GreetingsService"
  id="greetingsService" retries="2" timeout="2000" />
```

启动两个提供者后，我们可以用下面的代码片段验证故障转移行为:

```java
@Test
public void whenConsumerSaysHi_thenGotFailoverResponse() {
    ClassPathXmlApplicationContext localContext
      = new ClassPathXmlApplicationContext(
      "cluster/consumer-app-failtest.xml");
    localContext.start();
    GreetingsService greetingsService
      = (GreetingsService) localContext.getBean("greetingsService");
    String hiMessage = greetingsService.sayHi("baeldung");

    assertNotNull(hiMessage);
    assertEquals("hi, failover baeldung", hiMessage);
}
```

## 8。总结

在本教程中，我们咬了一小口 Dubbo。大多数用户被它的简单性和丰富强大的功能所吸引。

除了本文中介绍的内容之外，该框架还有许多特性有待开发，例如参数验证、通知和回调、通用实现和引用、远程结果分组和合并、服务升级和向后兼容性等等。

和往常一样，完整的实现可以在 Github 上找到[。](https://web.archive.org/web/20220628152926/https://github.com/eugenp/tutorials/tree/master/dubbo)