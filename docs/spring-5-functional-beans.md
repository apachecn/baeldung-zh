# Spring 5 功能 Bean 注册

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-5-functional-beans>

## 1。概述

Spring 5 支持应用程序上下文中的功能 bean 注册。

简单地说，**这可以通过在`GenericApplicationContext`类中定义的一个新的`registerBean()`方法**的重载版本来完成。

让我们来看看这种功能的几个实际例子。

## 2。`Maven`属地

设置`Spring 5`项目的最快方法是通过将`spring-boot-starter-parent`依赖项添加到`pom.xml:`来使用`Spring Boot`

```java
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.6.1</version>
</parent>
```

对于我们的例子，我们还需要 [`spring-boot-starter-web`](https://web.archive.org/web/20220628105250/https://search.maven.org/classic/#search%7Cga%7C1%7Ca%3A%22spring-boot-starter-web%22%20AND%20g%3A%22org.springframework.boot%22) 和`spring-boot-starter-test`，以便在`JUnit`测试中使用 web 应用程序上下文:

```java
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-test</artifactId>
    <scope>test</scope>
</dependency>
```

当然，为了使用新的函数方式注册 bean，`Spring Boot`不是必需的。我们也可以直接添加`spring-core`依赖项:

```java
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-core</artifactId>
    <version>5.3.3</version>
</dependency>
```

## 3。功能 Bean 注册

**`registerBean()`API 可以接收两种类型的函数接口作为参数**:

*   **一个`Supplier`自变量**用于创建对象
*   **一个`BeanDefinitionCustomizer vararg`** ，可以用来提供一个或多个 lambda 表达式来定制`BeanDefinition`；这个接口有一个单独的`customize()`方法

首先，让我们创建一个非常简单的类定义，我们将使用它来创建 beans:

```java
public class MyService {
    public int getRandomNumber() {
        return new Random().nextInt(10);
    }
}
```

让我们添加一个可以用来运行`JUnit`测试的`@SpringBootApplication`类:

```java
@SpringBootApplication
public class Spring5Application {
    public static void main(String[] args) {
        SpringApplication.run(Spring5Application.class, args);
    }
}
```

接下来，我们可以使用`@SpringBootTest`注释创建一个`GenericWebApplicationContext`实例来设置我们的测试类:

```java
@RunWith(SpringRunner.class)
@SpringBootTest(classes = Spring5Application.class)
public class BeanRegistrationIntegrationTest {
    @Autowired
    private GenericWebApplicationContext context;

    //...
}
```

我们在示例中使用的是`GenericWebApplicationContext`类型，但是任何类型的应用程序上下文都可以以相同的方式注册 bean。

让我们看看如何使用 lambda 表达式注册一个 bean 来创建实例:

```java
context.registerBean(MyService.class, () -> new MyService());
```

让我们验证我们现在可以检索 bean 并使用它:

```java
MyService myService = (MyService) context.getBean("com.baeldung.functional.MyService"); 

assertTrue(myService.getRandomNumber() < 10);
```

在这个例子中我们可以看到，如果 bean 名称没有被显式定义，它将由类的小写名称决定。上述相同的方法也可以用于显式 bean 名称:

```java
context.registerBean("mySecondService", MyService.class, () -> new MyService());
```

接下来，让我们看看如何通过添加一个 lambda 表达式来定制一个 bean:

```java
context.registerBean("myCallbackService", MyService.class, 
  () -> new MyService(), bd -> bd.setAutowireCandidate(false));
```

这个参数是一个回调函数，我们可以用它来设置 bean 属性，比如`autowire-candidate`标志或`primary`标志。

## 4。结论

在这个快速教程中，我们看到了如何使用函数方式注册 bean。

这个例子的源代码可以在 GitHub 上找到[。](https://web.archive.org/web/20220628105250/https://github.com/eugenp/tutorials/tree/master/spring-5)