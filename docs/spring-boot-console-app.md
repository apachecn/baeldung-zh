# Spring Boot 控制台应用程序

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-boot-console-app>

## 1.概观

在这个快速教程中，我们将探索如何使用 Spring Boot 创建一个简单的基于控制台的应用程序。

## 2.Maven 依赖性

我们的项目依赖于 spring-boot 父节点:

```java
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.4.0</version>
</parent> 
```

所需的初始依赖关系是:

```java
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter</artifactId>
</dependency> 
```

## 3.控制台应用程序

我们的控制台应用程序包含一个类:`SpringBootConsoleApplication.java `–这是我们 Spring Boot 控制台应用程序的主类。

我们在主类上使用了 **Spring 的`@SpringBootApplication`注释**来实现自动配置。

这个类也实现了 **Spring 的`CommandLineRunner`接口**。`CommandLineRunner`是一个简单的 Spring Boot 接口，带有一个`run`方法。加载应用程序上下文后，Spring Boot 将自动调用实现该接口的所有 bean 的`run`方法。

这是我们的控制台应用程序:

```java
@SpringBootApplication
public class SpringBootConsoleApplication 
  implements CommandLineRunner {

    private static Logger LOG = LoggerFactory
      .getLogger(SpringBootConsoleApplication.class);

    public static void main(String[] args) {
        LOG.info("STARTING THE APPLICATION");
        SpringApplication.run(SpringBootConsoleApplication.class, args);
        LOG.info("APPLICATION FINISHED");
    }

    @Override
    public void run(String... args) {
        LOG.info("EXECUTING : command line runner");

        for (int i = 0; i < args.length; ++i) {
            LOG.info("args[{}]: {}", i, args[i]);
        }
    }
}
```

我们还应该指定`spring.main.web-application-type=NONE` [弹簧属性](/web/20220525134236/https://www.baeldung.com/properties-with-spring)。该属性将明确通知 Spring 这不是一个 web 应用程序。

当我们执行`SpringBootConsoleApplication`时，我们可以看到以下记录:

```java
00:48:51.888 [main] INFO  c.b.s.SpringBootConsoleApplication - STARTING THE APPLICATION
00:48:52.752 [main] INFO  c.b.s.SpringBootConsoleApplication - No active profile set, falling back to default profiles: default
00:48:52.851 [main] INFO  o.s.c.a.AnnotationConfigApplicationContext 
  - Refreshing org.spring[[email protected]](/web/20220525134236/https://www.baeldung.com/cdn-cgi/l/email-protection)6497b078: startup date [Sat Jun 16 00:48:52 IST 2018]; root of context hierarchy
00:48:53.832 [main] INFO  o.s.j.e.a.AnnotationMBeanExporter - Registering beans for JMX exposure on startup
00:48:53.854 [main] INFO  c.b.s.SpringBootConsoleApplication - EXECUTING : command line runner
00:48:53.854 [main] INFO  c.b.s.SpringBootConsoleApplication - args[0]: Hello World!
00:48:53.860 [main] INFO  c.b.s.SpringBootConsoleApplication - Started SpringBootConsoleApplication in 1.633 seconds (JVM running for 2.373)
00:48:53.860 [main] INFO  c.b.s.SpringBootConsoleApplication - APPLICATION FINISHED
00:48:53.868 [Thread-2] INFO  o.s.c.a.AnnotationConfigApplicationContext 
  - Closing org.spring[[email protected]](/web/20220525134236/https://www.baeldung.com/cdn-cgi/l/email-protection)6497b078: startup date [Sat Jun 16 00:48:52 IST 2018]; root of context hierarchy
00:48:53.870 [Thread-2] INFO  o.s.j.e.a.AnnotationMBeanExporter - Unregistering JMX-exposed beans on shutdown
```

请注意，`run`方法是在应用程序上下文加载之后、`main`方法执行完成之前调用的。

大多数控制台应用程序只有一个实现`CommandLineRunner`的类。如果你的应用程序有多个实现了`CommandLineRunner`的类，执行的顺序可以使用 [Spring 的`@Order`注释](/web/20220525134236/https://www.baeldung.com/spring-order)来指定。

## 4.结论

在本文中，我们总结了如何使用 Spring Boot 创建一个简单的基于控制台的应用程序。

我们这里例子的完整源代码一如既往地在 GitHub 的[上。](https://web.archive.org/web/20220525134236/https://github.com/eugenp/tutorials/tree/master/spring-boot-modules/spring-boot-deployment)