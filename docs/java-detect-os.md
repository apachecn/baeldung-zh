# 如何使用 Java 检测操作系统

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-detect-os>

## 1。简介

有几种方法可以确定我们的代码在哪个操作系统上运行。

在这篇简短的文章中，我们将了解如何在 Java 中进行操作系统检测。

## 2。实施

一种方法是利用`System`。`getProperty(os.name)`获取操作系统的名称。

第二种方法是利用来自`Apache Commons Lang` API 的`SystemUtils`。

让我们来看看他们两个的行动。

### 2.1。使用系统属性

我们可以利用`System` 类来检测操作系统。

让我们来看看:

```java
public String getOperatingSystem() {
    String os = System.getProperty("os.name");
    // System.out.println("Using System Property: " + os);
    return os;
}
```

### 2.2。`SystemUtils`——Apache common lang

来自 Apache Commons 的 Lang 是另一个值得尝试的热门选项。这是一个很好的 API，很好地处理了这些细节。

让我们用`SystemUtils:`找出操作系统

```java
public String getOperatingSystemSystemUtils() {
    String os = SystemUtils.OS_NAME;
    // System.out.println("Using SystemUtils: " + os);
    return os;
}
```

## 3。结果

在我们的环境中执行代码会得到相同的结果:

```java
Using SystemUtils: Windows 10
Using System Property: Windows 10
```

## 4。结论

在这篇简短的文章中，我们看到了如何从 Java 中以编程方式找到/检测操作系统。

和往常一样，本文的代码示例可以在 GitHub 的[上找到。](https://web.archive.org/web/20221030103547/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-os)