# Spring 安全过滤器链中的自定义过滤器

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-security-custom-filter>

## 1。概述

在这个快速教程中，我们将重点关注为 Spring 安全过滤器链编写一个自定义过滤器。

## 延伸阅读:

## [Spring Security—@ pre filter 和@PostFilter](/web/20221208143815/https://www.baeldung.com/spring-security-prefilter-postfilter)

Learn how to use the @PreFilter and @PostFilter Spring Security annotations through practical examples.[Read more](/web/20221208143815/https://www.baeldung.com/spring-security-prefilter-postfilter) →

## [Spring Security Java 配置简介](/web/20221208143815/https://www.baeldung.com/java-config-spring-security)

A quick and practical guide to Java Config for Spring Security[Read more](/web/20221208143815/https://www.baeldung.com/java-config-spring-security) →

## [Spring Boot 安全自动配置](/web/20221208143815/https://www.baeldung.com/spring-boot-security-autoconfiguration)

A quick and practical guide to Spring Boot's default Spring Security configuration.[Read more](/web/20221208143815/https://www.baeldung.com/spring-boot-security-autoconfiguration) →

## 2。创建过滤器

默认情况下，Spring Security 提供了许多过滤器，大多数时候这些就足够了。

**当然，有时有必要通过创建新的过滤器来实现新的功能，以便在链中使用。**

我们将从实现`[org.springframework.web.filter.GenericFilterBean](https://web.archive.org/web/20221208143815/https://docs.spring.io/spring/docs/current/javadoc-api/org/springframework/web/filter/GenericFilterBean.html)`开始。

`GenericFilterBean`是一个简单的 [`javax.servlet.Filter`](https://web.archive.org/web/20221208143815/https://docs.oracle.com/javaee/7/api/javax/servlet/Filter.html) 实现，支持 Spring。

我们只需要实现一个方法:

```java
public class CustomFilter extends GenericFilterBean {

    @Override
    public void doFilter(
      ServletRequest request, 
      ServletResponse response,
      FilterChain chain) throws IOException, ServletException {
        chain.doFilter(request, response);
    }
}
```

## 3。使用安全配置中的过滤器

我们可以自由选择 XML 配置或 Java 配置来将过滤器连接到 Spring 安全配置中。

### 3.1。Java 配置

我们可以通过创建一个 [`SecurityFilterChain`](https://web.archive.org/web/20221208143815/https://docs.spring.io/spring-security/site/docs/current/api/org/springframework/security/web/SecurityFilterChain.html) bean 以编程方式注册过滤器。

例如，它与 [`HttpSecurity`](https://web.archive.org/web/20221208143815/https://docs.spring.io/spring-security/site/docs/5.0.0.M5/api/org/springframework/security/config/annotation/web/builders/HttpSecurity.html) 实例上的`addFilterAfter`方法一起工作:

```java
@Configuration
public class CustomWebSecurityConfigurerAdapter {

    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        http.addFilterAfter(
          new CustomFilter(), BasicAuthenticationFilter.class);
        return http.build();
    }
} 
```

有几种可能的方法:

*   `[addFilterBefore(filter, class)](https://web.archive.org/web/20221208143815/https://docs.spring.io/spring-security/site/docs/5.0.0.M5/api/org/springframework/security/config/annotation/web/builders/HttpSecurity.html#addFilterBefore(javax.servlet.Filter,%20java.lang.Class))` 在指定滤镜`class`的位置前增加一个 `filter` 。
*   [`addFilterAfter(filter, class)`](https://web.archive.org/web/20221208143815/https://docs.spring.io/spring-security/site/docs/5.0.0.M5/api/org/springframework/security/config/annotation/web/builders/HttpSecurity.html#addFilterAfter(javax.servlet.Filter,%20java.lang.Class)) 在指定滤镜`class`的位置后增加一个 `filter` 。
*   [`addFilterAt(filter, class)`](https://web.archive.org/web/20221208143815/https://docs.spring.io/spring-security/site/docs/5.0.0.M5/api/org/springframework/security/config/annotation/web/builders/HttpSecurity.html) 在指定过滤类的位置增加一个`filter`。
*   [`addFilter(filter)`](https://web.archive.org/web/20221208143815/https://docs.spring.io/spring-security/site/docs/5.0.0.M5/api/org/springframework/security/config/annotation/web/builders/HttpSecurity.html#addFilter(javax.servlet.Filter)) 添加一个`filter`，它必须是 Spring Security 提供的过滤器之一的实例或扩展。

### 3.2。XML 配置

我们可以使用标签`custom-filter`和这些名字[中的一个](https://web.archive.org/web/20221208143815/https://docs.spring.io/spring-security/site/docs/4.2.13.RELEASE/reference/htmlsingle/#ns-custom-filters)来指定过滤器的位置，从而将过滤器添加到链中。

例如，它可以由`after`属性指出:

```java
<http>
    <custom-filter after="BASIC_AUTH_FILTER" ref="myFilter" />
</http>

<beans:bean id="myFilter" class="com.baeldung.security.filter.CustomFilter"/>
```

以下是所有属性，用于指定过滤器在堆栈中的确切位置:

*   `after`描述过滤器，在此之后，一个定制过滤器将被放置在链中。
*   `before`定义我们的过滤器应该放在链中的哪个过滤器之前。
*   `position`允许在显式位置用自定义过滤器替换标准过滤器。

## 4。结论

在这篇简短的文章中，我们创建了一个定制过滤器，并将其连接到 Spring 安全过滤器链中。

与往常一样，所有代码示例都可以在示例 GitHub 项目中找到。