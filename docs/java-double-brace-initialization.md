# Java 双括号初始化

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-double-brace-initialization>

## 1。概述

在这个快速教程中，我们将展示如何使用双括号在单个 Java 表达式中创建和初始化对象。

我们还将了解为什么这种技术可以被视为反模式。

## 2。标准方法

通常，我们初始化并填充一组国家，如下所示:

```java
@Test
public void whenInitializeSetWithoutDoubleBraces_containsElements() {
    Set<String> countries = new HashSet<String>();                
    countries.add("India");
    countries.add("USSR");
    countries.add("USA");

    assertTrue(countries.contains("India"));
} 
```

从上面的例子可以看出，我们正在做以下事情:

1.  创建一个`HashSet`的实例
2.  将国家添加到`HashSet`
3.  最后，我们断言该国家是否出现在`HashSet`中

## 3。使用双拉条

然而，我们实际上可以将创建和初始化合并在一个语句中；这就是我们使用双括号的地方:

```java
@Test
public void whenInitializeSetWithDoubleBraces_containsElements() {
    Set<String> countries = new HashSet<String>() {
        {
           add("India");
           add("USSR");
           add("USA");
        }
    };

    assertTrue(countries.contains("India"));
} 
```

从上面的例子可以看出，我们是:

1.  创建一个扩展`HashSet`的匿名内部类
2.  提供一个实例初始化块，该块调用 add 方法并将国家名称添加到`HashSet`
3.  最后，我们可以断言该国家是否出现在`HashSet`中

## 4。使用双拉条的优点

使用双大括号有一些简单的优点:

*   与本机创建和初始化方式相比，代码行更少
*   代码可读性更好
*   创建初始化在同一个表达式中完成

## 5。使用双拉条的缺点

使用双大括号的缺点是:

*   模糊的、不广为人知的初始化方法
*   每次我们使用它时，它都会创建一个额外的类
*   不支持使用“钻石运算符”Java 7 中引入的一个特性
*   如果我们试图扩展的类被标记为`final`就不起作用
*   保存对封闭实例的隐藏引用，这可能会导致内存泄漏

由于这些缺点，双括号初始化被认为是一种反模式。

## 6。替代品

### 6.1。流工厂方法

相反，我们可以很好地利用新的 Java 8 Stream API 来初始化我们的`Set`:

```java
@Test
public void whenInitializeUnmodifiableSetWithDoubleBrace_containsElements() {
    Set<String> countries = Stream.of("India", "USSR", "USA")
      .collect(collectingAndThen(toSet(), Collections::unmodifiableSet));

    assertTrue(countries.contains("India"));
} 
```

### 6.2。Java 9 集合工厂方法

此外，Java 9 将带来一组有用的工厂方法，使以下事情成为可能:

```java
List<String> list = List.of("India", "USSR", "USA");
Set<String> set = Set.of("India", "USSR", "USA"); 
```

你可以在这篇文章中读到更多关于这个[的内容。](/web/20220810090259/https://www.baeldung.com/java-9-collections-factory-methods)

## 7 .**。结论**

在这篇简明教程中，我们讨论了双括号的用法及其优缺点。

这些例子的实现可以在 [GitHub 项目](https://web.archive.org/web/20220810090259/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-lang-syntax-2 "Double Brace usage examples on GitHub")中找到——这是一个基于 Maven 的项目，所以它应该很容易导入并按原样运行。