# 找不到 XML 架构命名空间的 Spring NamespaceHandler

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/unable-to-locate-spring-namespacehandler-for-xml-schema-namespace>

## 1。问题

本文将讨论 Spring 中最常见的配置问题之一—**一个 Spring 名称空间的名称空间处理程序找不到了**。大多数情况下，这意味着类路径中缺少一个特定的 Spring jar 所以让我们来看看这些缺少的模式可能是什么，以及每个模式缺少的依赖项是什么。

## 延伸阅读:

## [基于 XML 的 Spring 注入](/web/20220120025304/https://www.baeldung.com/spring-xml-injection)

Learn how to perform an XML-based injection with Spring.[Read more](/web/20220120025304/https://www.baeldung.com/spring-xml-injection) →

## [web.xml vs 带有 Spring 的初始化器](/web/20220120025304/https://www.baeldung.com/spring-xml-vs-java-config)

A quick and practical guide to XML and Java config in Spring.[Read more](/web/20220120025304/https://www.baeldung.com/spring-xml-vs-java-config) →

## [Top Spring 框架面试问题](/web/20220120025304/https://www.baeldung.com/spring-interview-questions)

A quick discussion of common questions about the Spring Framework that might come up during a job interview.[Read more](/web/20220120025304/https://www.baeldung.com/spring-interview-questions) →

## 2。`http://www.springframework.org/schema/security`

[安全名称空间](https://web.archive.org/web/20220120025304/http://static.springsource.org/spring-security/site/docs/3.1.x/reference/ns-config.html "Spring 3.1 Reference - Security Namespace Configuration")不可用是迄今为止实践中最常遇到的问题:

```java
<?xml version="1.0" encoding="UTF-8"?>
<beans:beans  
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" 
    xmlns:beans="http://www.springframework.org/schema/beans"
    xsi:schemaLocation="
        http://www.springframework.org/schema/security 
        http://www.springframework.org/schema/security/spring-security-3.2.xsd
        http://www.springframework.org/schema/beans 
        http://www.springframework.org/schema/beans/spring-beans-4.1.xsd">

</beans:beans>
```

这导致了以下异常:

```java
org.springframework.beans.factory.parsing.BeanDefinitionParsingException: 
Configuration problem: 
Unable to locate Spring NamespaceHandler for XML schema namespace 
[http://www.springframework.org/schema/security]
Offending resource: class path resource [securityConfig.xml]
```

解决方案很简单——项目的类路径中缺少`spring-security-config`依赖项:

```java
<dependency> 
   <groupId>org.springframework.security</groupId>
   <artifactId>spring-security-config</artifactId>
   <version>3.2.5.RELEASE</version>
</dependency>
```

这将把正确的名称空间处理程序——在本例中是`SecurityNamespaceHandler`——放到类路径上，并准备好解析`security`名称空间中的元素。

完整的 Spring 安全设置的完整 Maven 配置可以在我之前的 [Maven 教程](/web/20220120025304/https://www.baeldung.com/spring-security-with-maven "Spring Security with Maven")中找到。

## 3。`http://www.springframework.org/schema/aop`

当在类路径中没有必需的 aop spring 库的情况下使用**`aop`名称空间**时，也会出现同样的问题:

```java
<beans 

    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" 
    xmlns:aop="http://www.springframework.org/schema/aop"
    xsi:schemaLocation="
        http://www.springframework.org/schema/beans 
        http://www.springframework.org/schema/beans/spring-beans-4.1.xsd
        http://www.springframework.org/schema/aop
        http://www.springframework.org/schema/aop/spring-aop-4.1.xsd">

</beans>
```

确切的例外是:

```java
org.springframework.beans.factory.parsing.BeanDefinitionParsingException: 
Configuration problem: 
Unable to locate Spring NamespaceHandler for XML schema namespace 
[http://www.springframework.org/schema/aop]
Offending resource: ServletContext resource [/WEB-INF/webConfig.xml]
```

解决方案是类似的——需要将`spring-aop` jar 添加到项目的类路径中:

```java
<dependency>
   <groupId>org.springframework</groupId>
   <artifactId>spring-aop</artifactId>
   <version>4.1.0.RELEASE</version>
</dependency>
```

在这种情况下，添加新的依赖项后，`AopNamespaceHandler`将出现在类路径中。

## 4。`http://www.springframework.org/schema/tx`

使用**事务名称空间**——一个小但非常有用的名称空间，用于配置事务语义:

```java
<beans 

    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" 
    xmlns:tx="http://www.springframework.org/schema/tx"
    xsi:schemaLocation="
        http://www.springframework.org/schema/beans 
        http://www.springframework.org/schema/beans/spring-beans-4.1.xsd
        http://www.springframework.org/schema/tx
        http://www.springframework.org/schema/tx/spring-tx-4.1.xsd">

</beans>
```

如果正确的 jar 不在类路径上，也会导致异常:

```java
org.springframework.beans.factory.parsing.BeanDefinitionParsingException: 
Configuration problem: 
Unable to locate Spring NamespaceHandler for XML schema namespace
[http://www.springframework.org/schema/tx]
Offending resource: class path resource [daoConfig.xml]
```

这里缺少的依赖项是`spring-tx`:

```java
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-tx</artifactId>
    <version>4.1.0.RELEASE</version>
</dependency>
```

现在，右边的`NamspaceHandler`——即`TxNamespaceHandler`——将出现在类路径上，允许使用 XML 和注释进行声明式事务管理。

## 5。`http://www.springframework.org/schema/mvc`

向前移动到**`mvc`命名空间**:

```java
<beans 

    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" 
    xmlns:tx="http://www.springframework.org/schema/mvc"
    xsi:schemaLocation="
        http://www.springframework.org/schema/beans 
        http://www.springframework.org/schema/beans/spring-beans-4.1.xsd
        http://www.springframework.org/schema/mvc
        http://www.springframework.org/schema/mvc/spring-mvc-4.1.xsd">

</beans>
```

缺少依赖关系将导致以下异常:

```java
org.springframework.beans.factory.parsing.BeanDefinitionParsingException: 
Configuration problem: 
Unable to locate Spring NamespaceHandler for XML schema namespace
[http://www.springframework.org/schema/mvc]
Offending resource: class path resource [webConfig.xml]
```

在这种情况下，缺少的依赖项是`spring-mvc`:

```java
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-webmvc</artifactId>
    <version>4.1.0.RELEASE</version>
</dependency>
```

将它添加到`pom.xml`中会将`MvcNamespaceHandler`添加到类路径中——允许项目使用名称空间配置 MVC 语义。

## 6。结论

最后，如果您使用 Eclipse 来管理 web 服务器和部署——确保正确配置了项目的部署组装部分(T1)——也就是说，在部署时，Maven 依赖项实际上包含在类路径中。

本教程讨论了“无法定位 XML schema 名称空间的 Spring NamespaceHandler”问题的常见疑点，并提供了每个问题的解决方案。