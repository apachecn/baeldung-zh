# Akka 介绍春天

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/akka-with-spring>

## 1。简介

在本文中，我们将关注 Akka 与 Spring 框架的集成——允许将基于 Spring 的服务注入到 Akka actors 中。

在阅读本文之前，建议先了解 Akka 的基础知识。

## 延伸阅读:

## [Java 中的 Akka Actors 简介](/web/20220813062338/https://www.baeldung.com/akka-actors-java)

Learn how to build concurrent and distributed applications using Akka Actors in Java.[Read more](/web/20220813062338/https://www.baeldung.com/akka-actors-java) →

## [阿卡流指南](/web/20220813062338/https://www.baeldung.com/akka-streams)

A quick and practical guide to data stream transformations in Java using the Akka Streams library.[Read more](/web/20220813062338/https://www.baeldung.com/akka-streams) →

## 2。Akka 中的依赖注入

Akka 是一个基于 Actor 并发模型的强大的应用框架。这个框架是用 Scala 编写的，这当然也使得它在基于 Java 的应用程序中完全可用。因此**我们经常想要将 Akka 与现有的基于 Spring 的应用程序集成**或者简单地使用 Spring 将 beans 连接到 actors。

Spring/Akka 集成的问题在于 Spring 中 bean 的管理和 Akka 中 actors 的管理之间的差异: **actors 有一个特定的生命周期，它不同于典型的 Spring bean 生命周期**。

此外，actor 被分为 actor 本身(这是一个内部实现细节，不能由 Spring 管理)和 actor 引用，actor 引用可由客户端代码访问，并且可在不同的 Akka 运行时之间序列化和移植。

幸运的是，Akka 提供了一种机制，即 [Akka 扩展](https://web.archive.org/web/20220813062338/http://doc.akka.io/docs/akka/current/java/extending-akka.html)，这使得使用外部依赖注入框架成为一项相当容易的任务。

## 3。Maven 依赖关系

为了在我们的 Spring 项目中演示 Akka 的用法，我们需要一个最简单的 Spring 依赖——`spring-context`库和`akka-actor`库。可以将库版本提取到`pom`的`<properties>`部分:

```java
<properties>
    <spring.version>4.3.1.RELEASE</spring.version>
    <akka.version>2.4.8</akka.version>
</properties>

<dependencies>
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-context</artifactId>
        <version>${spring.version}</version>
    </dependency>

    <dependency>
        <groupId>com.typesafe.akka</groupId>
        <artifactId>akka-actor_2.11</artifactId>
        <version>${akka.version}</version>
    </dependency>

</dependencies>
```

请务必检查 Maven Central 以获得最新版本的 [`spring-context`](https://web.archive.org/web/20220813062338/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22org.springframework%22%20AND%20a%3A%22spring-context%22) 和 [`akka-actor`](https://web.archive.org/web/20220813062338/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22com.typesafe.akka%22%20AND%20a%3A%22akka-actor_2.11%22) 依赖项。

请注意，`akka-actor`依赖项的名称中有一个`_2.11`后缀，这表明这个版本的 Akka 框架是针对 Scala 版构建的。Scala 库的相应版本将被过渡性地包含在您的构建中。

## 4。给阿卡演员注射春豆

让我们创建一个简单的 Spring/Akka 应用程序，它由一个演员组成，可以通过向一个人发出问候来回答这个人的名字。问候的逻辑将被提取到一个单独的服务中。我们希望将该服务自动连接到一个 actor 实例。Spring integration 将帮助我们完成这项任务。

### 4.1。定义参与者和服务

为了演示如何将服务注入到 actor 中，我们将创建一个简单的类`GreetingActor`,定义为非类型化 actor(扩展 Akka 的`UntypedActor`基类)。每个 Akka actor 的主要方法是`onReceive`方法，它接收消息并根据一些特定的逻辑处理它。

在我们的例子中，`GreetingActor`实现检查消息是否是预定义的类型`Greet`，然后从`Greet`实例中获取这个人的名字，然后使用`GreetingService`接收这个人的问候，并用接收到的问候字符串回答发送者。如果消息属于其他未知类型，它将被传递给参与者预定义的`unhandled`方法。

让我们来看看:

```java
@Component
@Scope(ConfigurableBeanFactory.SCOPE_PROTOTYPE)
public class GreetingActor extends UntypedActor {

    private GreetingService greetingService;

    // constructor

    @Override
    public void onReceive(Object message) throws Throwable {
        if (message instanceof Greet) {
            String name = ((Greet) message).getName();
            getSender().tell(greetingService.greet(name), getSelf());
        } else {
            unhandled(message);
        }
    }

    public static class Greet {

        private String name;

        // standard constructors/getters

    }
}
```

注意,`Greet`消息类型被定义为这个 actor 内部的一个静态内部类，这被认为是一个好的实践。被接受的消息类型应该被定义为尽可能接近某个参与者，以避免混淆该参与者可以处理哪些消息类型。

**还要注意 Spring 注释`@Component`和`@Scope`**——它们将该类定义为一个具有`prototype`范围的 Spring 管理的 bean。

范围非常重要，因为每个 bean 检索请求都应该产生一个新创建的实例，因为这种行为与 Akka 的 actor 生命周期相匹配。如果您用其他作用域实现这个 bean，那么在 Akka 中重启 actors 的典型情况很可能无法正常工作。

最后，请注意，我们不必显式地`@Autowire`该`GreetingService`实例——这是可能的，因为 Spring 4.3 的新特性叫做`Implicit Constructor Injection`。

`GreeterService`的实现非常简单，注意我们通过添加`@Component`注释(默认为`singleton`范围)将其定义为 Spring 管理的 bean:

```java
@Component
public class GreetingService {

    public String greet(String name) {
        return "Hello, " + name;
    }
}
```

### 4.2。通过 Akka 延伸件添加弹簧支架

集成 Spring 和 Akka 最简单的方法是通过 Akka 扩展。

**扩展是为每个 actor 系统创建的单例实例。**它由实现标记接口`Extension`的扩展类本身和通常继承`AbstractExtensionId`的扩展 id 类组成。

因为这两个类是紧密耦合的，所以实现嵌套在`ExtensionId`类中的`Extension`类是有意义的:

```java
public class SpringExtension 
  extends AbstractExtensionId<SpringExtension.SpringExt> {

    public static final SpringExtension SPRING_EXTENSION_PROVIDER 
      = new SpringExtension();

    @Override
    public SpringExt createExtension(ExtendedActorSystem system) {
        return new SpringExt();
    }

    public static class SpringExt implements Extension {
        private volatile ApplicationContext applicationContext;

        public void initialize(ApplicationContext applicationContext) {
            this.applicationContext = applicationContext;
        }

        public Props props(String actorBeanName) {
            return Props.create(
              SpringActorProducer.class, applicationContext, actorBeanName);
        }
    }
}
```

**第一个**–`SpringExtension`实现了来自`AbstractExtensionId`类的单个`createExtension`方法——这说明了扩展实例的创建，即`SpringExt`对象。

`SpringExtension`类还有一个静态字段`SPRING_EXTENSION_PROVIDER`,它保存了对其唯一实例的引用。添加一个私有构造函数来显式声明`SpringExtention`应该是一个单例类通常是有意义的，但是为了清楚起见，我们将省略它。

**其次**，静态内部类`SpringExt`就是扩展本身。因为`Extension`仅仅是一个标记接口，我们可以定义我们认为合适的这个类的内容。

在我们的例子中，我们需要用`initialize`方法来保存一个 Spring `ApplicationContext`实例——这个方法在每次扩展初始化时只被调用一次。

我们还需要用`props`方法来创建一个`Props`对象。`Props`实例是一个 actor 的蓝图，在我们的例子中，`Props.create`方法接收一个`SpringActorProducer`类和这个类的构造函数参数。这些是调用这个类的构造函数的参数。

每当我们需要 Spring 管理的 actor 引用时，就会执行`props`方法。

**第三个**也是最后一块拼图是`SpringActorProducer`类。它实现了 Akka 的`IndirectActorProducer`接口，该接口允许通过实现`produce`和`actorClass`方法来覆盖参与者的实例化过程。

您可能已经猜到，**不是直接实例化，而是总是从 Spring 的`ApplicationContext`** 中检索一个 actor 实例。因为我们已经将 actor 设为了一个`prototype`作用域的 bean，所以每次调用`produce`方法都将返回 actor 的一个新实例:

```java
public class SpringActorProducer implements IndirectActorProducer {

    private ApplicationContext applicationContext;

    private String beanActorName;

    public SpringActorProducer(ApplicationContext applicationContext, 
      String beanActorName) {
        this.applicationContext = applicationContext;
        this.beanActorName = beanActorName;
    }

    @Override
    public Actor produce() {
        return (Actor) applicationContext.getBean(beanActorName);
    }

    @Override
    public Class<? extends Actor> actorClass() {
        return (Class<? extends Actor>) applicationContext
          .getType(beanActorName);
    }
}
```

### 4.3。将所有这些放在一起

剩下唯一要做的事情是创建一个 Spring 配置类(用`@Configuration`注释标记),它将告诉 Spring 扫描当前包以及所有嵌套的包(这由`@ComponentScan`注释确保)并创建一个 Spring 容器。

我们只需要添加一个额外的 bean——`ActorSystem`实例——并在这个`ActorSystem`上初始化 Spring 扩展:

```java
@Configuration
@ComponentScan
public class AppConfiguration {

    @Autowired
    private ApplicationContext applicationContext;

    @Bean
    public ActorSystem actorSystem() {
        ActorSystem system = ActorSystem.create("akka-spring-demo");
        SPRING_EXTENSION_PROVIDER.get(system)
          .initialize(applicationContext);
        return system;
    }
}
```

### 4.4。检索弹簧连接的演员

为了测试一切是否正常，我们可以将`ActorSystem`实例注入到我们的代码中(或者是一些 Spring 管理的应用程序代码，或者是基于 Spring 的测试)，使用我们的扩展为一个 actor 创建一个`Props`对象，通过`Props`对象检索一个 actor 的引用，并尝试问候某人:

```java
ActorRef greeter = system.actorOf(SPRING_EXTENSION_PROVIDER.get(system)
  .props("greetingActor"), "greeter");

FiniteDuration duration = FiniteDuration.create(1, TimeUnit.SECONDS);
Timeout timeout = Timeout.durationToTimeout(duration);

Future<Object> result = ask(greeter, new Greet("John"), timeout);

Assert.assertEquals("Hello, John", Await.result(result, duration));
```

这里我们使用典型的`akka.pattern.Patterns.ask`模式，返回 Scala 的`Future`实例。一旦计算完成，`Future`就用我们在`GreetingActor.onMessasge`方法中返回的值来解析。

我们可以通过将 Scala 的`Await.result`方法应用到`Future`来等待结果，或者，更好的是，用异步模式构建整个应用程序。

## 5。结论

在本文中，我们展示了如何将 Spring Framework 与 Akka 和 autowire beans 集成到 actors 中。

这篇文章的源代码可以在 GitHub 的[上找到。](https://web.archive.org/web/20220813062338/https://github.com/eugenp/tutorials/tree/master/akka-modules/spring-akka)