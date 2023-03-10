# 未定义名为“springSecurityFilterChain”的 Bean

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/no-bean-named-springsecurityfilterchain-is-defined>

## 1。问题

本文讨论了一个 Spring 安全配置问题——应用程序引导进程抛出以下异常:

```java
SEVERE: Exception starting filter springSecurityFilterChain
org.springframework.beans.factory.NoSuchBeanDefinitionException: 
No bean named 'springSecurityFilterChain' is defined
```

## 延伸阅读:

## [Spring Security Java 配置简介](/web/20221208143856/https://www.baeldung.com/java-config-spring-security)

A quick and practical guide to Java Config for Spring Security[Read more](/web/20221208143856/https://www.baeldung.com/java-config-spring-security) →

## [Spring Security 5–oauth 2 登录](/web/20221208143856/https://www.baeldung.com/spring-security-5-oauth2-login)

Learn how to authenticate users with Facebook, Google or other credentials using OAuth2 in Spring Security 5.[Read more](/web/20221208143856/https://www.baeldung.com/spring-security-5-oauth2-login) →

## [Servlet 3 异步支持 Spring MVC 和 Spring Security](/web/20221208143856/https://www.baeldung.com/spring-mvc-async-security)

Quick intro to the Spring Security support for async requests in Spring MVC.[Read more](/web/20221208143856/https://www.baeldung.com/spring-mvc-async-security) →

## 2。起因

这个异常的原因很简单——Spring Security 寻找一个名为`springSecurityFilterChain`的 bean(默认情况下),但是找不到它。这个 bean 是主**弹簧安全过滤器**—`DelegatingFilterProxy`—`web.xml`中定义的:

```java
<filter>
    <filter-name>springSecurityFilterChain</filter-name>
    <filter-class>org.springframework.web.filter.DelegatingFilterProxy</filter-class>
</filter>
<filter-mapping>
    <filter-name>springSecurityFilterChain</filter-name>
    <url-pattern>/*</url-pattern>
</filter-mapping>
```

这只是一个代理，它将其所有逻辑委托给`springSecurityFilterChain` bean。

## 3。解决方案

上下文中缺少此 bean 的最常见原因是安全 XML 配置没有定义**元素**:

```java
<?xml version="1.0" encoding="UTF-8"?>
<beans:beans  
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" 
  xmlns:beans="http://www.springframework.org/schema/beans"
  xmlns:sec="http://www.springframework.org/schema/security"
  xsi:schemaLocation="
    http://www.springframework.org/schema/security
    http://www.springframework.org/schema/security/spring-security-3.1.xsd
    http://www.springframework.org/schema/beans
    http://www.springframework.org/schema/beans/spring-beans-3.2.xsd">

</beans:beans>
```

如果 XML 配置使用安全名称空间——如上例所示，那么声明一个简单的< http >元素将确保过滤器 bean 被创建并且一切正常启动:

```java
<http auto-config='true'>
    <intercept-url pattern="/**" access="ROLE_USER" />
</http>
```

另一个可能的原因是**安全配置根本没有导入**到 web 应用程序的整个上下文中。

如果安全 XML 配置文件被命名为`springSecurityConfig.xml`，确保**资源被导入**:

```java
@ImportResource({"classpath:springSecurityConfig.xml"})
```

或者在 XML 中:

```java
<import resource="classpath:springSecurityConfig.xml" />
```

最后，过滤器 bean 的默认名称可以在`web.xml`中更改——通常是使用一个具有 Spring 安全性的现有过滤器:

```java
<filter>
    <filter-name>springSecurityFilterChain</filter-name>
    <filter-class>
      org.springframework.web.filter.DelegatingFilterProxy
    </filter-class>
    <init-param>
        <param-name>targetBeanName</param-name>
        <param-value>customFilter</param-value>
    </init-param>
</filter>
```

## 4。结论

本文讨论了一个非常具体的 Spring 安全问题——缺少过滤器链 bean——并展示了这个常见问题的解决方案。