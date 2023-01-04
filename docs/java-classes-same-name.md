# 在 Java 中处理同名的类

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-classes-same-name>

## 1.介绍

与主流编程语言一样，Java 中的类命名遵循一种被称为大写字母语法的国际惯例。然而，当涉及到处理同名的类时，就有一个挑战。

自从 1998 年《JDK》的早期发行以来，人们一直在争论如何解决这种不寻常的情况。这里是 [JDK-4194542](https://web.archive.org/web/20221101132734/https://bugs.java.com/bugdatabase/view_bug.do?bug_id=4194542 "JDK-4194542") ，关于这个话题的第一个公开的 bug，从那以后，JDK 开发团队的建议是使用完全限定的类名。然而，目前还没有计划让 JDK 在短期内具备这种功能。

最近，在 2019 年 8 月，Java 开发者社区提出了一个关于如何解决这种情况的新提议( [JEP](https://web.archive.org/web/20221101132734/https://gist.github.com/cardil/b29a81efd64a09585076fe00e3d34de7 "JEP") )，它正在获得全球 Java 开发者的更多支持。

在本教程中，我们将讨论处理同名类的策略和建议。

## 2.定义类别

首先，让我们在定制包`com.baeldung.date.`中创建一个名为`Date` 的类

```java
package com.baeldung.date;

public class Date {

    private long currentTimeMillis;

    public Date() {
        this(System.currentTimeMillis());
    }

    public Date(long currentTimeMillis) {
        this.currentTimeMillis = currentTimeMillis;
    }

    public long getTime() {
        return currentTimeMillis;
    }
}
```

## 3.完全限定的类名

当这种用法是孤立的并且不经常重复时，我们将使用这种方法来避免冲突。尽管如此，使用完全限定名通常被认为是一种糟糕的风格。

让我们来看看如何使用它，特别是如果包名很短并且是描述性的，可以使代码更有表现力，从而减少混乱并增加可读性。

另一方面，当内部使用的对象是太大的类或方法时，它有助于调试:

```java
public class DateUnitTest {

    @Test
    public void whenUsingFullyQualifiedClassNames() {

        java.util.Date javaDate = new java.util.Date();
        com.baeldung.date.Date baeldungDate = new com.baeldung.date.Date(javaDate.getTime());

        Assert.assertEquals(javaDate.getTime(), baeldungDate.getTime());
    }
}
```

## 4.导入最常用的一个

**我们导入最常用的类路径，使用最少用的类路径，因为这是 Java 开发人员的常用技术和最佳实践:**

```java
import java.util.Date;

public class DateUnitTest {

    @Test
    public void whenImportTheMostUsedOne() {

        Date javaDate = new Date();
        com.baeldung.date.Date baeldungDate = new com.baeldung.date.Date(javaDate.getTime());

        Assert.assertEquals(javaDate.getTime(), baeldungDate.getTime());
    }
}
```

## 5.结论

在本文中，我们举例说明了根据特定情况使用同名类的两种可能方法，并观察了它们之间的主要区别。

和往常一样，本文的完整代码示例可以在 GitHub 的[中找到。](https://web.archive.org/web/20221101132734/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-lang-5)