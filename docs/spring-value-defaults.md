# 使用带有默认值的 Spring @Value

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-value-defaults>

## 1。概述

Spring 的`@Value`注释提供了一种将属性值注入组件的便捷方式。对于属性可能不存在的情况，**提供合理的缺省值也非常有用。**

这就是我们在本教程中要关注的——如何为`@Value` Spring 注释指定默认值。

关于`@Value`的更详细的快速指南，请在这里查看文章[。](/web/20221001115718/https://www.baeldung.com/spring-value-annotation)

## 延伸阅读:

## [Spring @ Value 快速指南](/web/20221001115718/https://www.baeldung.com/spring-value-annotation)

Learn to use the Spring @Value annotation to configure fields from property files, system properties, etc.[Read more](/web/20221001115718/https://www.baeldung.com/spring-value-annotation) →

## [弹簧和 Spring Boot 的特性](/web/20221001115718/https://www.baeldung.com/properties-with-spring)

Tutorial for how to work with properties files and property values in Spring.[Read more](/web/20221001115718/https://www.baeldung.com/properties-with-spring) →

## [春天表情语言指南](/web/20221001115718/https://www.baeldung.com/spring-expression-language)

This article explores Spring Expression Language (SpEL), a powerful expression language that supports querying and manipulating object graphs at runtime.[Read more](/web/20221001115718/https://www.baeldung.com/spring-expression-language) →

## 2。字符串默认值

让我们看看为`String`属性设置默认值的基本语法:

```java
@Value("${some.key:my default value}")
private String stringWithDefaultValue; 
```

如果`some.key` 无法解析，`stringW` *ithDefaultValue* 将被设置为`my default value`的默认值。

类似地，我们可以将零长度的`String`设置为默认值:

```java
@Value("${some.key:})"
private String stringWithBlankDefaultValue;
```

## 3。原语

为了给像`boolean`和`int`这样的原始类型设置默认值，我们使用文字值:

```java
@Value("${some.key:true}")
private boolean booleanWithDefaultValue;
```

```java
@Value("${some.key:42}")
private int intWithDefaultValue; 
```

如果我们愿意，我们可以通过将类型改为`Boolean`和`Integer`来使用原始包装器。

## 4。数组

我们还可以将逗号分隔的值列表注入数组:

```java
@Value("${some.key:one,two,three}")
private String[] stringArrayWithDefaults;

@Value("${some.key:1,2,3}")
private int[] intArrayWithDefaults;
```

在上面的第一个例子中，值`one`、`two`和`three` 作为默认值被注入到`stringArrayWithDefaults`中。

在第二个例子中，值`1`、 `2`和`3` 作为默认值被注入到`intArrayWithDefaults`中。

## 5。使用 SpEL

我们还可以使用 Spring Expression Language (SpEL)来指定表达式和默认值。

在下面的例子中，我们希望将`some.system.key` 设置为系统属性，如果没有设置，我们希望使用`my default system property value`作为默认值:

```java
@Value("#{systemProperties['some.key'] ?: 'my default system property value'}")
private String spelWithDefaultValue;
```

## 6。结论

在这篇简短的文章中，我们看了如何为一个属性设置默认值，我们希望使用 Spring 的`@Value`注释注入这个属性的值。

像往常一样，本文中使用的所有代码示例都可以在 GitHub 项目的[中找到。](https://web.archive.org/web/20221001115718/https://github.com/eugenp/tutorials/tree/master/spring-boot-modules/spring-boot-properties-2)