# Spring 处理程序映射指南

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-handler-mappings>

## 1。简介

在 Spring MVC 中，`DispatcherServlet`充当[前端控制器](/web/20220524010413/https://www.baeldung.com/java-front-controller-pattern)——接收所有传入的 HTTP 请求并处理它们。

简而言之，在处理程序映射 `.`的帮助下，通过将请求传递给相关组件**来进行处理**

`HandlerMapping`是定义请求和[处理程序对象](/web/20220524010413/https://www.baeldung.com/spring-mvc-handler-adapters)之间映射的接口。虽然 Spring MVC 框架提供了一些现成的实现，但是开发人员可以通过实现接口来提供定制的映射策略。

本文讨论了 Spring MVC 提供的一些实现，即`BeanNameUrlHandlerMapping`、`SimpleUrlHandlerMapping`、`ControllerClassNameHandlerMapping`，它们的配置，以及它们之间的区别。

## 2。 `**BeanNameUrlHandlerMapping**`

`BeanNameUrlHandlerMapping`是默认的`HandlerMapping`实现。`BeanNameUrlHandlerMapping`将请求 URL 映射到同名的 beans。

这种特殊的映射支持直接名称匹配，也支持使用“*”模式的模式匹配。

例如，传入的 URL `“/foo”` 映射到一个名为`“/foo”`的 bean。模式映射的一个例子是将对`“/foo*”`的请求映射到名称以`“/foo”`开头的 beans，如`“/foo2/”`或`“/fooOne/”`。

让我们在这里配置这个例子，并注册一个 bean 控制器来处理对`“/beanNameUrl”`的请求:

```java
@Configuration
public class BeanNameUrlHandlerMappingConfig {
    @Bean
    BeanNameUrlHandlerMapping beanNameUrlHandlerMapping() {
        return new BeanNameUrlHandlerMapping();
    }

    @Bean("/beanNameUrl")
    public WelcomeController welcome() {
        return new WelcomeController();
    }
}
```

这是上述基于 Java 的配置的 XML 等效形式:

```java
<bean class="org.springframework.web.servlet.handler.BeanNameUrlHandlerMapping" />
<bean name="/beanNameUrl" class="com.baeldung.WelcomeController" /> 
```

值得注意的是，在这两种配置中，**为`BeanNameUrlHandlerMapping`定义 bean 并不需要**，因为它是由 Spring MVC 提供的。删除这个 bean 定义不会导致任何问题，请求仍然会被映射到它们注册的处理程序 bean。

现在所有对 `“/beanNameUrl”`的请求都将由`DispatcherServlet`转发给`WelcomeController`。`WelcomeController`返回一个名为`welcome`的视图名。

以下代码测试此配置，并确保返回正确的视图名称:

```java
public class BeanNameMappingConfigTest {
    // ...

    @Test
    public void whenBeanNameMapping_thenMappedOK() {
        mockMvc.perform(get("/beanNameUrl"))
          .andExpect(status().isOk())
          .andExpect(view().name("welcome"));
    }
}
```

## 3。`SimpleUrlHandlerMapping`

接下来，`SimpleUrlHandlerMapping`是最灵活的`HandlerMapping`实现。它允许 bean 实例和 URL 之间或者 bean 名称和 URL 之间的直接和声明性映射。

让我们将请求`“/simpleUrlWelcome”`和`“/*/simpleUrlWelcome”`映射到`“welcome”` bean:

```java
@Configuration
public class SimpleUrlHandlerMappingConfig {

    @Bean
    public SimpleUrlHandlerMapping simpleUrlHandlerMapping() {
        SimpleUrlHandlerMapping simpleUrlHandlerMapping
          = new SimpleUrlHandlerMapping();

        Map<String, Object> urlMap = new HashMap<>();
        urlMap.put("/simpleUrlWelcome", welcome());
        simpleUrlHandlerMapping.setUrlMap(urlMap);

        return simpleUrlHandlerMapping;
    }

    @Bean
    public WelcomeController welcome() {
        return new WelcomeController();
    }
}
```

或者，下面是等效的 XML 配置:

```java
<bean class="org.springframework.web.servlet.handler.SimpleUrlHandlerMapping">
    <property name="mappings">
        <value>
            /simpleUrlWelcome=welcome
            /*/simpleUrlWelcome=welcome
        </value>
    </property>
</bean>
<bean id="welcome" class="com.baeldung.WelcomeController" />
```

需要注意的是，在 XML 配置中，`“<value>”`标签之间的映射必须以`java.util.Properties`类接受的形式完成，并且应该遵循语法:`path= Handler_Bean_Name`。

URL 通常应该以斜杠开头，但是，如果路径不是以斜杠开头，Spring MVC 会自动添加它。

在 XML 中配置上述示例的另一种方式是使用`“props”`属性，而不是`“value”`。`Props`有一列`“prop”`标签，每个标签定义一个映射，其中`“key”`引用映射的 URL，标签的值是 bean 的名称。

```java
<bean class="org.springframework.web.servlet.handler.SimpleUrlHandlerMapping">
    <property name="mappings">
        <props>
            <prop key="/simpleUrlWelcome">welcome</prop>
            <prop key="/*/simpleUrlWelcome">welcome</prop>
        </props>
    </property>
</bean>
```

下面的测试用例确保对“/ `simpleUrlWelcome`”的请求由“`WelcomeController”` 处理，它返回一个名为 `“welcome”` 的视图名:

```java
public class SimpleUrlMappingConfigTest {
    // ...

    @Test
    public void whenSimpleUrlMapping_thenMappedOK() {
        mockMvc.perform(get("/simpleUrlWelcome"))
          .andExpect(status().isOk())
          .andExpect(view().name("welcome"));
    }
}
```

## 4。`ControllerClassNameHandlerMapping`(5 年春移除)

`ControllerClassNameHandlerMapping`将 URL 映射到一个注册的控制器 bean(或者一个用`@Controller`注释注释的控制器),该控制器 bean 具有相同的名称或者以相同的名称开头。

在许多情况下，特别是对于处理单一请求类型的简单控制器实现，这可能更方便。Spring MVC 使用的惯例是使用类的名称并去掉后缀`“Controller”`，然后将名称改为小写，并作为带有前导`“/”`的映射返回。

例如，`“WelcomeController”`将作为到`“/welcome*”`的映射返回，也就是到任何以`“welcome”`开头的 URL。

我们来配置一下`ControllerClassNameHandlerMapping`:

```java
@Configuration
public class ControllerClassNameHandlerMappingConfig {

    @Bean
    public ControllerClassNameHandlerMapping controllerClassNameHandlerMapping() {
        return new ControllerClassNameHandlerMapping();
    }

    @Bean
    public WelcomeController welcome() {
        return new WelcomeController();
    }
}
```

请注意，从 Spring 4.3 开始，`ControllerClassNameHandlerMapping`被**取代，取而代之的是注释驱动的处理程序方法。**

另一个重要注意事项是，控制器名称将始终以小写形式返回(减去“控制器”后缀)。因此，如果我们有一个名为“`WelcomeBaeldungController`”的控制器，它将只处理对`“/welcomebaeldung”`的请求，而不处理对`“/welcomeBaeldung”`的请求。

在下面的 Java config 和 XML config 中，我们定义了`ControllerClassNameHandlerMapping` bean，并为我们将用来处理请求的控制器注册了 bean。我们还注册了一个类型为`“WelcomeController”`的 bean，该 bean 将处理所有以 `“/welcome”`开头的请求。

下面是等效的 XML 配置:

```java
<bean class="org.springframework.web.servlet.mvc.support.ControllerClassNameHandlerMapping" />
<bean class="com.baeldung.WelcomeController" /> 
```

使用上述配置时，对“/ `welcome`”的请求将由“`WelcomeController`”处理。

以下代码将确保对“/ `welcome` *”的请求(如“/ `welcometest`”)由“WelcomeController”处理，它返回一个名为“`welcome`”的视图名称:

```java
public class ControllerClassNameHandlerMappingTest {
    // ...

    @Test
    public void whenControllerClassNameMapping_thenMappedOK() {
        mockMvc.perform(get("/welcometest"))
          .andExpect(status().isOk())
          .andExpect(view().name("welcome"));
    }
}
```

## 5。配置优先级

Spring MVC 框架允许同时实现多个`HandlerMapping`接口。

让我们创建一个配置并注册两个控制器，都映射到 URL“/welcome”，只是使用不同的映射并返回不同的视图名称:

```java
@Configuration
public class HandlerMappingDefaultConfig {

    @Bean("/welcome")
    public BeanNameHandlerMappingController beanNameHandlerMapping() {
        return new BeanNameHandlerMappingController();
    }

    @Bean
    public WelcomeController welcome() {
        return new WelcomeController();
    }
}
```

如果没有注册显式的处理程序映射器，将使用默认的`BeanNameHandlerMapping`。让我们通过测试来证明这种行为:

```java
@Test
public void whenConfiguringPriorities_thenMappedOK() {
    mockMvc.perform(get("/welcome"))
      .andExpect(status().isOk())
      .andExpect(view().name("bean-name-handler-mapping"));
} 
```

如果我们显式注册一个不同的处理程序映射器，默认映射器将被覆盖。然而，有趣的是，当两个映射器被显式注册时会发生什么:

```java
@Configuration
public class HandlerMappingPrioritiesConfig {

    @Bean
    BeanNameUrlHandlerMapping beanNameUrlHandlerMapping() {
        BeanNameUrlHandlerMapping beanNameUrlHandlerMapping 
          = new BeanNameUrlHandlerMapping();
        return beanNameUrlHandlerMapping;
    }

    @Bean
    public SimpleUrlHandlerMapping simpleUrlHandlerMapping() {
        SimpleUrlHandlerMapping simpleUrlHandlerMapping
          = new SimpleUrlHandlerMapping();
        Map<String, Object> urlMap = new HashMap<>();
        urlMap.put("/welcome", simpleUrlMapping());
        simpleUrlHandlerMapping.setUrlMap(urlMap);
        return simpleUrlHandlerMapping;
    }

    @Bean
    public SimpleUrlMappingController simpleUrlMapping() {
        return new SimpleUrlMappingController();
    }

    @Bean("/welcome")
    public BeanNameHandlerMappingController beanNameHandlerMapping() {
        return new BeanNameHandlerMappingController();
    }
}
```

为了控制使用哪种映射，使用 `setOrder(int order)`方法设置优先级。该方法采用一个`int`参数，其中较低的值意味着较高的优先级。

在 XML 配置中，您可以使用名为`“order”`的属性来配置优先级:

```java
<bean class="org.springframework.web.servlet.handler.BeanNameUrlHandlerMapping">
    <property name="order" value="2" />
</bean> 
```

让我们通过跟随`beanNameUrlHandlerMapping.setOrder(1)` 和`simpleUrlHandlerMapping.setOrder(0).` 将`order`属性添加到处理程序映射 beans 中，`order` 属性的较低值反映了较高的优先级。让我们通过测试来证明新的行为:

```java
@Test
public void whenConfiguringPriorities_thenMappedOK() {
    mockMvc.perform(get("/welcome"))
      .andExpect(status().isOk())
      .andExpect(view().name("simple-url-handler-mapping"));
}
```

当测试上述配置时，您会看到对`“/welcome”`的请求将由`SimpleUrlHandlerMapping` bean 处理，该 bean 调用`SimpleUrlHandlerController`并返回`simple-url-handler-mapping` 视图。通过相应地调整`order`属性的值，我们可以很容易地将`BeanNameHandlerMapping` 配置为优先。

## 6。结论

在本文中，我们通过探索 Spring MVC 框架中的不同实现，讨论了 URL 映射是如何在该框架中处理的。

本文附带的代码可以在 GitHub 上找到[。](https://web.archive.org/web/20220524010413/https://github.com/eugenp/tutorials/tree/master/spring-web-modules/spring-mvc-basics)