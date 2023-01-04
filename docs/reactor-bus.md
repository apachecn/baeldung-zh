# 项目反应堆总线介绍

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/reactor-bus>

## 1。概述

在这篇简短的文章中，我们将通过为一个反应式的、事件驱动的应用程序设置一个真实的场景来介绍[反应器总线](https://web.archive.org/web/20220813070710/https://projectreactor.io/docs/core/snapshot/reference/)。

**注:电抗器母线项目在电抗器 3.x: [归档电抗器母线库](https://web.archive.org/web/20220813070710/https://github.com/reactor-attic/reactor-bus#reactor-bus)** 中已删除

## 2。项目反应堆的基础知识

### 2.1。为什么是反应堆？

现代应用程序需要处理大量并发请求和大量数据。标准的阻塞代码不再足以满足这些需求。

反应式设计[模式](https://web.archive.org/web/20220813070710/https://en.wikipedia.org/wiki/Reactor_pattern)是一种基于**事件的架构方法，用于异步处理来自单个或多个服务处理程序的大量并发服务请求**。

Project Reactor 基于这种模式，有一个清晰而雄心勃勃的目标:在 JVM 上构建非阻塞的、反应式的应用程序

### 2.2。示例场景

在我们开始之前，这里有一些有趣的场景，在这些场景中利用反应式架构风格是有意义的，只是为了了解我们可能在哪里应用它:

*   像亚马逊这样的大型在线购物平台的通知服务
*   面向银行业的大量交易处理服务
*   股票价格同时变化的股票交易业务

## 3。Maven 依赖关系

让我们开始使用 Project Reactor Bus，将下面的依赖项添加到我们的`pom.xml:`中

```java
<dependency>
    <groupId>io.projectreactor</groupId>
    <artifactId>reactor-bus</artifactId>
    <version>2.0.8.RELEASE</version>
</dependency>
```

我们可以在 [Maven Central](https://web.archive.org/web/20220813070710/https://search.maven.org/classic/#search%7Cga%7C1%7Cg%3A%22io.projectreactor%22) 查看`reactor-bus`的最新版本。

## 4。构建演示应用程序

为了更好地理解基于反应器的方法的好处，让我们看一个实际的例子。

我们将构建一个简单的应用程序，负责向在线购物平台的用户发送通知。例如，如果用户下了新订单，应用程序会通过电子邮件或短信发送订单确认。

典型的同步实现自然会受到电子邮件或 SMS 服务吞吐量的限制。因此，流量高峰(如节假日)通常会成为问题。

使用反应式方法，我们可以将系统设计得更加灵活，更好地适应外部系统(如网关服务器)中可能出现的故障或超时。

让我们看一下应用程序——从更传统的方面开始，然后转向更具反应性的结构。

### 4.1。简单 POJO

首先，让我们创建一个 POJO 类来表示通知数据:

```java
public class NotificationData {

    private long id;
    private String name;
    private String email;
    private String mobile;

    // getter and setter methods
}
```

### 4.2。服务层

现在让我们定义一个简单的服务层:

```java
public interface NotificationService {

    void initiateNotification(NotificationData notificationData) 
      throws InterruptedException;

}
```

以及模拟长期运行操作的实现:

```java
@Service
public class NotificationServiceimpl implements NotificationService {

    @Override
    public void initiateNotification(NotificationData notificationData) 
      throws InterruptedException {

      System.out.println("Notification service started for "
        + "Notification ID: " + notificationData.getId());

      Thread.sleep(5000);

      System.out.println("Notification service ended for "
        + "Notification ID: " + notificationData.getId());
    }
}
```

请注意，为了说明通过 SMS 或电子邮件网关发送消息的真实场景，我们有意在使用`Thread.sleep(5000).` 的`initiateNotification`方法中引入了 5 秒钟的延迟

因此，当一个线程命中服务时，它将被阻塞五秒钟。

### 4.3。消费者

现在让我们进入应用程序的更具反应性的方面，实现一个消费者——然后我们将它映射到反应器事件总线:

```java
@Service
public class NotificationConsumer implements 
  Consumer<Event<NotificationData>> {

    @Autowired
    private NotificationService notificationService;

    @Override
    public void accept(Event<NotificationData> notificationDataEvent) {
        NotificationData notificationData = notificationDataEvent.getData();

        try {
            notificationService.initiateNotification(notificationData);
        } catch (InterruptedException e) {
            // ignore        
        }	
    }
}
```

正如我们所看到的，我们创建的消费者实现了`Consumer<T>` 接口。主要的逻辑存在于`accept`方法中。

这是我们在典型的 [Spring 监听器](/web/20220813070710/https://www.baeldung.com/spring-events#listener)实现中可以遇到的类似方法。

### 4.4。控制器

最后，既然我们能够消费事件，让我们也生成它们。

我们将在一个简单的控制器中实现这一点:

```java
@Controller
public class NotificationController {

    @Autowired
    private EventBus eventBus;

    @GetMapping("/startNotification/{param}")
    public void startNotification(@PathVariable Integer param) {
        for (int i = 0; i < param; i++) {
            NotificationData data = new NotificationData();
            data.setId(i);

            eventBus.notify("notificationConsumer", Event.wrap(data));

            System.out.println(
              "Notification " + i + ": notification task submitted successfully");
        }
    }
}
```

这是不言自明的——我们在这里通过`EventBus`发出事件。

例如，如果客户端点击参数值为 10 的 URL，那么将通过事件总线发送 10 个事件。

### 4.5。Java 配置

现在让我们把所有东西放在一起，创建一个简单的 [Spring Boot](/web/20220813070710/https://www.baeldung.com/spring-boot) 应用程序。

首先，我们需要配置`EventBus`和`Environment`bean:

```java
@Configuration
public class Config {

    @Bean
    public Environment env() {
        return Environment.initializeIfEmpty().assignErrorJournal();
    }

    @Bean
    public EventBus createEventBus(Environment env) {
        return EventBus.create(env, Environment.THREAD_POOL);
    }
}
```

在我们的例子中，**我们用环境**中可用的默认线程池实例化了`EventBus`。

或者，我们可以使用定制的`Dispatcher` 实例:

```java
EventBus evBus = EventBus.create(
  env, 
  Environment.newDispatcher(
    REACTOR_CAPACITY,REACTOR_CONSUMERS_COUNT,   
    DispatcherType.THREAD_POOL_EXECUTOR));
```

现在，我们准备创建一个主应用程序代码:

```java
import static reactor.bus.selector.Selectors.$;

@SpringBootApplication
public class NotificationApplication implements CommandLineRunner {

    @Autowired
    private EventBus eventBus;

    @Autowired
    private NotificationConsumer notificationConsumer;

    @Override
    public void run(String... args) throws Exception {
        eventBus.on($("notificationConsumer"), notificationConsumer);
    }

    public static void main(String[] args) {
        SpringApplication.run(NotificationApplication.class, args);
    }
}
```

在我们的`run`方法**中，我们注册了当通知匹配给定的选择器**时被触发的`notificationConsumer` 。

注意我们如何使用静态导入的`$`属性来创建一个`Selector`对象。

## 5。测试应用程序

现在让我们创建一个测试来看看我们的`NotificationApplication` 在运行:

```java
@RunWith(SpringRunner.class)
@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
public class NotificationApplicationIntegrationTest {

    @LocalServerPort
    private int port;

    @Test
    public void givenAppStarted_whenNotificationTasksSubmitted_thenProcessed() {
        RestTemplate restTemplate = new RestTemplate();
        restTemplate.getForObject("http://localhost:" + port + "/startNotification/10", String.class);
    }
}
```

正如我们所看到的，一旦请求被执行，所有十个**任务立即被提交，而不会产生任何阻塞**。一旦提交，通知事件将被并行处理。

```java
Notification 0: notification task submitted successfully
Notification 1: notification task submitted successfully
Notification 2: notification task submitted successfully
Notification 3: notification task submitted successfully
Notification 4: notification task submitted successfully
Notification 5: notification task submitted successfully
Notification 6: notification task submitted successfully
Notification 7: notification task submitted successfully
Notification 8: notification task submitted successfully
Notification 9: notification task submitted successfully
Notification service started for Notification ID: 1
Notification service started for Notification ID: 2
Notification service started for Notification ID: 3
Notification service started for Notification ID: 0
Notification service ended for Notification ID: 1
Notification service ended for Notification ID: 0
Notification service started for Notification ID: 4
Notification service ended for Notification ID: 3
Notification service ended for Notification ID: 2
Notification service started for Notification ID: 6
Notification service started for Notification ID: 5
Notification service started for Notification ID: 7
Notification service ended for Notification ID: 4
Notification service started for Notification ID: 8
Notification service ended for Notification ID: 6
Notification service ended for Notification ID: 5
Notification service started for Notification ID: 9
Notification service ended for Notification ID: 7
Notification service ended for Notification ID: 8
Notification service ended for Notification ID: 9
```

重要的是要记住，在我们的场景中，不需要以任何特定的顺序处理这些事件。

## 6。结论

在这个快速教程中，**我们已经创建了一个简单的事件驱动应用程序**。我们还看到了如何开始编写更具反应性和非阻塞的代码。

然而，**这个场景只是触及了主题的表面，并且代表了开始实验反应范式**的良好基础。

和往常一样，源代码可以在 GitHub 上的[处获得。](https://web.archive.org/web/20220813070710/https://github.com/eugenp/tutorials/tree/master/spring-reactor)