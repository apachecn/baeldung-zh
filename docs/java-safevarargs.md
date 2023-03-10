# Java @SafeVarargs 注释

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-safevarargs>

## 1.概观

在这个快速教程中，我们将看看`@SafeVarargs`注释。

## 2。`@SafeVarargs`注解

Java 5 引入了 varargs 的概念，即可变长度的方法参数，以及参数化类型。

将这些结合起来会给我们带来问题:

```java
public static <T> T[] unsafe(T... elements) {
    return elements; // unsafe! don't ever return a parameterized varargs array
}

public static <T> T[] broken(T seed) {
    T[] plant = unsafe(seed, seed, seed); // broken! This will be an Object[] no matter what T is
    return plant;
}

public static void plant() {
   String[] plants = broken("seed"); // ClassCastException
}
```

这些问题对于编译器来说很难确认，所以每当两者结合时它都会给出警告，就像在`unsafe:`的情况下一样

```java
warning: [unchecked] Possible heap pollution from parameterized vararg type T
  public static <T> T[] unsafe(T... elements) {
```

这个方法如果使用不当，就像在`broken`、**的情况下，会将一个`Object[]`数组污染到堆中，而不是预期的类型`b`、**。

为了消除这个警告，我们可以在 final 或 static 方法和构造函数上添加`@SafeVarargs`注释**。**

`@SafeVarargs`类似于`@SupressWarnings`,因为它允许我们声明一个特定的编译器警告是误报。**一旦我们确保我们的行动是安全的**，我们就可以添加这个注释:

```java
public class Machine<T> {
    private List<T> versions = new ArrayList<>();

    @SafeVarargs
    public final void safe(T... toAdd) {
        for (T version : toAdd) {
            versions.add(version);
        }
    }
}
```

varargs 的安全使用本身就是一个棘手的概念。要了解更多信息，Josh Bloch 在他的书《有效的 Java》中有一个很好的解释。

## 3.结论

在这篇简短的文章中，我们看到了如何在 Java 中使用`@SafeVarargs`注释。

示例的完整源代码可以在 GitHub 的[中找到。](https://web.archive.org/web/20220627084953/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-annotations)