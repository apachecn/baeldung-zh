# SpringBoot 中的字符编码过滤器

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-boot-characterencodingfilter>

## 1.概观

在本文中，我们将了解`CharacterEncodingFilter`以及它在 [Spring Boot](/web/20221128045702/https://www.baeldung.com/spring-boot) 应用程序中的用法。

## 2.`CharacterEncodingFilter`

[`CharacterEncodingFilter`](https://web.archive.org/web/20221128045702/https://docs.spring.io/spring/docs/current/javadoc-api/org/springframework/web/filter/CharacterEncodingFilter.html) **是一个 servlet 过滤器，帮助我们** **为请求和响应**指定字符编码。当浏览器没有设置字符编码时，或者如果我们想要请求和响应的特定解释时，这个过滤器是有用的。

## 3.履行

让我们看看如何在 Spring Boot 应用程序中配置这个过滤器。

首先，让我们创建一个`CharacterEncodingFilter:`

```
CharacterEncodingFilter filter = new CharacterEncodingFilter();
filter.setEncoding("UTF-8");
filter.setForceEncoding(true);
```

在我们的例子中，我们将编码设置为 UTF-8。但是，我们可以根据需要设置任何其他编码。

我们还使用了`**forceEncoding**` **属性来执行编码**，而不管它是否出现在浏览器的请求中。由于该标志被设置为`true,`，所提供的编码也将被用作响应编码。

最后，**我们将** **用`FilterRegistrationBean`** 注册过滤器，这提供了将`Filter`实例注册为过滤器链的一部分的配置:

```
FilterRegistrationBean registrationBean = new FilterRegistrationBean();
registrationBean.setFilter(filter);
registrationBean.addUrlPatterns("/*");
return registrationBean;
```

在非 spring boot 应用程序中，我们可以在 [web.xml](/web/20221128045702/https://www.baeldung.com/spring-xml-vs-java-config) 文件中添加这个过滤器来获得相同的效果。

## 4.结论

在本文中，我们描述了对`CharacterEncodingFilter`的需求，并看到了它的配置示例。

和往常一样，本文的完整代码可以在 GitHub 上的[处获得。](https://web.archive.org/web/20221128045702/https://github.com/eugenp/tutorials/tree/master/spring-boot-modules/spring-boot-mvc-3)