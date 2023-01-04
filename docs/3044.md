# spring 中 applicationContext.xml 和 spring-servlet.xml 的区别

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-applicationcontext-vs-spring-servlet-xml>

## 1.介绍

在开发 Spring 应用程序时，有必要告诉框架在哪里寻找 beans。当应用程序启动时，框架会定位并注册所有这些应用程序，以便进一步执行。类似地，我们需要定义映射，web 应用程序的所有传入请求都将在该映射中得到处理。

所有的 Java web 框架都建立在 [servlet api](/web/20220529015741/https://www.baeldung.com/java-servlets-containers-intro) 之上。在 web 应用程序中，有三个文件起着至关重要的作用。**通常情况下，我们将它们按顺序排列为:****`web.xml`->-`applicationContext.xml`->-`spring-servlet.xml`**

在本文中，我们将看看`applicationContext`和`spring-servlet`之间的区别。

## 2.`applicationContext.xml`

[控制反转(IoC)](/web/20220529015741/https://www.baeldung.com/inversion-control-and-dependency-injection-in-spring) 是 Spring 框架的核心。在支持 IoC 的框架中，容器通常负责实例化、创建和删除对象。在春天， [`applicationContext`](/web/20220529015741/https://www.baeldung.com/spring-application-context) 扮演着 IoC 容器的角色。

当开发一个标准的 J2EE 应用程序时，我们在`web.xml`文件中声明`ContextLoaderListener`。此外，还定义了一个`contextConfigLocation` 来表示 XML 配置文件。

```
<context-param>
    <param-name>contextConfigLocation</param-name>
    <param-value>/WEB-INF/applicationContext*.xml</param-value>
</context-param>
```

当应用程序启动时，Spring 加载这个配置文件并使用它来创建一个`WebApplicationContext` 对象。在默认`,` 没有`contextConfigLocation,` 的情况下，系统会寻找`/WEB-INF/applicationContext.xml` 来加载`.`

**简而言之，`applicationContext`是春天的中枢界面。它为应用程序提供配置信息。**

在这个文件中，我们提供了与应用程序相关的配置。通常，这些是基本数据源、属性占位符文件和项目本地化的消息源，以及其他增强。

让我们看一下样本文件:

```
<?xml version="1.0" encoding="UTF-8"?>
<beans 
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xmlns:c="http://www.springframework.org/schema/c"
  xmlns:p="http://www.springframework.org/schema/p"
  xmlns:context="http://www.springframework.org/schema/context"
  xsi:schemaLocation="http://www.springframework.org/schema/beans
  http://www.springframework.org/schema/beans/spring-beans-4.1.xsd
  http://www.springframework.org/schema/context
  http://www.springframework.org/schema/context/spring-context-4.1.xsd">

    <context:property-placeholder location="classpath:/database.properties" />

    <bean id="dataSource" class="org.apache.commons.dbcp.BasicDataSource">
        <property name="driverClassName" value="${jdbc.driverClassName}" />
        <property name="url" value="${jdbc.url}" />
        <property name="username" value="${jdbc.username}" />
        <property name="password" value="${jdbc.password}" />
        <property name="initialSize" value="5" />
        <property name="maxActive" value="10" />
    </bean>

    <bean id="messageSource"
        class="org.springframework.context.support.ResourceBundleMessageSource">
        <property name="basename" value="messages" />
    </bean>
</beans>
```

**`ApplicationContext`是`BeanFactory`接口的完整超集，因此提供了`BeanFactory`的所有功能。**还提供集成的生命周期管理，自动注册`BeanPostProcessor`和`BeanFactoryPostProcessor`，方便访问`MessageSource`，发布`ApplicationEvent.`

## 3.`spring-servlet.xml`

在 Spring 中，单个前端 servlet 接收传入的请求，并将它们委托给适当的控制器方法。基于[前端控制器设计模式](/web/20220529015741/https://www.baeldung.com/java-front-controller-pattern)的前端 servlet 处理特定 web 应用程序的所有 HTTP 请求。这个前端 servlet 拥有对传入请求的所有控制权。

类似地，`spring-servlet`充当前端控制器 servlet 并提供单一入口点。它需要即将到来的 URI。在幕后，它使用`HandlerMapping` 实现来定义请求和处理程序对象之间的映射。

让我们来看看示例代码:

```
<?xml version="1.0" encoding="UTF-8"?>
<beans 
  xmlns:mvc="http://www.springframework.org/schema/mvc"
  xmlns:context="http://www.springframework.org/schema/context"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="
    http://www.springframework.org/schema/beans     
    http://www.springframework.org/schema/beans/spring-beans.xsd
    http://www.springframework.org/schema/mvc 
    http://www.springframework.org/schema/mvc/spring-mvc.xsd
    http://www.springframework.org/schema/context 
    http://www.springframework.org/schema/context/spring-context.xsd">

    <mvc:annotation-driven />
    <context:component-scan base-package="com.baeldung.controller" />

    <bean id="viewResolver"
      class="org.springframework.web.servlet.view.UrlBasedViewResolver">
	<property name="viewClass"
          value="org.springframework.web.servlet.view.JstlView" />
	<property name="prefix" value="/WEB-INF/jsp/" />
	<property name="suffix" value=".jsp" />
    </bean>

</beans>
```

## 4.`applicationContext.xml`对`spring-servlet.xml`

让我们看一下总结视图:

| **功能** | `**applicationContext.xml**` | `**spring-servlet.xml**` |
| **框架** | 它是 Spring 框架的一部分。 | 它是 Spring MVC 框架的一部分。 |
| **目的** | 定义 spring beans 的容器。 | 处理输入请求的前端控制器。 |
| **范围** | 它定义了在所有 servlets 之间共享的 beans。 | 它只定义特定于 servlet 的 beans。 |
| **管理** | 它管理像`datasource,` 这样的全局事物，其中定义了连接工厂。 | 相反，只有像控制器和`viewresolver`这样的 web 相关的东西才会在其中定义。 |
| **参考文献** | 它无法访问`spring-servlet`的 beans。 | 它可以访问在`applicationContext`中定义的 beans。 |
| **分享** | 整个应用程序共有的属性将放在这里。 | 这里将只显示特定于一个 servlet 的属性。 |
| **扫描** | 我们定义过滤器来包含/排除包。 | 我们为控制器声明组件扫描。 |
| **事件** | 在一个应用程序中定义多个上下文文件是很常见的。 | 类似地，我们可以在一个 web 应用程序中定义多个文件。 |
| **加载** | 文件 applicationContext.xml 由`ContextLoaderListener`加载。 | 文件 spring-servlet.xml 由`DispatcherServlet`加载。 |
| **必需的** | 可选择的 | 命令的 |

## 5.结论

在本教程中，我们学习了`applicationContext`和`spring-servlet` 文件。然后，我们讨论了他们在 Spring 应用程序中的角色和职责。最后，我们研究了它们之间的差异。