# Spring 中基于 XML 的注入

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-xml-injection>

## 1。简介

在这个基础教程中，我们将学习如何使用 Spring 框架进行简单的基于 XML 的 bean 配置。

## 2。概述

让我们从在`pom.xml`中添加 Spring 的库依赖开始:

```java
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-context</artifactId>
    <version>5.1.4.RELEASE</version>         
</dependency> 
```

Spring 依赖的最新版本可以在[这里](https://web.archive.org/web/20221001115718/https://search.maven.org/classic/#search%7Cga%7C1%7Cg%3A%22org.springframework%22%20AND%20a%3A%22spring-context%22)找到。

## 3。依赖注入——概述

[依赖注入](/web/20221001115718/https://www.baeldung.com/inversion-control-and-dependency-injection-in-spring)是一种由外部容器提供对象依赖的技术。

假设我们有一个应用程序类，它依赖于一个实际处理业务逻辑的服务:

```java
public class IndexApp {
    private IService service;
    // standard constructors/getters/setters
} 
```

现在假设`IService`是一个接口:

```java
public interface IService {
    public String serve();
} 
```

这个接口可以有多种实现。

让我们快速看一下一个潜在的实现:

```java
public class IndexService implements IService {
    @Override
    public String serve() {
        return "Hello World";
    }
} 
```

这里，`IndexApp`是依赖于名为`IService`的低层组件的高层组件。

本质上，我们将`IndexApp`从`IService`的特定实现中分离出来，该实现会基于各种因素而变化。

## 4。依赖注入——在行动中

让我们看看如何注入一个依赖项。

### 4.1。使用属性

让我们看看如何使用基于 XML 的配置将依赖关系连接在一起:

```java
<bean 
  id="indexService" 
  class="com.baeldung.di.spring.IndexService" />

<bean 
  id="indexApp" 
  class="com.baeldung.di.spring.IndexApp" >
    <property name="service" ref="indexService" />
</bean> 
```

可以看到，我们正在创建一个`IndexService`的实例，并给它分配一个 id。默认情况下，该 bean 是单例的。此外，我们正在创建一个`IndexApp`的实例。

在这个 bean 中，我们使用 setter 方法注入另一个 bean。

### 4.2。使用构造函数

我们可以使用构造函数注入依赖关系，而不是通过 setter 方法注入 bean:

```java
<bean 
  id="indexApp" 
  class="com.baeldung.di.spring.IndexApp">
    <constructor-arg ref="indexService" />
</bean> 
```

### 4.3。使用静态工厂

我们也可以注入一个工厂返回的 bean。让我们创建一个简单的工厂，它根据提供的数字返回一个`IService`的实例:

```java
public class StaticServiceFactory {
    public static IService getNumber(int number) {
        // ...
    }
} 
```

现在让我们看看如何使用上述实现，通过基于 XML 的配置将 bean 注入到`IndexApp`中:

```java
<bean id="messageService"
  class="com.baeldung.di.spring.StaticServiceFactory"
  factory-method="getService">
    <constructor-arg value="1" />
</bean>   

<bean id="indexApp" class="com.baeldung.di.spring.IndexApp">
    <property name="service" ref="messageService" />
</bean> 
```

在上面的例子中，我们使用`factory-method`调用静态的`getService`方法来创建一个 id 为`messageService`的 bean，并将其注入到`IndexApp.`中

### 4.4。使用工厂方法

让我们考虑一个实例工厂，它根据提供的数字返回一个`IService`的实例。这一次，方法不是静态的:

```java
public class InstanceServiceFactory {
    public IService getNumber(int number) {
        // ...
    }
} 
```

现在让我们看看如何使用上面的实现通过 XML 配置将 bean 注入到`IndexApp`中:

```java
<bean id="indexServiceFactory" 
  class="com.baeldung.di.spring.InstanceServiceFactory" />
<bean id="messageService"
  class="com.baeldung.di.spring.InstanceServiceFactory"
  factory-method="getService" factory-bean="indexServiceFactory">
    <constructor-arg value="1" />
</bean>  
<bean id="indexApp" class="com.baeldung.di.spring.IndexApp">
    <property name="service" ref="messageService" />
</bean> 
```

在上面的例子中，我们使用`factory-method`在`InstanceServiceFactory`的一个实例上调用`getService`方法来创建一个 id 为`messageService`的 bean，并将其注入到`IndexApp`中。

## 5。测试

这是我们访问已配置 beans 的方式:

```java
@Test
public void whenGetBeans_returnsBean() {
    ApplicationContext applicationContext = new ClassPathXmlApplicationContext("...");
    IndexApp indexApp = applicationContext.getBean("indexApp", IndexApp.class);
    assertNotNull(indexApp);
} 
```

## 6。结论

在这个快速教程中，我们举例说明了如何使用 Spring Framework 通过基于 XML 的配置来注入依赖性。

这些例子的实现可以在 GitHub 项目中找到——这是一个基于 Maven 的项目，所以它应该很容易导入和运行。