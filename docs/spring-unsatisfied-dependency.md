# 春季未满足的依赖性

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-unsatisfied-dependency>

## 1.概观

在这个快速教程中，我们将解释 Spring 的`UnsatisfiedDependencyException`，它的成因以及如何避免。

## 2.`UnsatisfiedDependencyException`的起因

**`UnsatisfiedDependencyException`顾名思义，当某个 bean 或属性依赖不满足时抛出。**

当一个 Spring 应用程序试图连接一个 bean 并且不能解析一个强制依赖时，这可能会发生。

## 3.示例应用程序

假设我们有一个服务类`PurchaseDeptService`，它依赖于`InventoryRepository`:

```java
@Service
public class PurchaseDeptService {
    public PurchaseDeptService(InventoryRepository repository) {
        this.repository = repository;
    }
}
```

```java
public interface InventoryRepository {
} 
```

```java
@Repository
public class ShoeRepository implements InventoryRepository {
}
```

```java
@SpringBootApplication
public class SpringDependenciesExampleApplication {

    public static void main(String[] args) {
        SpringApplication.run(SpringDependenciesExampleApplication.class, args);
    }
} 
```

现在，我们假设所有这些类都位于同一个名为`com.baeldung.dependency.exception.app`的包中。

当我们运行这个 Spring Boot 应用程序时，一切正常。

让我们看看如果我们跳过一个配置步骤，会遇到什么样的问题。

## 4.缺少组件注释

现在让我们从我们的`ShoeRepository`类中移除`@Repository `注释:

```java
public class ShoeRepository implements InventoryRepository {
}
```

当我们再次启动应用程序时，我们会看到下面的错误消息:`UnsatisfiedDependencyException: Error creating bean with name ‘purchaseDeptService': Unsatisfied dependency expressed through constructor parameter 0`

Spring 没有被指示连接一个`ShoeRepository` bean 并将其添加到应用程序上下文中，所以它不能注入它并抛出异常。

将`@Repository`注释添加回`ShoeRepository`解决了这个问题。

## 5.包裹未扫描

现在让我们把我们的`ShoeRepository`(连同`InventoryRepository`)放到一个名为`com.baeldung.dependency.exception.repository`的单独的包中。

同样，当我们运行我们的应用程序时，它抛出了`UnsatisfiedDependencyException`。

为了解决这个问题，我们可以在父包上配置包扫描，并确保包含所有相关的类:

```java
@SpringBootApplication
@ComponentScan(basePackages = {"com.baeldung.dependency.exception"})
public class SpringDependenciesExampleApplication {
    public static void main(String[] args) {
        SpringApplication.run(SpringDependenciesExampleApplication.class, args);
    }
} 
```

## 6.非唯一依赖项解析

假设我们添加了另一个`InventoryRepository`实现— `DressRepository`:

```java
@Repository
public class DressRepository implements InventoryRepository {
} 
```

现在当我们运行我们的应用程序时，它会再次抛出`UnsatisfiedDependencyException`。

然而，这一次的情况有所不同。碰巧的是，当有多个 bean 满足依赖关系时,**依赖关系不能被解析。**

为了解决这个问题，我们可能想要添加`@Qualifier`来区分存储库:

```java
@Qualifier("dresses")
@Repository
public class DressRepository implements InventoryRepository {
} 
```

```java
@Qualifier("shoes")
@Repository
public class ShoeRepository implements InventoryRepository {
}
```

此外，我们必须给`PurchaseDeptService`构造函数依赖关系添加一个限定符:

```java
public PurchaseDeptService(@Qualifier("dresses") InventoryRepository repository) {
    this.repository = repository;
}
```

这将使`DressRepository`成为唯一可行的选项，Spring 将把它注入`PurchaseDeptService`。

## 7.结论

在这篇文章中，我们看到了几种最常见的遇到`UnsatisfiedDependencyException`的情况，然后我们学习了如何解决这些问题。

我们还有一个关于[Spring bean creation exception](/web/20221208143856/https://www.baeldung.com/spring-beancreationexception)的更一般的教程。

这里展示的代码可以在 GitHub 上找到[。](https://web.archive.org/web/20221208143856/https://github.com/eugenp/tutorials/tree/master/spring-di)