# 在 Java 中将 Double 转换为 Long

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-convert-double-long>

## 1.概观

在本教程中，我们将探索在 Java 中从`double`转换到`long`的各种方法。

## 2.使用类型转换

让我们检查一种使用 cast 操作符将`double`转换为`long`的简单方法:

```
Assert.assertEquals(9999, (long) 9999.999);
```

对值 9999.999 的`double`应用`(long)` cast 运算符会得到 9999。

这是一个[缩小原始转换](/web/20221207130348/https://www.baeldung.com/java-primitive-conversions)，因为我们正在失去精度。当一个`double`被强制转换为一个`long`时，结果将保持不变，不包括小数点。

## 3.使用`Double.longValue`

现在，让我们探索一下将`double`转换成`long`的`Double's`内置方法`longValue`:

```
Assert.assertEquals(9999, Double.valueOf(9999.999).longValue());
```

正如我们所见，对值 9999.999 的`double`应用`longValue`方法得到 9999。在内部，**`longValue`方法正在执行一个简单的强制转换**。

## 4.使用`Math`方法

最后，让我们看看如何使用 [`Math`](/web/20221207130348/https://www.baeldung.com/java-lang-math) 类中的`round, ceil, and floor`方法将`double`转换为`long`:

让我们首先检查`Math.round.` 这产生了一个最接近参数的值:

```
Assert.assertEquals(9999, Math.round(9999.0));
Assert.assertEquals(9999, Math.round(9999.444));
Assert.assertEquals(10000, Math.round(9999.999));
```

其次，`Math.` `ceil`将产生大于或等于自变量的最小值:

```
Assert.assertEquals(9999, Math.ceil(9999.0), 0);
Assert.assertEquals(10000, Math.ceil(9999.444), 0);
Assert.assertEquals(10000, Math.ceil(9999.999), 0);
```

另一方面，`Math.floor`与`Math.ceil.` 正好相反，它返回小于或等于参数的最大值:

```
Assert.assertEquals(9999, Math.floor(9999.0), 0);
Assert.assertEquals(9999, Math.floor(9999.444), 0);
Assert.assertEquals(9999, Math.floor(9999.999), 0);
```

注意，`Math.ceil`和`Math.round`都返回一个`double`值，但是在这两种情况下，返回的值都相当于一个`long`值。

## 5.结论

在本文中，我们讨论了在 Java 中将`double`转换为`long`的各种方法。在将每个方法应用到任务关键型代码之前，最好先了解每个方法的行为。

本教程的完整源代码可以从 GitHub 上的[处获得。](https://web.archive.org/web/20221207130348/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-numbers-3)