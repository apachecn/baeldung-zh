# 春豆瞄准镜快速指南

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-bean-scopes>

## 1。概述

在这个快速教程中，我们将了解 Spring 框架中不同类型的 bean 作用域。

bean 的范围定义了 bean 的生命周期和在我们使用它的上下文中的可见性。

Spring 框架的最新版本定义了 6 种类型的作用域:

*   一个
*   原型
*   请求
*   会议
*   应用
*   websocket

最后提到的四个作用域，`request, session, application` 和`websocket`，仅在 web 感知应用程序中可用。

## 延伸阅读:

## [什么是春豆？](/web/20221002235014/https://www.baeldung.com/spring-bean)

A quick and practical explanation of what a Spring Bean is.[Read more](/web/20221002235014/https://www.baeldung.com/spring-bean) →

## [春豆注解](/web/20221002235014/https://www.baeldung.com/spring-bean-annotations)

Learn how and when to use the standard Spring bean annotations - @Component, @Repository, @Service and @Controller.[Read more](/web/20221002235014/https://www.baeldung.com/spring-bean-annotations) →

## 2。单例范围

当我们用`singleton`范围定义一个 bean 时，容器创建了该 bean 的一个实例；对该 bean 名称的所有请求都将返回同一个被缓存的对象。对该对象的任何修改都将反映在对该 bean 的所有引用中。如果没有指定其他范围，此范围是默认值。

让我们创建一个`Person`实体来举例说明作用域的概念:

```java
public class Person {
    private String name;

    // standard constructor, getters and setters
}
```

之后，我们通过使用`@Scope`注释来定义具有`singleton`范围的 bean:

```java
@Bean
@Scope("singleton")
public Person personSingleton() {
    return new Person();
}
```

我们也可以用一个常数代替`String`值，如下所示:

```java
@Scope(value = ConfigurableBeanFactory.SCOPE_SINGLETON)
```

现在，我们可以继续编写一个测试，显示引用同一个 bean 的两个对象将具有相同的值，即使其中只有一个对象更改了它们的状态，因为它们都引用同一个 bean 实例:

```java
private static final String NAME = "John Smith";

@Test
public void givenSingletonScope_whenSetName_thenEqualNames() {
    ApplicationContext applicationContext = 
      new ClassPathXmlApplicationContext("scopes.xml");

    Person personSingletonA = (Person) applicationContext.getBean("personSingleton");
    Person personSingletonB = (Person) applicationContext.getBean("personSingleton");

    personSingletonA.setName(NAME);
    Assert.assertEquals(NAME, personSingletonB.getName());

    ((AbstractApplicationContext) applicationContext).close();
}
```

本例中的`scopes.xml`文件应该包含所用 beans 的 xml 定义:

```java
<?xml version="1.0" encoding="UTF-8"?>
<beans 
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://www.springframework.org/schema/beans 
    http://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="personSingleton" class="org.baeldung.scopes.Person" scope="singleton"/>    
</beans>
```

## 3。原型范围

具有`prototype`作用域的 bean 将在每次从容器中被请求时返回不同的实例。它是通过将值`prototype`设置为 bean 定义中的 *@Scope* 注释来定义的:

```java
@Bean
@Scope("prototype")
public Person personPrototype() {
    return new Person();
}
```

我们也可以像对待`singleton`范围一样使用一个常量:

```java
@Scope(value = ConfigurableBeanFactory.SCOPE_PROTOTYPE)
```

我们现在将编写一个与之前类似的测试，显示两个对象请求相同的 bean 名称和`prototype `范围。它们将具有不同的状态，因为它们不再引用同一个 bean 实例:

```java
private static final String NAME = "John Smith";
private static final String NAME_OTHER = "Anna Jones";

@Test
public void givenPrototypeScope_whenSetNames_thenDifferentNames() {
    ApplicationContext applicationContext = 
      new ClassPathXmlApplicationContext("scopes.xml");

    Person personPrototypeA = (Person) applicationContext.getBean("personPrototype");
    Person personPrototypeB = (Person) applicationContext.getBean("personPrototype");

    personPrototypeA.setName(NAME);
    personPrototypeB.setName(NAME_OTHER);

    Assert.assertEquals(NAME, personPrototypeA.getName());
    Assert.assertEquals(NAME_OTHER, personPrototypeB.getName());

    ((AbstractApplicationContext) applicationContext).close();
} 
```

`scopes.xml`文件类似于上一节中给出的文件，只是添加了范围为`prototype`的 bean 的 xml 定义:

```java
<bean id="personPrototype" class="org.baeldung.scopes.Person" scope="prototype"/>
```

## 4。网络感知范围

如前所述，有四个额外的作用域只在支持 web 的应用程序上下文中可用。我们在实践中很少使用这些。

`request`作用域为单个 HTTP 请求创建一个 bean 实例，而 s `ession`作用域为一个 HTTP 会话创建一个 bean 实例。

`application` 范围为`ServletContext`的生命周期创建 bean 实例，而`websocket` 范围为特定的`WebSocket` 会话创建 bean 实例。

让我们创建一个用于实例化 beans 的类:

```java
public class HelloMessageGenerator {
    private String message;

    // standard getter and setter
}
```

### 4.1。请求范围

我们可以使用`@Scope`注释定义具有`request`范围的 bean:

```java
@Bean
@Scope(value = WebApplicationContext.SCOPE_REQUEST, proxyMode = ScopedProxyMode.TARGET_CLASS)
public HelloMessageGenerator requestScopedBean() {
    return new HelloMessageGenerator();
}
```

属性`proxyMode`是必需的，因为在 web 应用程序上下文实例化的时候，没有活动的请求。Spring 创建一个作为依赖注入的代理，并在请求需要时实例化目标 bean。

我们还可以使用一个`@RequestScope`组合注释，作为上述定义的快捷方式:

```java
@Bean
@RequestScope
public HelloMessageGenerator requestScopedBean() {
    return new HelloMessageGenerator();
}
```

接下来我们可以定义一个控制器，它有一个对`requestScopedBean`的注入引用。为了测试特定于 web 的范围，我们需要访问同一个请求两次。

如果我们在每次请求运行时显示`message`，我们可以看到该值被重置为`null`，即使它后来在方法中被更改。这是因为为每个请求返回了不同的 bean 实例。

```java
@Controller
public class ScopesController {
    @Resource(name = "requestScopedBean")
    HelloMessageGenerator requestScopedBean;

    @RequestMapping("/scopes/request")
    public String getRequestScopeMessage(final Model model) {
        model.addAttribute("previousMessage", requestScopedBean.getMessage());
        requestScopedBean.setMessage("Good morning!");
        model.addAttribute("currentMessage", requestScopedBean.getMessage());
        return "scopesExample";
    }
}
```

### 4.2。会话范围

我们可以用类似的方式定义具有*会话*范围的 bean:

```java
@Bean
@Scope(value = WebApplicationContext.SCOPE_SESSION, proxyMode = ScopedProxyMode.TARGET_CLASS)
public HelloMessageGenerator sessionScopedBean() {
    return new HelloMessageGenerator();
}
```

我们还可以使用一个专用的组合注释来简化 bean 定义:

```java
@Bean
@SessionScope
public HelloMessageGenerator sessionScopedBean() {
    return new HelloMessageGenerator();
}
```

接下来，我们参照`sessionScopedBean`定义一个控制器。同样，我们需要运行两个请求，以显示`message`字段的值对于会话是相同的。

在这种情况下，当第一次发出请求时，值`message`是`null.`,但是，一旦它被更改，该值将为后续请求保留，因为为整个会话返回相同的 bean 实例。

```java
@Controller
public class ScopesController {
    @Resource(name = "sessionScopedBean")
    HelloMessageGenerator sessionScopedBean;

    @RequestMapping("/scopes/session")
    public String getSessionScopeMessage(final Model model) {
        model.addAttribute("previousMessage", sessionScopedBean.getMessage());
        sessionScopedBean.setMessage("Good afternoon!");
        model.addAttribute("currentMessage", sessionScopedBean.getMessage());
        return "scopesExample";
    }
}
```

### 4.3。适用范围

`application` 范围为`ServletContext.`的生命周期创建 bean 实例

这类似于`singleton`范围，但是关于 bean 的范围有一个非常重要的区别。

当 bean 的作用域为`application`时，bean 的同一个实例被运行在同一个`ServletContext`中的多个基于 servlet 的应用程序共享，而`singleton`作用域的 bean 的作用域仅为一个应用程序上下文。

让我们创建具有`application`范围的 bean:

```java
@Bean
@Scope(
  value = WebApplicationContext.SCOPE_APPLICATION, proxyMode = ScopedProxyMode.TARGET_CLASS)
public HelloMessageGenerator applicationScopedBean() {
    return new HelloMessageGenerator();
}
```

类似于`request`和`session`范围，我们可以使用一个更短的版本:

```java
@Bean
@ApplicationScope
public HelloMessageGenerator applicationScopedBean() {
    return new HelloMessageGenerator();
}
```

现在让我们创建一个引用这个 bean 的控制器:

```java
@Controller
public class ScopesController {
    @Resource(name = "applicationScopedBean")
    HelloMessageGenerator applicationScopedBean;

    @RequestMapping("/scopes/application")
    public String getApplicationScopeMessage(final Model model) {
        model.addAttribute("previousMessage", applicationScopedBean.getMessage());
        applicationScopedBean.setMessage("Good afternoon!");
        model.addAttribute("currentMessage", applicationScopedBean.getMessage());
        return "scopesExample";
    }
}
```

在这种情况下，一旦在`applicationScopedBean`中设置，值`message` 将被保留用于所有后续请求、会话，甚至用于将访问该 bean 的不同 servlet 应用程序，只要它运行在同一个`ServletContext.`中

### 4.4。WebSocket 作用域

最后，让我们创建具有 *websocket* 作用域的 bean:

```java
@Bean
@Scope(scopeName = "websocket", proxyMode = ScopedProxyMode.TARGET_CLASS)
public HelloMessageGenerator websocketScopedBean() {
    return new HelloMessageGenerator();
}
```

第一次访问时，`WebSocket`范围内的 beans 存储在`WebSocket`会话属性中。在整个`WebSocket`会话期间，无论何时访问该 bean，都会返回该 bean 的同一个实例。

我们也可以说它展示了单例行为，但仅限于一个`W` `ebSocket`会话。

## 5。结论

在本文中，我们讨论了 Spring 提供的不同 bean 作用域以及它们的预期用途。

本文的实现可以在[GitHub 项目](https://web.archive.org/web/20221002235014/https://github.com/eugenp/tutorials/tree/master/spring-core-2 "The Full Registration Example Project on Github ")中找到。