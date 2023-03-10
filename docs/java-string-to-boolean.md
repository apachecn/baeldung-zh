# 将 Java 字符串转换为布尔值

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-string-to-boolean>

## 1.概观

在本教程中，我们将**探索使用 Java 的`Boolean`类将**转换成`boolean`的不同方法。

## 2.`Boolean.parseBoolean()`

`Boolean.parseBoolean()`允许我们传入一个`String`并接收一个原语`boolean`。

首先，让我们写一个测试来看看`parseBoolean()`如何用值 `true:`转换一个`String`

```java
assertThat(Boolean.parseBoolean("true")).isTrue();
```

当然，测试通过了。

事实上，`parseBoolean()`的语义非常清楚，IntelliJ IDEA 警告我们传递字符串文字`“true”`是多余的。

换句话说，**这个方法对于把一个`String`变成一个`boolean`非常有效。**

## 3.`Boolean.valueOf()`

`Boolean.valueOf()`也让我们传入一个`String`，但是这个方法返回一个`Boolean`类实例，而不是一个原语`boolean`。

我们可以看到这个方法也成功地转换了我们的`String:`

```java
assertThat(Boolean.valueOf("true")).isTrue();
```

这个方法实际上使用`parseBoolean()`在后台进行它的`String`转换，并简单地使用结果返回一个静态定义的`Boolean`实例。

因此，**只有在需要返回的`Boolean`实例时，才应该使用这个方法。**如果只需要一个原始结果，直接使用`parseBoolean()`会更有效。

## 4.`Boolean.getBoolean()`

`Boolean.getBoolean()`是第三种方法，它接受一个`String`并返回一个`boolean`。

如果不看文档或这个方法的实现，人们可能会合理地假设这个方法也用于将它的`String`参数转换成`boolean:`

```java
assertThat(Boolean.getBoolean("true")).isTrue(); // this test fails!
```

这个测试失败的原因是**`String`参数应该代表一个`boolean`系统属性的名称。**

通过定义系统属性:

```java
System.setProperty("CODING_IS_FUN", "true");
assertThat(Boolean.getBoolean("CODING_IS_FUN")).isTrue();
```

最后，测试通过。检查这个方法的实现可以发现，它也使用了`parseBoolean()`方法来完成它的`String`转换。

请注意，`getBoolean()`实际上是`parseBoolean(System.getProperty(“true”)),` 的快捷方式，意思是我们不应该被这个名字误导。

因此，**`Boolean.getBoolean(“true”);`**返回`true`的唯一方式是如果存在一个名为`“true”`的系统属性，并且它的值被解析为`true`。

## 4.结论

在这个简短的教程中，我们已经看到了`Boolean.parseBoolean()`、`Boolean.valueOf()`和`Boolean.getBoolean()`之间的主要区别。

**虽然`parseBoolean()`和`valueOf()`都可以将`String`转换成`boolean`，但重要的是要记住`Boolean.getBoolean()`不能。**

本教程中所有例子的源代码可以在 Github 的[中找到。](https://web.archive.org/web/20220525125439/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-lang-3)