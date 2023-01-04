# 警告:“不推荐使用 WebMvcConfigurerAdapter 类型”

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/web-mvc-configurer-adapter-deprecated>

## 1.介绍

在这个快速教程中，我们将看看在使用 Spring 5.x.x 版本时可能会看到的一个警告，即引用不赞成使用的`WebMvcConfigurerAdapter`类的警告。

我们将了解为什么会出现这个警告以及如何处理它。

## 2。警告出现的原因

如果我们正在使用 Spring version 5(或 Spring Boot 2) ，这个警告**就会出现，无论是升级现有的应用程序还是用旧的 API 构建新的应用程序。**

让我们简单地回顾一下它背后的历史。

在 Spring 的早期版本中，包括版本 4，如果我们想要配置一个 web 应用程序，我们可以使用`WebMvcConfigurerAdapter`类:

```java
@Configuration
public WebConfig extends WebMvcConfigurerAdapter {

    // ...
}
```

这是一个抽象类，实现了`WebMvcConfigurer`接口，并包含所有继承方法的空实现。

通过对它进行子类化，我们可以覆盖它的方法，这些方法提供了到各种 MVC 配置元素的挂钩，比如视图解析器、拦截器等等。

但是，Java 8 在接口中增加了默认方法的概念。自然地，Spring 团队更新了框架以充分利用新的 Java 语言特性。

## 3.解决办法

如前所述，从 Spring 5 开始，`WebMvcConfigurer`接口包含其所有方法的默认实现。因此，抽象适配器类被标记为不推荐使用。

让我们看看**我们如何开始直接使用界面，并摆脱警告**:

```java
@Configuration
public WebConfig implements WebMvcConfigurer {
    // ...
}
```

仅此而已！这种改变应该很容易实现。

如果有任何对被覆盖方法的`super()`调用，我们也应该移除它们。否则，我们可以像往常一样覆盖任何配置回调。

虽然删除警告不是强制性的，但建议这样做，因为新的 API 更方便，并且在未来的版本中可能会删除不推荐使用的类。

## 4.结论

在这篇简短的文章中，我们看到了如何修复关于不推荐使用`WebMvcConfigurerAdapter`类的警告。