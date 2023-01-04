# Spring Boot 2 中的惰性初始化

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-boot-lazy-initialization>

## 1.概观

在本教程中，我们将从 [Spring Boot 2.2 开始，看看如何在应用程序级别配置惰性初始化。](/web/20221129021422/https://www.baeldung.com/new-spring-boot-2)

## 2.惰性初始化

默认情况下，在 Spring 中，所有已定义的 beans 及其依赖项都是在创建应用程序上下文时创建的。

**相反，当我们用惰性初始化配置 bean 时，****bean 只会在需要的时候被创建，它的依赖项被注入。**

## 3.Maven 依赖性

为了在我们的应用程序中包含 Spring Boot，我们需要将它包含在我们的类路径中。

有了 Maven，我们可以只添加[和`spring-boot-starter` 的依赖关系](https://web.archive.org/web/20221129021422/https://mvnrepository.com/artifact/org.springframework.boot/spring-boot-starter):

```
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter</artifactId>
        <version>2.4.3</version>
    </dependency>
</dependencies> 
```

## 4.启用惰性初始化

Spring Boot 2 引入了`spring.main.lazy-initialization`属性，使得在整个应用程序中配置惰性初始化变得更加容易。

**将属性值设置为`true`意味着应用程序中的所有 beans 都将使用延迟初始化。**

让我们在`application.yml`配置文件中配置该属性:

```
spring:
  main:
    lazy-initialization: true
```

或者，如果是这样，在我们的`application.properties`文件中:

```
spring.main.lazy-initialization=true
```

该配置影响上下文中的所有 beans。因此，如果我们想为特定的 bean 配置惰性初始化，我们可以通过 [`@Lazy`方法](/web/20221129021422/https://www.baeldung.com/spring-lazy-annotation)来完成。

甚至，我们可以使用新的属性，结合`@Lazy`注释，设置为`false`。

或者换句话说，**除了那些我们用`@Lazy(false)`** `.`显式配置的，所有定义的 beans 都将使用惰性初始化

### 4.1.使用`SpringApplicationBuilder`

配置惰性初始化的另一种方法是使用`SpringApplicationBuilder`方法:

```
SpringApplicationBuilder(Application.class)
  .lazyInitialization(true)
  .build(args)
  .run();
```

在上面的例子中，我们使用 [`lazyInitialization`](https://web.archive.org/web/20221129021422/https://docs.spring.io/spring-boot/docs/current/api/org/springframework/boot/builder/SpringApplicationBuilder.html#lazyInitialization-boolean-) 方法来控制应用程序是否应该被惰性初始化。

### 4.2.使用`SpringApplication`

或者，我们也可以使用`SpringApplication`类:

```
SpringApplication app = new SpringApplication(Application.class);
app.setLazyInitialization(true);
app.run(args);
```

在这里，我们使用 [`setLazyInitialization`](https://web.archive.org/web/20221129021422/https://docs.spring.io/spring-boot/docs/current/api/org/springframework/boot/SpringApplication.html#setLazyInitialization-boolean-) 的方法来配置我们的应用程序进行惰性初始化。

需要记住的重要一点是，应用程序属性文件中定义的**属性优先于使用`SpringApplication`或`SpringApplicationBuilder`** 设置的标志。

## 5.奔跑

让我们创建一个简单的服务来测试我们刚刚描述的内容。

通过向构造函数添加消息，我们将确切地知道 bean 是何时创建的。

```
public class Writer {

    private final String writerId;

    public Writer(String writerId) {
        this.writerId = writerId;
        System.out.println(writerId + " initialized!!!");
    }

    public void write(String message) {
        System.out.println(writerId + ": " + message);
    }

}
```

同样，让我们创建`SpringApplication`并注入我们之前创建的服务。

```
@SpringBootApplication
public class Application {

    @Bean("writer1")
    public Writer getWriter1() {
        return new Writer("Writer 1");
    }

    @Bean("writer2")
    public Writer getWriter2() {
        return new Writer("Writer 2");
    }

    public static void main(String[] args) {
        ApplicationContext ctx = SpringApplication.run(Application.class, args);
        System.out.println("Application context initialized!!!");

        Writer writer1 = ctx.getBean("writer1", Writer.class);
        writer1.write("First message");

        Writer writer2 = ctx.getBean("writer2", Writer.class);
        writer2.write("Second message");
    }
}
```

让我们将`spring.main.lazy-initialization`属性值设置为`false`，并运行我们的应用程序。

```
Writer 1 initialized!!!
Writer 2 initialized!!!
Application context initialized!!!
Writer 1: First message
Writer 2: Second message
```

正如我们所看到的，beans 是在应用程序上下文启动时创建的。

现在让我们将`spring.main.lazy-initialization`的值改为`true`，并再次运行我们的应用程序。

```
Application context initialized!!!
Writer 1 initialized!!!
Writer 1: First message
Writer 2 initialized!!!
Writer 2: Second message
```

因此，应用程序不会在启动时创建 beans，而是在需要时才创建。

## 6.惰性初始化的影响

在整个应用程序中启用惰性初始化会产生积极和消极的影响。

让我们来谈谈其中的一些，因为在新功能的官方发布中[对它们进行了描述:](https://web.archive.org/web/20221129021422/https://spring.io/blog/2019/03/14/lazy-initialization-in-spring-boot-2-2)

1.  惰性初始化可能会减少应用程序启动时创建的 beans 数量——因此，**我们可以改善应用程序的启动时间**
2.  由于直到需要时才会创建 beans，**我们可以屏蔽问题，在运行时而不是启动时得到它们**
3.  这些问题可能包括内存不足错误、配置错误或类定义发现错误
4.  此外，当我们处于 web 环境中时，**按需触发 bean 创建会增加 HTTP 请求的延迟**——bean 创建只会影响第一个请求，但是**这可能会对负载平衡和自动伸缩产生负面影响**。

## 7.结论

在本教程中，我们用 Spring Boot 2.2 中引入的新属性`spring.main.lazy-initialization,`配置了惰性初始化。

和往常一样，本教程的源代码可以在 GitHub 上的[处获得。](https://web.archive.org/web/20221129021422/https://github.com/eugenp/tutorials/tree/master/spring-boot-modules/spring-boot-performance)