# 空参数的 IllegalArgumentException 或 NullPointerException？

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-illegalargumentexception-or-nullpointerexception>

## 1.介绍

在我们编写应用程序时所做的决定中，很多都是关于何时抛出[异常](/web/20220906080534/https://www.baeldung.com/java-exceptions)以及抛出哪种类型的异常。

在这个快速教程中，我们将解决当有人将一个`null`参数传递给我们的方法之一时抛出哪个异常的问题:`IllegalArgumentException`或 [`NullPointerException`](/web/20220906080534/https://www.baeldung.com/java-14-nullpointerexception) 。

我们将通过分析双方的观点来探讨这个话题。

## 2.`IllegalArgumentException`

首先，让我们看看抛出一个`IllegalArgumentException`的论点。

让我们创建一个简单的方法，当传递一个`null`时抛出一个`IllegalArgumentException`:

```java
public void processSomethingNotNull(Object myParameter) {
    if (myParameter == null) {
        throw new IllegalArgumentException("Parameter 'myParameter' cannot be null");
    }
}
```

现在，让我们转向支持`IllegalArgumentException`的论点。

### 2.1.Javadoc 是这样说使用它的

当我们阅读`IllegalArgumentException` 的 [Javadoc 时，它说**是在非法或不适当的值被传递给方法**时使用的。如果我们的方法不期望一个`null`对象，我们可以认为它是非法或不合适的，这将是我们抛出的一个合适的异常。](https://web.archive.org/web/20220906080534/https://docs.oracle.com/en/java/javase/14/docs/api/java.base/java/lang/IllegalArgumentException.html)

### 2.2.它符合开发者的期望

接下来，让我们思考一下，作为开发人员，当我们在应用程序中看到堆栈跟踪时，我们是如何思考的。我们收到一个`NullPointerException`的一个非常常见的场景是当我们不小心试图访问一个`null`对象时。在这种情况下，我们将尽可能深入堆栈，看看我们引用的是什么`null`。

当我们得到一个`IllegalArgumentException`时，我们很可能会认为我们正在传递一些错误的东西给一个方法。在这种情况下，我们将在堆栈中查找我们调用的最底层的方法，并从那里开始我们的调试。如果我们考虑这种思维方式，那么`IllegalArgumentException`将会让我们更接近错误发生的地方。

### 2.3.其他论点

在我们开始讨论`NullPointerException`之前，让我们先来看几个支持`IllegalArgumentException`的小观点。一些开发商认为，只有 JDK 应该抛出`NullPointerException`。正如我们将在下一节看到的，Javadoc 不支持这种理论。另一个论点是使用`IllegalArgumentException`更一致，因为这是我们对其他非法参数值使用的。

## 3.`NullPointerException`

接下来，让我们考虑一下`NullPointerException`的论点。

让我们创建一个抛出`NullPointerException`的例子:

```java
public void processSomethingElseNotNull(Object myParameter) {
    if (myParameter == null) {
        throw new NullPointerException("Parameter 'myParameter' cannot be null");
    }
}
```

### 3.1.Javadoc 是这样说使用它的

根据`NullPointerException` 的 [Javadoc， **`NullPointerException`的意思是试图在需要**对象的地方使用`null`。如果我们的方法参数不打算是`null`，那么我们可以合理地认为这是一个被请求的对象，并抛出`NullPointerException`。](https://web.archive.org/web/20220906080534/https://docs.oracle.com/en/java/javase/14/docs/api/java.base/java/lang/NullPointerException.html)

### 3.2.这和 JDK 原料药是一致的

让我们花一点时间来思考我们在开发过程中调用的许多常见的 JDK 方法。如果我们提供一个`null`，他们中的许多人会抛出一个`NullPointerException`。另外，`Objects.requireNonNull()` 抛出一个`NullPointerException `，如果我们根据 [`Objects`文档传入`null.`，](https://web.archive.org/web/20220906080534/https://docs.oracle.com/en/java/javase/14/docs/api/java.base/java/util/Objects.html#requireNonNull(T))它的存在主要是为了验证参数。

除了抛出`NullPointerException`的 JDK 方法之外，我们还可以找到从集合 API 的方法中抛出特定异常类型的其他例子。如果索引超出列表大小，则 **`ArrayList.addAll(index, Collection)`抛出一个`IndexOutOfBoundsException`，如果集合是`null`** ，则抛出一个`NullPointerException`。这是两种非常特殊的异常类型，而不是更一般的`IllegalArgumentException`。

我们可以认为`IllegalArgumentException`是针对我们没有更具体的异常类型的情况。

## 4.结论

正如我们在本教程中看到的，这是一个没有明确答案的问题。这两个异常的文档似乎有所重叠，当单独使用时，它们听起来都很合适。基于开发人员如何进行调试以及在 JDK 方法本身中看到的模式，双方都有额外的令人信服的论据。

无论我们选择哪个异常，我们都应该在整个应用程序中保持一致。此外，通过向异常构造函数提供有意义的信息，我们可以使我们的异常更加有用。例如，如果我们在异常消息中提供参数名称，我们的应用程序将更容易调试。

与往常一样，GitHub 上的示例代码[可用。](https://web.archive.org/web/20220906080534/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-exceptions-3)