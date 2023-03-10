# SpringBootServletInitializer 的快速介绍

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-boot-servlet-initializer>

## 1。概述

在本教程中，我们将快速介绍一下`SpringBootServletInitializer`。

这是对`WebApplicationInitializer`的扩展，其中**从部署在 web 容器上的传统 WAR 档案**中运行一个`SpringApplication`。这个类将应用程序上下文中的`Servlet`、`Filter`和`ServletContextInitializer`bean 绑定到服务器。

扩展`SpringBootServletInitializer`类还允许我们通过覆盖`configure()`方法来配置 servlet 容器运行的应用程序。

## 2。`SpringBootServletInitializer`

为了更加实际，我们将展示一个扩展了`Initializer`类的主类的例子。

我们的名为`WarInitializerApplication`的`@SpringBootApplication`类扩展了`SpringBootServletInitializer`并覆盖了`configure()`方法。该方法使用`SpringApplicationBuilder`简单地将我们的类注册为应用程序的配置类:

```java
@SpringBootApplication
public class WarInitializerApplication extends SpringBootServletInitializer {

    @Override
    protected SpringApplicationBuilder configure(
      SpringApplicationBuilder builder) {
        return builder.sources(WarInitializerApplication.class);
    }

    public static void main(String[] args) {
        SpringApplication sa = new SpringApplication(
          WarInitializerApplication.class);
        sa.run(args);
    }

    @RestController
    public static class WarInitializerController {

        @GetMapping("/")
        public String handler() {
           // ...
        }
    }
} 
```

现在，如果我们将我们的应用程序打包成一个 WAR，我们将能够以传统方式将它部署在任何 web 容器上，这也将执行我们在`configure()`方法中添加的逻辑。

如果我们想把它打包成一个 JAR 文件，那么我们需要把相同的逻辑添加到`main()`方法中，这样嵌入的容器也可以获取它。

## 3。结论

在本文中，我们介绍了`SpringBootServletInitializer`,并展示了如何使用它从经典的 WAR 档案中运行 Spring Boot 应用程序。

这个例子的完整源代码可以在 GitHub 的[上找到。](https://web.archive.org/web/20221206125728/https://github.com/eugenp/tutorials/tree/master/spring-boot-modules/spring-boot-mvc-4)这是一个基于 Maven 的项目，因此可以导入并按原样使用。