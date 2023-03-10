# 使用 Java 断言

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-assert>

## 1。简介

Java `assert`关键字允许开发人员快速验证程序的某些假设或状态。

在本文中，**我们将看看如何使用 Java `assert`关键字。**

## 2。Java 断言的历史

Java `assert`关键字是在 Java 1.4 中引入的，所以它已经存在很长时间了。然而，它仍然是一个鲜为人知的关键字，可以极大地减少样板文件，使我们的代码更具可读性。

例如，在我们的代码中，我们经常需要验证某些可能阻止我们的应用程序正常工作的条件。通常我们会这样写:

```java
Connection conn = getConnection();
if(conn == null) {
    throw new RuntimeException("Connection is null");
}
```

使用断言，我们可以用一条`assert`语句删除`if`和`throw`语句。

## 3。启用 Java 断言

因为 Java 断言使用了`assert`关键字，所以不需要导入库或包。

注意，在 Java 1.4 之前，使用“assert”这个词来命名变量、方法等是完全合法的。当在较新的 JVM 版本中使用较旧的代码时，这可能会产生命名冲突。

因此，为了向后兼容，**JVM 默认禁用断言验证**。必须使用`-enableassertions`命令行参数或其简写`-ea:`显式启用它们

```java
java -ea com.baeldung.assertion.Assertion
```

在这个例子中，我们已经为所有的类启用了断言。

我们还可以为特定的包和类启用断言:

```java
java -ea:com.baeldung.assertion... com.baeldung.assertion.Assertion
```

这里，我们已经为`com.baeldung.assertion`包中的所有类启用了断言。

同样，对于特定的包和类，可以使用`-disableassertions`命令行参数或其简写`-da`来禁用它们。我们也可以一起使用这四个论点。

## 4。使用 Java 断言

要添加断言，**只需使用`assert`关键字并给它一个`boolean`条件**:

```java
public void setup() {
    Connection conn = getConnection();
    assert conn != null;
}
```

Java 还为接受字符串的断言提供了第二种语法，如果抛出一个字符串，它将用于构造`AssertionError`:

```java
public void setup() {
    Connection conn = getConnection();
    assert conn != null : "Connection is null";
}
```

在这两种情况下，代码都检查到外部资源的连接是否返回非空值。如果该值为`null,`，JVM 将**自动抛出一个`AssertionError`** 。

在第二种情况下，异常将具有额外的详细信息，这些信息将显示在堆栈跟踪中，有助于调试问题。

让我们看看启用断言后运行我们的类的结果:

```java
Exception in thread "main" java.lang.AssertionError: Connection is null
        at com.baeldung.assertion.Assertion.setup(Assertion.java:15)
        at com.baeldung.assertion.Assertion.main(Assertion.java:10)
```

## 5。处理一个`AssertionError`

类`AssertionError`扩展了`Error`，T1 本身又扩展了`Throwable`。这意味着`AssertionError`是一个未检查的异常。

因此，使用断言的方法不需要声明它们，并且进一步的调用代码不应该试图捕获它们。

**`AssertionErrors`表示应用程序中不可恢复的情况**，所以不要试图处理它们或试图恢复。

## 6。最佳实践

关于断言，要记住的最重要的事情是它们可以被禁用，所以**永远不要假设它们会被执行**。

因此，在使用断言时，请记住以下几点:

*   始终检查空值，并在适当的地方清空`Optionals`
*   避免使用断言来检查公共方法的输入，而是使用未检查的异常，如`IllegalArgumentException`或`NullPointerException`
*   不要在断言条件中调用方法，而是将方法的结果赋给一个局部变量，并用`assert`使用该变量
*   断言对于代码中永远不会被执行的地方非常有用，比如一个`switch`语句的`default`情况，或者在一个永远不会结束的循环之后

## 7 .**。结论**

Java `assert`关键字已经存在很多年了，但仍然是该语言的一个鲜为人知的特性。它可以帮助删除大量样板代码，使代码更具可读性，并有助于在程序开发的早期识别错误。

请记住，断言在默认情况下是不启用的，所以永远不要假设它们在代码中使用时会被执行。

和往常一样，完整的源代码可以在 GitHub 上找到[。](https://web.archive.org/web/20220924073308/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-lang)