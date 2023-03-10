# 寻找春天的版本

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-find-version>

## 1。概述

在本教程中，我们将演示如何以编程方式找出我们的应用程序正在使用的 Spring、JDK 和 Java 的版本。

## 2。如何获得 Spring 版本

我们将从学习如何获得我们的应用程序正在使用的 Spring 版本开始。

为了做到这一点，**我们将使用 [`SpringVersion`](https://web.archive.org/web/20220627171002/https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/core/SpringVersion.html) 类的**方法:

```java
assertEquals("5.1.10.RELEASE", SpringVersion.getVersion());
```

## 3。获得 JDK 版本

接下来，我们将得到我们目前在项目中使用的 JDK 版本。需要注意的是 [Java 和 JDK 不是同一个东西](/web/20220627171002/https://www.baeldung.com/jvm-vs-jre-vs-jdk)，所以它们会有不同的版本号。

如果我们使用的是 Spring 4.x，有一个名为 [`JdkVersion`](https://web.archive.org/web/20220627171002/https://docs.spring.io/spring/docs/4.2.4.RELEASE/javadoc-api/org/springframework/core/JdkVersion.html) 的类，我们可以用它来获取这些信息。然而，这个类已经从 Spring 5.x 中删除了，所以我们必须考虑到这一点并解决它。

在内部，Spring 4.x `JdkVersion `类从 [`SystemProperties`](https://web.archive.org/web/20220627171002/https://docs.spring.io/spring-boot/docs/current/api/org/springframework/boot/system/SystemProperties.html) 类获取版本，所以我们可以做同样的事情。利用类`SystemProperties,` 我们将访问属性`java.version` :

```java
assertEquals("1.8.0_191", SystemProperties.get("java.version"));
```

或者，我们可以不使用 Spring 类直接访问属性:

```java
assertEquals("1.8.0_191", System.getProperty("java.version"));
```

## 4。获取 Java 版本

最后，我们将看到如何获得我们的应用程序运行的 Java 版本。为此，**我们将使用 [`JavaVersion`](https://web.archive.org/web/20220627171002/https://docs.spring.io/spring-boot/docs/current/api/org/springframework/boot/system/JavaVersion.html)** :

```java
assertEquals("1.8", JavaVersion.getJavaVersion().toString());
```

以上，我们称之为`JavaVersion#getJavaVersion` 方法。默认情况下，这会返回一个带有特定 Java 版本的 enum，比如 [`EIGHT`](https://web.archive.org/web/20220627171002/https://docs.spring.io/spring-boot/docs/current/api/org/springframework/boot/system/JavaVersion.html#EIGHT) 。为了保持格式与上述方法一致，我们使用它的`toString `方法解析它。

## 5。结论

在本文中，我们了解到获取我们的应用程序正在使用的 Spring、JDK 和 Java 的版本非常简单。

和往常一样，完整的代码可以在 GitHub 上找到[。](https://web.archive.org/web/20220627171002/https://github.com/eugenp/tutorials/tree/master/spring-core-5)