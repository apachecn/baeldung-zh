# 如何获取所有 Spring 管理的 Beans？

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-show-all-beans>

## 1。概述

在本文中，我们将探索在容器中显示所有 Spring 管理的 beans 的不同技术。

## 2。国际奥委会容器

bean 是 Spring 管理的应用程序的基础；所有 beans 都驻留在 IOC 容器中，IOC 容器负责管理它们的生命周期。

我们可以通过两种方式获得该容器中所有 beans 的列表:

1.  使用`ListableBeanFactory`界面
2.  使用 Spring Boot 执行器

## 3。使用`ListableBeanFactory`界面

**`ListableBeanFactory`接口提供了`getBeanDefinitionNames()`方法**，该方法返回该工厂中定义的所有 beans 的名称。该接口由所有 bean 工厂实现，这些 bean 工厂预加载它们的 bean 定义，以枚举它们的所有 bean 实例。

你可以在官方文档中找到所有已知子接口及其实现类的列表。

对于这个例子，我们将使用一个 Spring Boot 应用程序。

首先，我们将创建一些春豆。让我们创建一个简单的弹簧控制器`FooController`:

```java
@Controller
public class FooController {

    @Autowired
    private FooService fooService;

    @RequestMapping(value="/displayallbeans") 
    public String getHeaderAndBody(Map model){
        model.put("header", fooService.getHeader());
        model.put("message", fooService.getBody());
        return "displayallbeans";
    }
}
```

这个控制器依赖于另一个 Spring bean `FooService`:

```java
@Service
public class FooService {

    public String getHeader() {
        return "Display All Beans";
    }

    public String getBody() {
        return "This is a sample application that displays all beans "
          + "in Spring IoC container using ListableBeanFactory interface "
          + "and Spring Boot Actuators.";
    }
}
```

注意，我们在这里创建了两个不同的 beans:

1.  `fooController`
2.  `fooService`

在执行这个应用程序时，我们将使用`applicationContext` 对象并调用它的`getBeanDefinitionNames()` 方法，这将返回我们的`applicationContext` 容器中的所有 beans:

```java
@SpringBootApplication
public class Application {
    private static ApplicationContext applicationContext;

    public static void main(String[] args) {
        applicationContext = SpringApplication.run(Application.class, args);
        displayAllBeans();
    }

    public static void displayAllBeans() {
        String[] allBeanNames = applicationContext.getBeanDefinitionNames();
        for(String beanName : allBeanNames) {
            System.out.println(beanName);
        }
    }
}
```

这将打印来自`applicationContext` 容器的所有 beans:

```java
fooController
fooService
//other beans
```

注意，除了我们定义的 bean 之外，**还将记录这个容器**中的所有其他 bean。为了清楚起见，我们在这里省略了它们，因为它们相当多。

## 4。使用 Spring Boot 执行器

Spring Boot 执行器功能提供了用于监控应用程序统计数据的端点。

它包括许多内置端点，包括/ `beans.` 这显示了我们应用程序中所有 Spring 托管 beans 的完整列表。你可以在的官方文档中找到[以上现有端点的完整列表。](https://web.archive.org/web/20220526055346/https://docs.spring.io/spring-boot/docs/current/reference/htmlsingle/#production-ready-endpoints)

现在，我们将点击 URL `http://<address>:<management-port>/beans.` ,如果我们没有指定任何单独的管理端口，我们可以使用默认的服务器端口。这将返回一个`JSON` 响应，显示 Spring IoC 容器中的所有 beans:

```java
[
    {
        "context": "application:8080",
        "parent": null,
        "beans": [
            {
                "bean": "fooController",
                "aliases": [],
                "scope": "singleton",
                "type": "com.baeldung.displayallbeans.controller.FooController",
                "resource": "file [E:/Workspace/tutorials-master/spring-boot/target
                  /classes/com/baeldung/displayallbeans/controller/FooController.class]",
                "dependencies": [
                    "fooService"
                ]
            },
            {
                "bean": "fooService",
                "aliases": [],
                "scope": "singleton",
                "type": "com.baeldung.displayallbeans.service.FooService",
                "resource": "file [E:/Workspace/tutorials-master/spring-boot/target/
                  classes/com/baeldung/displayallbeans/service/FooService.class]",
                "dependencies": []
            },
            // ...other beans
        ]
    }
]
```

当然，这也包括驻留在同一个 spring 容器中的许多其他 beans，但是为了清楚起见，我们在这里省略了它们。

如果你想了解更多关于 Spring Boot 致动器的信息，你可以直接进入主 [Spring Boot 致动器](/web/20220526055346/https://www.baeldung.com/spring-boot-actuators)指南。

## 5。结论

在本文中，我们学习了如何使用`ListableBeanFactory` 接口和 Spring Boot 执行器在`Spring IoC Container` 中显示所有的 beans。

本教程的完整实现可以在 Github 的[上找到。](https://web.archive.org/web/20220526055346/https://github.com/eugenp/tutorials/tree/master/spring-boot-modules/spring-boot-di)