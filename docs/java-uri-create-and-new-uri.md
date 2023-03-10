# URI.create()和新 URI()之间的区别

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-uri-create-and-new-uri>

## 1.概观

当我们试图访问网络中的一些资源时，我们必须首先获得资源的统一资源标识符(URI)。

Java 标准库提供了 [`URI`](/web/20221208143917/https://www.baeldung.com/java-url-vs-uri) 类，让我们更容易处理 URIs。当然，要使用`URI`类，第一步是获得一个`URI`实例。

假设我们有网络中某个资源的地址串。有两种方法可以获得`URI`实例:

*   用地址字符串直接调用构造函数-`URI myUri = new URI(theAddress);`
*   调用`URI.create()` `static`方法—`URI myUri = URI.create(theAddress);`

在这个快速教程中，我们将仔细研究这两种方法，并讨论它们的区别。

## 2.使用`URI`构造函数

首先，让我们看看构造函数的签名:

```java
public URI(String str) throws URISyntaxException
```

我们可以看到，用无效地址调用构造函数可能会引发`URISyntaxException`。为了简单起见，让我们创建一个单元测试来看看这个案例:

```java
assertThrows(URISyntaxException.class, () -> new URI("I am an invalid URI string.")); 
```

我们要注意的是， **`URISyntaxException`是** `**Exception**`的子类:

```java
public class URISyntaxException extends Exception { ... } 
```

因此，**是一个[被检查的异常](/web/20221208143917/https://www.baeldung.com/java-checked-unchecked-exceptions)T4。换句话说，**当我们调用这个构造函数时，我们必须处理`URISyntaxException`** :**

```java
try {
    URI myUri = new URI("https://www.baeldung.com/articles");
    assertNotNull(myUri);
} catch (URISyntaxException e) {
    fail();
}
```

我们已经看到了作为单元测试的构造函数用法和异常处理。如果我们执行测试，它就通过了。**然而，没有`try-catch`** 代码就无法编译。

正如我们所见，使用构造函数创建一个`URI`实例非常简单。接下来，让我们转到`URI.create()`方法。

## 3.使用`URI.create()`方法

我们已经提到过`URI.create()`也可以创建一个`URI`实例。为了理解`create()`方法和构造函数之间的区别，让我们看一下`create()`方法的源代码:

```java
public static URI create(String str) {
    try {
        return new URI(str);
    } catch (URISyntaxException x) {
        throw new IllegalArgumentException(x.getMessage(), x);
    }
}
```

正如我们在上面的代码中看到的，`create()`方法的实现非常简单。首先，它直接调用构造函数。此外，如果`URISyntaxException`被抛出，**将异常包装在一个新的`IllegalArgumentException. `** 中

接下来，让我们创建一个测试，看看如果我们给`create()`一个无效的 URI 字符串，我们是否能得到预期的`IllegalArgumentException`:

```java
assertThrows(IllegalArgumentException.class, () -> URI.create("I am an invalid URI string."));
```

如果我们试一试，测试就会通过。

如果我们仔细看看`IllegalArgumentException`类，我们可以看到它是`RuntimeException`的子类:

```java
public class IllegalArgumentException extends RuntimeException { ... }
```

我们知道`RuntimeException`是一个未检查的异常。这意味着**当我们使用`create()`方法创建一个`URI`实例时，我们不必在显式的`try-catch`** 中处理异常:

```java
URI myUri = URI.create("https://www.baeldung.com/articles");
assertNotNull(myUri);
```

因此，`create()`和`URI()`构造函数的主要区别在于，`create()`方法将构造函数的检查异常(`URISyntaxException`)变为未检查异常(`IllegalArgumentException`)。

**如果我们不想处理被检查的异常，我们可以使用`create()` `static`方法创建 URI 实例**。

## 5.结论

在本文中，我们讨论了使用构造函数和`URI.create()`方法实例化一个`URI`对象的区别。

像往常一样，这里展示的所有代码片段都可以在 GitHub 上获得。