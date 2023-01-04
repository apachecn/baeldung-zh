# Paths.get 和 Path.of 之间的区别

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-paths-get-path-of>

## 1.概观

在本文中，我们将评论方法 [`Paths.get()`](/web/20221123160409/https://www.baeldung.com/java-nio-2-path#creating-a-path) 和`Path.of()`之间的异同。

## 2.相同的行为

`Path.of()`方法将一个`[URI](/web/20221123160409/https://www.baeldung.com/java-url-vs-uri)`作为参数，并将其转换为关联对象的 [`Path`](/web/20221123160409/https://www.baeldung.com/java-path-vs-file#javaniofilepath-class) 。

现在让我们来看看`Paths.get()`的代码:

```
public final class Paths {
    public static Path get(URI uri) {
        return Path.of(uri);
    }
}
```

**正如我们所见，`Paths.get()`唯一做的事情就是调用`Path.of().`,因此，这两个方法返回相同的结果。**

## 3.两种方法之间的差异

我们现在将评论这两种方法之间的区别。

### 3.1.介绍版本

**在 Java 8 之前，不可能在一个接口内定义一个[默认静态方法](/web/20221123160409/https://www.baeldung.com/java-static-default-methods)。**因此`Path`需要一个配套接口`Paths`。此时，所有的[工厂方法](/web/20221123160409/https://www.baeldung.com/creational-design-patterns#factory-method)都在路径中定义。

然后这个限制被去掉了，在 Java 11 中，工厂方法的代码最终被移到了`Path`接口。另外，`Paths.get()`的代码更新为遵从`Path.of()`。`Paths.get()`确实被保留了，以保证向后兼容。

### 3.2.命名模式

不仅代码被移动了，而且工厂方法的名称也改变了。原名的问题是看起来像 getter。然而，`Paths.get()`并没有得到任何属于`Paths`对象的东西。**Java 中静态工厂方法的标准名称是`of`。例如，`[EnumSet.of()](/web/20221123160409/https://www.baeldung.com/java-enumset)`就遵循这种模式。因此，为了保持一致性，新方法被称为`Path.of()`。**

## 4.我们应该使用哪一个？

如果我们正在使用 Java 7 和 10 之间的版本，我们别无选择，只能使用`Paths.get()`。**否则，如果我们使用更高版本，我们应该选择`Path.of()`。正如在类的注释中所说的，`Paths`类在未来的 Java 版本中可能会被弃用。此外，直接使用来自`Path`的工厂方法节省了额外的输入。**

## 5.结论

在本教程中，我们已经了解到两个相同的方法，`Paths.get()`和`Path.of(),`由于某些历史原因而共存。我们分析了它们的差异，并根据我们的情况得出了最适合我们的结论。