# Java 中的抽象方法错误

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-abstractmethoderror>

## 1.概观

有时，我们可能会在应用程序运行时遇到`[AbstractMethodError](https://web.archive.org/web/20221208143917/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/AbstractMethodError.html)`。如果我们不清楚这个错误，可能需要一段时间来确定问题的原因。

在本教程中，我们将仔细看看`AbstractMethodError`。我们会明白`AbstractMethodError` 是什么，什么时候会发生。

## 2.`AbstractMethodError`简介

当应用程序试图调用未实现的抽象方法时，抛出 **`AbstractMethodError`。**

我们知道如果有未实现的抽象方法，编译器会先抱怨。因此，应用程序根本不会构建。

我们可能会问，我们如何在运行时得到这个错误？

首先，让我们看看`AbstractMethodError`在 Java 异常层次结构中的位置:

```java
java.lang.Object
|_java.lang.Throwable
  |_java.lang.Error
    |_java.lang.LinkageError
      |_java.lang.IncompatibleClassChangeError
        |_java.lang.AbstractMethodError
```

正如上面的层次结构所示，这个错误是`IncompatibleClassChangeError`的一个子类。顾名思义， **`AbstractMethodError`通常在编译后的类或 JAR 文件之间存在不兼容时抛出。**

接下来，我们来了解一下这个错误是如何发生的。

## 3.这个错误是如何发生的

当我们构建一个应用程序时，通常我们会导入一些库来简化我们的工作。

比方说，在我们的应用程序中，我们包含了一个`baeldung-queue`库。`baeldung-queue`库是一个高级规范库，它只包含一个接口:

```java
public interface BaeldungQueue {
    void enqueue(Object o);
    Object dequeue();
} 
```

同样，为了使用`BaeldungQueue` 接口，我们导入了一个`BaeldungQueue`实现库:`good-queue`。`good-queue`库也只有一个类:

```java
public class GoodQueue implements BaeldungQueue {
    @Override
    public void enqueue(Object o) {
       //implementation 
    }

    @Override
    public Object dequeue() {
        //implementation 
    }
} 
```

现在，如果`good-queue` 和`baeldung-queue`都在类路径中，我们可以在应用程序中创建一个`BaeldungQueue`实例:

```java
public class Application {
    BaeldungQueue queue = new GoodQueue();

    public void someMethod(Object element) {
        queue.enqueue(element);
        // ...
        queue.dequeue();
        // ...
    }
} 
```

到目前为止，一切顺利。

有一天，我们得知`baeldung-queue` 发布了版本`2.0`,并且它采用了一种新的方法:

```java
public interface BaeldungQueue {
    void enqueue(Object o);
    Object dequeue();

    int size();
} 
```

我们希望在我们的应用程序中使用新的`size()`方法。因此，我们将`baeldung-queue`库从`1.0`升级到`2.0`。然而，我们忘记检查是否有新版本的`good-queue` 库实现了 *BaeldungQueue* 接口变更。

因此，我们在类路径中有`good-queue 1.0` 和`baeldung-queue 2.0`。

此外，我们开始在应用程序中使用新方法:

```java
public class Application {
    BaeldungQueue queue = new GoodQueue();

    public void someMethod(Object element) {
        // ...
        int size = queue.size(); //<-- AbstractMethodError will be thrown
        // ...
    }
} 
```

我们的代码编译起来不会有任何问题。

但是，当运行时执行行`queue.size()`时，会抛出一个`AbstractMethodError`。这是因为`good-queue` `1.0`库没有实现 *BaeldungQueue* 接口中的方法`size()`。

## 4.现实世界的例子

通过简单的`BaeldungQueue`和`GoodQueue` 场景，我们可以知道应用程序何时会抛出`AbstractMethodError. `

在本节中，我们将看到一个`AbstractMethodError`的实际例子。

`[java.sql.Connection](https://web.archive.org/web/20221208143917/https://docs.oracle.com/en/java/javase/11/docs/api/java.sql/java/sql/Connection.html)`是 JDBC API 中的一个重要接口。从 1.7 版本开始，`Connection`接口增加了几个新方法，比如`[getSchema()](https://web.archive.org/web/20221208143917/https://docs.oracle.com/en/java/javase/11/docs/api/java.sql/java/sql/Connection.html#getSchema()).`

[H2 数据库](https://web.archive.org/web/20221208143917/https://www.h2database.com/html/main.html)是一个非常快速的开源 SQL 数据库。从版本`1.4.192`开始，它增加了对`java.sql.Connection.getSchema()`方法的支持。然而，在以前的版本中，H2 数据库还没有实现这个方法。

接下来，我们将从旧的 H2 数据库版本`1.4.191`上的 Java 8 应用程序中调用`java.sql.Connection.getSchema()`方法。让我们看看会发生什么。

让我们创建一个单元测试类来验证调用`Connection.getSchema()`方法是否会抛出`AbstractMethodError`:

```java
class AbstractMethodErrorUnitTest {
    private static final String url = "jdbc:h2:mem:A-DATABASE;INIT=CREATE SCHEMA IF NOT EXISTS myschema";
    private static final String username = "sa";

    @Test
    void givenOldH2Database_whenCallgetSchemaMethod_thenThrowAbstractMethodError() throws SQLException {
        Connection conn = DriverManager.getConnection(url, username, "");
        assertNotNull(conn);
        Assertions.assertThrows(AbstractMethodError.class, () -> conn.getSchema());
    }
} 
```

如果我们运行测试，它会通过，确认对`getSchema()`的调用抛出了`AbstractMethodError`。

## 5.结论

有时候我们可能会在运行时看到`AbstractMethodError`。在本文中，我们已经通过例子讨论了错误何时发生。

当我们升级应用程序的一个库时，检查其他依赖项是否正在使用该库并考虑更新相关的依赖项总是一个好的做法。

另一方面，一旦我们面对`AbstractMethodError`，对这个错误有了很好的理解，我们可能会很快解决问题。

和往常一样，GitHub 上的[提供了这篇文章的完整源代码。](https://web.archive.org/web/20221208143917/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-exceptions-3)