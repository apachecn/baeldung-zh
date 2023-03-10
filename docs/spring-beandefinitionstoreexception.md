# spring bean definenotiontorexception

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-beandefinitionstoreexception>

## 1。概述

在本文中，我们将讨论 Spring`**org.springframework.beans.factory.BeanDefinitionStoreException**`——这通常是一个`BeanFactory`的职责。当 bean 定义无效时，该 bean 的加载就会出现问题。本文将讨论这种异常最常见的原因以及针对每种原因的解决方案。

## 2。`java.io.FileNotFoundException`起因:

潜在的`IOException`可能导致`BeanDefinitionStoreException`的原因有多种:

### 2.1。`IOException`从 ServletContext 资源解析 XML 文档

这通常发生在 Spring Web 应用程序中，当在 Spring MVC 的`web.xml`中设置了一个`DispatcherServlet`:

```java
<servlet>  
   <servlet-name>mvc</servlet-name>  
   <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>  
</servlet>
```

默认情况下，Spring 会在 web 应用程序的`/WEB-INF`目录中查找一个名为`springMvcServlet-servlet.xml`的文件。

如果该文件不存在，则会引发以下异常:

```java
org.springframework.beans.factory.BeanDefinitionStoreException: 
Ioexception Parsing Xml Document from Servletcontext Resource [/WEB-INF/mvc-servlet.xml]; 
nested exception is java.io.FileNotFoundException: 
Could not open ServletContext resource [/WEB-INF/mvc-servlet.xml]
```

**解决方案**当然是确保`mvc-servlet.xml`文件确实存在于`/WEB-INF`下；如果没有，那么可以创建一个样本:

```java
<?xml version="1.0" encoding="UTF-8"?>
<beans 

   xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
   xsi:schemaLocation="
      http://www.springframework.org/schema/beans 
      http://www.springframework.org/schema/beans/spring-beans-3.2.xsd" >

</beans>
```

### 2.2。`IOException`从类路径资源解析 XML 文档

当应用程序中的某个内容指向一个不存在的 XML 资源，或者没有放在它应该在的地方时，通常会发生这种情况。

指向这样的资源可能以多种方式发生。

以 Java 配置为例，这可能看起来像:

```java
@Configuration
@ImportResource("beans.xml")
public class SpringConfig {...}
```

在 XML 中，这将是:

```java
<import resource="beans.xml"/>
```

或者甚至手动创建一个 Spring XML 上下文:

```java
ApplicationContext context = new ClassPathXmlApplicationContext("beans.xml");
```

如果文件不存在，所有这些都会导致相同的异常:

```java
org.springframework.beans.factory.BeanDefinitionStoreException: 
Ioexception Parsing Xml Document from Servletcontext Resource [/beans.xml]; 
nested exception is java.io.FileNotFoundException: 
Could not open ServletContext resource [/beans.xml]
```

**解决方案**是创建文件，并把它放在项目的`/src/main/resources`目录下——这样，文件将存在于类路径中，并被 Spring 找到和使用。

## 3。`Could Not Resolve Placeholder …`起因:

当 Spring 试图解析一个属性，但由于许多可能的原因之一而无法解析时，就会出现这个错误。

但是首先，属性的用法——可能在 XML 中使用:

```java
... value="${some.property}" ...
```

该属性也可以用在 Java 代码中:

```java
@Value("${some.property}")
private String someProperty;
```

首先要检查的是属性的名称实际上与属性定义相匹配；在本例中，我们需要定义以下属性:

```java
some.property=someValue
```

然后，我们需要检查属性文件在 Spring 中的定义位置——这在我的[Spring 属性教程](/web/20220701021129/https://www.baeldung.com/properties-with-spring " Example of Properties with Spring")中有详细描述。一个好的最佳实践是将所有属性文件放在应用程序的`/src/main/resources`目录下，并通过以下方式加载它们:

```java
"classpath:app.properties"
```

从显而易见的地方继续——Spring 无法解析属性的另一个可能原因是，在 Spring 上下文中可能有多个`PropertyPlaceholderConfigurer`bean(或者多个`property-placeholder`元素)

如果是这种情况，那么**解决方案**要么将它们合并成一个，要么用`ignoreUnresolvablePlaceholders`在父上下文中配置一个。

## 4。`java.lang.NoSuchMethodError`起因:

这种错误有多种形式，其中最常见的是:

```java
org.springframework.beans.factory.BeanDefinitionStoreException:
Unexpected exception parsing XML document from ServletContext resource [/WEB-INF/mvc-servlet.xml];
nested exception is java.lang.NoSuchMethodError:
org.springframework.beans.MutablePropertyValues.add (Ljava/lang/String;Ljava/lang/Object;)
Lorg/springframework/beans/MutablePropertyValues;
```

当类路径中有多个 Spring 版本时，通常会发生这种情况。在项目类路径中偶然出现一个**旧版本的 Spring 比人们想象的更常见——我在 [Spring Security with Maven 文章](/web/20220701021129/https://www.baeldung.com/spring-security-with-maven#maven_problem "Spring Security with older Spring Core dependencies")中描述了这个问题和解决方案。**

简而言之，这个错误的解决方案很简单——检查类路径上的所有 Spring jars 并确保它们都有相同的版本——并且版本是 3.0 或更高版本。

类似地，例外不仅限于`MutablePropertyValues`bean——同样的问题还有其他几个实例，都是由相同的版本不一致引起的:

```java
org.springframework.beans.factory.BeanDefinitionStoreException:
Unexpected exception parsing XML document from class path resource [/WEB-INF/mvc-servlet.xml];
- nested exception is java.lang.NoSuchMethodError:
org.springframework.util.ReflectionUtils.makeAccessible(Ljava/lang/reflect/Constructor;)V
```

## 5。`java.lang.NoClassDefFoundError`起因:

与 Maven 和现有的 Spring 依赖项类似的一个常见问题是:

```java
org.springframework.beans.factory.BeanDefinitionStoreException:
Unexpected exception parsing XML document from ServletContext resource [/WEB-INF/mvc-servlet.xml];
nested exception is java.lang.NoClassDefFoundError: 
org/springframework/transaction/interceptor/TransactionInterceptor
```

当在 XML 配置中配置了事务功能时，会出现这种情况:

```java
<tx:annotation-driven/>
```

`NoClassDefFoundError`意味着 Spring 事务支持——即`spring-tx`——在类路径中不存在。

解决方案很简单—`spring-tx`需要在 Maven pom 中定义:

```java
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-tx</artifactId>
    <version>4.1.0.RELEASE</version>
</dependency>
```

当然，这不仅限于事务功能——如果 AOP 丢失，也会抛出类似的错误:

```java
Exception in thread "main" org.springframework.beans.factory.BeanDefinitionStoreException: 
Unexpected exception parsing XML document from class path resource [/WEB-INF/mvc-servlet.xml]; 
nested exception is java.lang.NoClassDefFoundError: 
org/aopalliance/aop/Advice
```

现在需要的 jar 是:`spring-aop`(以及隐式的`aopalliance`):

```java
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-aop</artifactId>
    <version>4.1.0.RELEASE</version>
</dependency>
```

## 6。结论

在这篇文章的结尾，我们应该有一个清晰的地图来导航可能导致`Bean Definition Store Exception`的各种原因和问题，以及如何解决所有这些问题。

在 github 项目中可以找到这些异常示例的实现——这是一个基于 Eclipse 的项目，因此应该很容易导入和运行。