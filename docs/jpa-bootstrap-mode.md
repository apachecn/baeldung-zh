# JPA 存储库的 BootstrapMode

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/jpa-bootstrap-mode>

 ![](img/182c6598ac805ad395733e56fea955b9.png)

JPA can behave very differently depending on the exact circumstances under which it is used. Code that works in our local environment or in staging performs very poorly (or even flat out fails) when thrown against real-scale databases in production environments.

Debugging these JPA issues in production is pretty difficult - existing APMs don’t provide enough granular insights at the code level, and tracking every single place someone queried entities one by one instead of in bulk can be a grueling, time-consuming task.

**Lightrun** is a new approach to debugging in production. Using Lightrun’s Logs and Snapshots, you can now get debugger-level granularity in production without opening inbound ports, redeploying, restarting, or even stropping the running application.

In addition, instrumenting Lightrun Metrics at runtime allows you to track down persistence issues securely and in real-time. Want to see it in action? **Check out our 2-minute tutorial** for debugging JPA performance issues in production using Lightrun:

[>> Debugging Spring Persistence and JPA Issues Using Lightrun](/web/20220524054113/https://www.baeldung.com/lightrun-n-jpa)

## 1.介绍

在这个简短的教程中，**我们将关注不同类型的 JPA 存储库的`BootstrapMode`，Spring 提供这些存储库来改变它们的实例化**的编排。

在启动时，Spring Data 扫描存储库，并将它们的 bean 定义注册为单例范围的 bean。在初始化期间，存储库立即获得一个`EntityManager`。具体来说，他们获取 JPA 元模型并验证声明的查询。

**JPA 默认同步引导。因此，存储库的实例化被阻塞，直到引导过程完成**。随着存储库数量的增长，应用程序可能需要很长时间才能开始接受请求。

## 2.引导存储库的不同选项

让我们从添加`spring-data-jpa`依赖项开始。正如我们使用 Spring Boot 一样，我们将使用相应的 [`spring-boot-starter-data-jpa`](https://web.archive.org/web/20220524054113/https://search.maven.org/search?q=g:org.springframework.boot%20a:spring-boot-starter-data-jpa) 依赖关系:

```
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-jpa</artifactId>
</dependency>
```

我们可以通过一个配置属性告诉 Spring 使用默认的存储库引导行为:

```
spring.data.jpa.repositories.bootstrap-mode=default
```

我们可以通过使用基于注释的配置来做到这一点:

```
@SpringBootApplication
@EnableJpaRepositories(bootstrapMode = BootstrapMode.DEFAULT)
public class Application {
    // ...
} 
```

第三种方法是使用`@DataJpaTest`注释，仅限于单个测试类:

```
@DataJpaTest(bootstrapMode = BootstrapMode.LAZY)
class BootstrapmodeLazyIntegrationTest {
    // ...
}
```

对于下面的例子，**我们将使用`@DataJpaTest`注释并探索不同的存储库引导选项**。

### 2.1.默认

引导模式的默认值将急切地实例化存储库。因此，像任何其他 Spring beans 一样，它们的初始化将在注入时发生。

让我们创建一个`Todo`实体:

```
@Entity
public class Todo {
    @Id
    private Long id;
    private String label;

    // standard setters and getters
}
```

接下来，我们需要它的关联存储库。让我们创建一个扩展`CrudRepository`的:

```
public interface TodoRepository extends CrudRepository<Todo, Long> {
}
```

最后，让我们添加一个使用我们的存储库的测试:

```
@DataJpaTest
class BootstrapmodeDefaultIntegrationTest {

    @Autowired
    private TodoRepository todoRepository;

    @Test
    void givenBootstrapmodeValueIsDefault_whenCreatingTodo_shouldSuccess() {
        Todo todo = new Todo("Something to be done");

        assertThat(todoRepository.save(todo)).hasNoNullFieldsOrProperties();
    }
}
```

在开始我们的测试之后，让我们检查日志，在那里我们将发现 Spring 是如何引导我们的`TodoRepository`:

```
[2022-03-22 14:46:47,597]-[main] INFO  org.springframework.data.repository.config.RepositoryConfigurationDelegate - Bootstrapping Spring Data JPA repositories in DEFAULT mode.
[2022-03-22 14:46:47,737]-[main] TRACE org.springframework.data.repository.config.RepositoryConfigurationDelegate - Spring Data JPA - Registering repository: todoRepository - Interface: com.baeldung.boot.bootstrapmode.repository.TodoRepository - Factory: org.springframework.data.jpa.repository.support.JpaRepositoryFactoryBean
[2022-03-22 14:46:49,718]-[main] DEBUG org.springframework.data.repository.core.support.RepositoryFactorySupport - Initializing repository instance for com.baeldung.boot.bootstrapmode.repository.TodoRepository…
[2022-03-22 14:46:49,792]-[main] DEBUG org.springframework.data.repository.core.support.RepositoryFactorySupport - Finished creation of repository instance for com.baeldung.boot.bootstrapmode.repository.TodoRepository.
[2022-03-22 14:46:49,858]-[main] INFO  com.baeldung.boot.bootstrapmode.BootstrapmodeDefaultIntegrationTest - Started BootstrapmodeDefaultIntegrationTest in 3.547 seconds (JVM running for 4.877) 
```

在我们的例子中，**我们很早就初始化了存储库，一旦应用程序启动了**，就可以使用它们。

### 2.2.懒惰的

通过对 JPA 存储库使用 lazy`BootstrapMode`, Spring 注册了我们的存储库的 bean 定义，但是没有立即实例化它。因此，使用懒惰选项，第一次使用触发其初始化。

让我们修改我们的测试，将惰性选项应用到`bootstrapMode`:

```
@DataJpaTest(bootstrapMode = BootstrapMode.LAZY)
```

然后，让我们用新的配置启动测试，并检查相应的日志:

```
[2022-03-22 15:09:01,360]-[main] INFO  org.springframework.data.repository.config.RepositoryConfigurationDelegate - Bootstrapping Spring Data JPA repositories in LAZY mode.
[2022-03-22 15:09:01,398]-[main] TRACE org.springframework.data.repository.config.RepositoryConfigurationDelegate - Spring Data JPA - Registering repository: todoRepository - Interface: com.baeldung.boot.bootstrapmode.repository.TodoRepository - Factory: org.springframework.data.jpa.repository.support.JpaRepositoryFactoryBean
[2022-03-22 15:09:01,971]-[main] INFO  com.baeldung.boot.bootstrapmode.BootstrapmodeLazyIntegrationTest - Started BootstrapmodeLazyIntegrationTest in 1.299 seconds (JVM running for 2.148)
[2022-03-22 15:09:01,976]-[main] DEBUG org.springframework.data.repository.config.RepositoryConfigurationDelegate$LazyRepositoryInjectionPointResolver - Creating lazy injection proxy for com.baeldung.boot.bootstrapmode.repository.TodoRepository…
[2022-03-22 15:09:02,588]-[main] DEBUG org.springframework.data.repository.core.support.RepositoryFactorySupport - Initializing repository instance for com.baeldung.boot.bootstrapmode.repository.TodoRepository… 
```

我们应该注意这里的几个缺点:

*   Spring 可能在没有初始化存储库的情况下就开始接受请求，从而在处理第一批请求时增加了延迟。
*   将`BootstrapMode`设置为全局懒惰容易出错。Spring 不会验证测试中没有包含的存储库中包含的查询和元数据。

我们应该只在开发期间使用惰性引导，以避免在生产环境中部署应用程序时出现潜在的初始化错误。为此，我们可以优雅地使用[弹簧型材](/web/20220524054113/https://www.baeldung.com/spring-profiles)。

### 2.3.延期的

当异步引导 JPA 时，Deferred 是正确的选择。因此，存储库不会等待`EntityManagerFactory`的初始化。

让我们通过使用`ThreadPoolTaskExecutor`——它的 Spring 实现之一——在配置类中声明一个`AsyncTaskExecutor`,并覆盖`submit`方法，该方法返回一个 [`Future`](/web/20220524054113/https://www.baeldung.com/java-future) :

```
@Bean
AsyncTaskExecutor delayedTaskExecutor() {
    return new ThreadPoolTaskExecutor() {
        @Override
        public <T> Future<T> submit(Callable<T> task) {
            return super.submit(() -> {
                Thread.sleep(5000);
                return task.call();
            });
        }
    };
}
```

接下来，让我们将一个`EntityManagerFactory` bean 添加到我们的配置中，如[我们的 Spring JPA 指南](/web/20220524054113/https://www.baeldung.com/the-persistence-layer-with-spring-and-jpa)所示，并指出我们想要使用我们的异步执行器进行后台引导:

```
@Bean
LocalContainerEntityManagerFactoryBean entityManagerFactory(DataSource dataSource, AsyncTaskExecutor delayedTaskExecutor) {
    LocalContainerEntityManagerFactoryBean factory = new LocalContainerEntityManagerFactoryBean();

    factory.setPackagesToScan("com.baeldung.boot.bootstrapmode");
    factory.setJpaVendorAdapter(new HibernateJpaVendorAdapter());
    factory.setDataSource(dataSource);
    factory.setBootstrapExecutor(delayedTaskExecutor);
    Map<String, Object> properties = new HashMap<>();
    properties.put("hibernate.hbm2ddl.auto", "create-drop");
    factory.setJpaPropertyMap(properties);
    return factory;
}
```

最后，让我们修改我们的测试来启用延迟引导模式:

```
@DataJpaTest(bootstrapMode = BootstrapMode.DEFERRED)
```

让我们再次启动测试并检查日志:

```
[2022-03-23 10:31:16,513]-[main] INFO  org.springframework.data.repository.config.RepositoryConfigurationDelegate - Bootstrapping Spring Data JPA repositories in DEFERRED mode.
[2022-03-23 10:31:16,543]-[main] TRACE org.springframework.data.repository.config.RepositoryConfigurationDelegate - Spring Data JPA - Registering repository: todoRepository - Interface: com.baeldung.boot.bootstrapmode.repository.TodoRepository - Factory: org.springframework.data.jpa.repository.support.JpaRepositoryFactoryBean
[2022-03-23 10:31:16,545]-[main] DEBUG org.springframework.data.repository.config.RepositoryConfigurationDelegate - Registering deferred repository initialization listener.
[2022-03-23 10:31:17,108]-[main] INFO  org.springframework.data.repository.config.DeferredRepositoryInitializationListener - Triggering deferred initialization of Spring Data repositories…
[2022-03-23 10:31:22,538]-[main] DEBUG org.springframework.data.repository.core.support.RepositoryFactorySupport - Initializing repository instance for com.baeldung.boot.bootstrapmode.repository.TodoRepository…
[2022-03-23 10:31:22,572]-[main] INFO  com.baeldung.boot.bootstrapmode.BootstrapmodeDeferredIntegrationTest - Started BootstrapmodeDeferredIntegrationTest in 6.769 seconds (JVM running for 7.519) 
```

在这个例子中，应用程序上下文引导完成触发了存储库的初始化。简而言之，Spring 将存储库标记为懒惰，并注册了一个`DeferredRepositoryInitializationListener` bean。当`ApplicationContext`触发一个`ContextRefreshedEvent`时，它初始化所有的库。

因此，在应用程序启动之前，Spring Data 初始化存储库并验证它们包含的查询和元数据。

## 3.结论

在本文中，**我们研究了初始化 JPA 存储库的各种方法，以及在什么情况下使用它们**。
和往常一样，本文中使用的所有代码示例都可以在 GitHub 上找到[。](https://web.archive.org/web/20220524054113/https://github.com/eugenp/tutorials/tree/master/spring-boot-modules/spring-boot-data-2)