# Spring @Autowired 字段为空–常见原因和解决方案

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-autowired-field-null>

## 1.概观

在本教程中，我们将看到导致 [`Autowired`](/web/20221108014505/https://www.baeldung.com/spring-autowire) 字段上出现 [`NullPointerException`](/web/20221108014505/https://www.baeldung.com/java-14-nullpointerexception) 的常见错误。我们还将解释如何解决这个问题。

## 2.问题的呈现

首先，让我们用一个空的`doWork`方法定义一个弹簧组件:

```
@Component
public class MyComponent {
    public void doWork() {}
}
```

然后，让我们定义我们的服务类。我们将使用 Spring capacities 在我们的服务中注入一个`MyComponent` bean，这样我们就可以在服务方法中调用`doWork`方法:

```
public class MyService {

    @Autowired
    MyComponent myComponent;

    public String serve() {
        myComponent.doWork();
        return "success";
    }
}
```

现在，让我们添加一个控制器，它将实例化一个服务并调用`serve`方法:

```
@Controller
public class MyController {

    public String control() {
        MyService userService = new MyService();
        return userService.serve();
    }
}
```

乍一看，我们的代码可能看起来非常好。但是，在运行应用程序后，调用我们的控制器的控制方法将导致以下异常:

```
java.lang.NullPointerException: null
  at com.baeldung.autowiring.service.MyService.serve(MyService.java:14)
  at com.baeldung.autowiring.controller.MyController.control(MyController.java:12)
```

这里发生了什么？当我们在控制器中调用`MyService`构造函数时，我们创建了一个不受 Spring 管理的对象。由于不知道这个`MyService`物体的存在，Spring 无法将一颗`MyComponent`豆注入其中。因此，我们创建的 MyService 对象中的`MyComponent`实例将保持为空，导致我们在尝试调用该对象上的方法时得到的 [`NullPointerException`](/web/20221108014505/https://www.baeldung.com/java-14-nullpointerexception) 。

## 3.解决办法

**为了解决这个问题，我们必须让控制器中使用的`MyService`实例成为 Spring 管理的 Bean。**

首先，让我们告诉 Spring 为我们的`MyService`类生成一个 Bean。我们有各种可能性来实现这一点。最简单的方法是用 [`@Component`](/web/20221108014505/https://www.baeldung.com/spring-component-annotation) 注释或其任何派生物来修饰`MyService`类。例如，我们可以执行以下操作:

```
@Service
public class MyService {

    @Autowired
    MyComponent myComponent;

    public String serve() {
        myComponent.doWork();
        return "success";
    }
}
```

达到相同目标的另一个替代方法是在一个`@Configuration`文件中添加一个 [`@Bean`](/web/20221108014505/https://www.baeldung.com/spring-bean-annotations) 方法:

```
@Configuration
public class MyServiceConfiguration {

    @Bean
    MyService myService() {
        return new MyService();
    }
}
```

然而，将`MyService`类变成 Spring 管理的 bean 是不够的。现在，我们必须在控制器内部自动连接它，而不是在它上面调用`new` 。让我们看看控制器的固定版本是什么样子的:

```
@Controller
public class MyController {

    @Autowired
    MyService myService;

    public String control() {
        return myService.serve();
    }
}
```

现在，调用 control 方法将按预期返回`serve`方法的结果。

## 4.结论

在本文中，我们看到了一个非常常见的错误，当我们无意中将 Spring injection 与我们通过调用它们的构造函数创建的对象混合在一起时，这个错误会导致一个 [`NullPointerException`](/web/20221108014505/https://www.baeldung.com/java-14-nullpointerexception) 。我们通过避免这个责任 mic-mac 解决了这个问题，并将我们用来管理自己的对象变成了 Spring 管理的 Bean。

与往常一样，GitHub 上的[代码是可用的。](https://web.archive.org/web/20221108014505/https://github.com/eugenp/tutorials/tree/master/spring-di-3)