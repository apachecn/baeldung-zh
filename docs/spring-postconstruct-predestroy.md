# Spring 后期构造和前期设计注释

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-postconstruct-predestroy>

## 1.介绍

Spring 允许我们为 [bean 的创建和销毁](/web/20220626110206/https://www.baeldung.com/running-setup-logic-on-startup-in-spring)附加自定义动作。例如，我们可以通过实现`InitializingBean`和`DisposableBean`接口来实现。

在这个快速教程中，我们将看看第二种可能性，`@PostConstruct`和`@PreDestroy`注释。

## 2.`@PostConstruct`

**Spring 只调用一次用`@PostConstruct`标注的方法，就在 bean 属性**初始化之后。请记住，即使没有任何东西需要初始化，这些方法也会运行。

用`@PostConstruct`标注的方法可以有任何访问级别，但不能是静态的。

`@PostConstruct`的一个可能用途是填充数据库。例如，在开发期间，我们可能想要创建一些默认用户:

```java
@Component
public class DbInit {

    @Autowired
    private UserRepository userRepository;

    @PostConstruct
    private void postConstruct() {
        User admin = new User("admin", "admin password");
        User normalUser = new User("user", "user password");
        userRepository.save(admin, normalUser);
    }
}
```

上面的例子将首先初始化`UserRepository`，然后运行`@PostConstruct`方法。

## 3.`@PreDestroy`

用`@PreDestroy`标注的方法只运行一次，就在 Spring 从应用程序上下文中移除 bean 之前。

与`@PostConstruct`相同，用`@PreDestroy`标注的方法可以有任何访问级别，但不能是静态的。

```java
@Component
public class UserRepository {

    private DbConnection dbConnection;
    @PreDestroy
    public void preDestroy() {
        dbConnection.close();
    }
}
```

这个方法的目的应该是在 bean 被破坏之前释放资源或执行其他清理任务，比如关闭数据库连接。

## 4.Java 9+版本

注意，`@PostConstruct`和`@PreDestroy`注释都是 Java EE 的一部分。由于 [Java EE 在 Java 9](/web/20220626110206/https://www.baeldung.com/java-enterprise-evolution) 中被弃用，并且在 Java 11 中被删除，我们必须添加一个额外的依赖项来使用这些注释:

```java
<dependency>
    <groupId>javax.annotation</groupId>
    <artifactId>javax.annotation-api</artifactId>
    <version>1.3.2</version>
</dependency>
```

## 5.结论

在这篇简短的文章中，我们学习了如何使用`@PostConstruct`和`@PreDestroy`注释。

和往常一样，所有的源代码都可以在 [GitHub](https://web.archive.org/web/20220626110206/https://github.com/eugenp/tutorials/tree/master/spring-core) 上获得。