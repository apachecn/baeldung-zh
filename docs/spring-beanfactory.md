# 春天豆制品厂指南

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-beanfactory>

## 1.介绍

本文将关注于**探索 Spring BeanFactory API** 。

BeanFactory 接口提供了一个简单而灵活的配置机制，通过 Spring IoC 容器来管理任何性质的对象。在深入研究这个中央 Spring API 之前，让我们先来看看一些基础知识。

## 2.基础——bean 和容器

简单地说，beans 是构成 Spring 应用程序主干的 java 对象，由 Spring IoC 容器管理。除了由容器管理之外，bean 没有什么特别的(在所有其他方面，它是应用程序中许多对象中的一个)。

Spring 容器负责实例化、配置和组装 beans。容器通过读取我们为应用程序定义的配置元数据来获取关于实例化、配置和管理哪些对象的信息。

## 3.Maven 依赖性

让我们将所需的 Maven [依赖项](https://web.archive.org/web/20220626081250/https://search.maven.org/classic/#search%7Cga%7C1%7Cg%3A%22org.springframework%22%20AND%20a%3A%22spring-beans%22)添加到`pom.xml` 文件中。我们将使用 Spring Beans 依赖项来设置 BeanFactory:

```java
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-beans</artifactId>
    <version>5.2.8.RELEASE</version>
</dependency>
```

## 4.`BeanFactory`界面

有趣的是，我们先来看看`org.springframework.beans.factory`包中的[接口定义](https://web.archive.org/web/20220626081250/https://github.com/spring-projects/spring-framework/blob/master/spring-beans/src/main/java/org/springframework/beans/factory/BeanFactory.java)，并在这里讨论它的一些重要 API。

### 4.1。`getBean()`API

各种版本的 `[getBean()](https://web.archive.org/web/20220626081250/https://docs.spring.io/spring/docs/5.0.x/javadoc-api/org/springframework/beans/factory/BeanFactory.html#getBean-java.lang.String-)`方法返回指定 bean 的一个实例，该实例可以在整个应用程序中共享或独立。

### 4.2。`containsBean()` API

该方法确认该 bean 工厂是否包含具有给定名称的 bean。更具体地说，它确认 [`getBean(java.lang.String)`](https://web.archive.org/web/20220626081250/https://docs.spring.io/spring/docs/5.0.x/javadoc-api/org/springframework/beans/factory/BeanFactory.html#getBean-java.lang.String-) 是否能够获得具有给定名称的 bean 实例。

### 4.3。`isSingleton()` API

`isSingleton` API 可以用来查询这个 bean 是否是一个共享的单例。也就是说如果 [`getBean(java.lang.String)`](https://web.archive.org/web/20220626081250/https://docs.spring.io/spring/docs/5.0.x/javadoc-api/org/springframework/beans/factory/BeanFactory.html#getBean-java.lang.String-) 将总是返回相同的实例。

### 4.4。`isPrototype()` API

这个 API 将确认 [`getBean(java.lang.String)`](https://web.archive.org/web/20220626081250/https://docs.spring.io/spring/docs/5.0.x/javadoc-api/org/springframework/beans/factory/BeanFactory.html#getBean-java.lang.String-) 是否返回独立的实例——这意味着 bean 是否配置了原型范围。

需要注意的重要一点是，这个返回`false`的方法并没有明确指出一个单例对象。它表示非独立的实例，也可能对应于其他范围。

我们需要使用 [`isSingleton(java.lang.String)`](https://web.archive.org/web/20220626081250/https://docs.spring.io/spring/docs/5.0.x/javadoc-api/org/springframework/beans/factory/BeanFactory.html#isSingleton-java.lang.String-) 操作来显式检查共享的单例实例。

### 4.5。其他 API

当`isTypeMatch(String name, Class targetType)`方法检查具有给定名称的 bean 是否匹配指定的类型时，`getType(String name)`在识别具有给定名称的 bean 的类型时很有用。

最后，`getAliases(String name)`返回给定 bean 名称的别名，如果有的话。

## 5\. `BeanFactory` API

`BeanFactory`保存 bean 定义，并在客户端应用程序请求时实例化它们——这意味着:

*   它通过实例化 bean 并调用适当的销毁方法来处理 bean 的生命周期
*   它能够在实例化相关对象时在它们之间创建关联
*   需要指出的是`BeanFactory`不支持基于注释的依赖注入，而 BeanFactory 的超集`ApplicationContext`支持

请务必阅读[应用程序上下文](https://web.archive.org/web/20220626081250/https://docs.spring.io/spring/docs/3.0.0.M3/reference/html/ch04s08.html),看看它还能做些什么。

## 6.定义 Bean

让我们定义一个简单的 bean:

```java
public class Employee {
    private String name;
    private int age;

    // standard constructors, getters and setters
}
```

## 7.用 XML 配置`BeanFactory`

我们可以用 XML 配置`BeanFactory`。让我们创建一个文件`bean factory-example.xml:`

```java
<bean id="employee" class="com.baeldung.beanfactory.Employee">
    <constructor-arg name="name" value="Hello! My name is Java"/>
    <constructor-arg name="age" value="18"/>
</bean>    
<alias name="employee" alias="empalias"/>
```

注意，我们还为`employee` bean 创建了一个别名。

## 8.`BeanFactory`同`ClassPathResource`

`ClassPathResource`属于`org.springframework.core.io`包。让我们运行一个快速测试，并使用`ClassPathResource`初始化`XmlBeanFactory`，如下所示:

```java
public class BeanFactoryWithClassPathResourceTest {

    @Test
    public void createBeanFactoryAndCheckEmployeeBean() {
        Resource res = new ClassPathResource("beanfactory-example.xml");
        BeanFactory factory = new XmlBeanFactory(res);
        Employee emp = (Employee) factory.getBean("employee");

        assertTrue(factory.isSingleton("employee"));
        assertTrue(factory.getBean("employee") instanceof Employee);
        assertTrue(factory.isTypeMatch("employee", Employee.class));
        assertTrue(factory.getAliases("employee").length > 0);
    }
}
```

## 9.结论

在这篇简短的文章中，我们了解了 Spring `BeanFactory` API 提供的主要方法，以及一个说明配置及其用法的例子。

支持这些例子的代码都可以在 GitHub 的[上找到。](https://web.archive.org/web/20220626081250/https://github.com/eugenp/tutorials/tree/master/spring-core-3/src/test/java/com/baeldung/beanfactory)