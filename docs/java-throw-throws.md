# Java 中抛出和抛出的区别

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-throw-throws>

## 1.介绍

在本教程中，我们将看看 Java 中的`throw`和`throws`。我们将解释何时应该使用它们。

接下来，我们将展示一些它们基本用法的例子。

## 2.`Throw`和`Throws`

让我们从一个简单的介绍开始。这些关键字与异常处理相关。当我们的应用程序的正常流程被中断时，就会出现异常。

可能有很多原因。用户可能会发送错误的输入数据。我们可能会失去连接或发生其他意外情况。良好的异常处理是在那些不愉快的时刻出现后保持我们的应用程序工作的关键。

**我们使用 `throw`关键字从代码中显式抛出一个异常**。它可以是任何方法或静态块。这个异常必须是`Throwable.` 的子类，也可以是`Throwable`本身。我们不能用一个`throw`抛出多个异常。

`Throws`关键字可以放在方法声明中。**它表示这个方法可以抛出哪些异常。**我们必须用 try-catch 处理这些异常。

这两个关键词不能互换！

## 3.`Throw`在爪哇

让我们看一个从方法中抛出异常的基本例子。

首先，想象我们正在编写一个简单的计算器。基本的算术运算之一是除法。因此，我们被要求实现这一功能:

```java
public double divide(double a, double b) {
    return a / b;
}
```

因为我们不能被零除，所以我们需要对现有代码进行一些修改。看起来这是提出一个例外的好时机。

让我们这样做:

```java
public double divide(double a, double b) {
    if (b == 0) {
        throw new ArithmeticException("Divider cannot be equal to zero!");
    }
    return a / b;
}
```

如你所见，我们使用了完全符合我们需求的`ArithmeticException`。我们可以传递一个单独的`String` 构造函数参数，它是异常消息。

### 3.1.良好做法

我们应该总是喜欢最具体的例外。我们需要找到最适合我们特殊活动的课程。例如，抛出`NumberFormatException `而不是`IllegalArgumentException. `我们应该避免抛出不具体的`Exception`。

比如`java.lang`包里有一个`Integer`类。让我们来看看其中一个工厂方法声明:

```java
public static Integer valueOf(String s) throws NumberFormatException 
```

这是一个静态工厂方法，从`String. `创建`Integer`实例，如果输入`String`错误，该方法将抛出`NumberFormatException.`

**一个好主意是定义我们自己的、更具描述性的异常。**在我们的`Calculator `类中，例如`DivideByZeroException. `

让我们看一下示例实现:

```java
public class DivideByZeroException extends RuntimeException {

    public DivideByZeroException(String message) {
        super(message);
    }
}
```

### 3.2.包装现有异常

有时，我们希望将现有的异常包装到我们定义的异常中。

让我们从定义我们自己的异常开始:

```java
public class DataAcessException extends RuntimeException {

    public DataAcessException(String message, Throwable cause) {
        super(message, cause);
    }
}
```

构造函数有两个参数:异常消息和原因，原因可以是`Throwable. `的任何子类

让我们为`findAll() `函数写一个假的实现:

```java
public List<String> findAll() throws SQLException {
    throw new SQLException();
}
```

现在，在`SimpleService`中，让我们调用一个存储库函数，它可以产生`SQLException:`

```java
public void wrappingException() {
    try {
        personRepository.findAll();
    } catch (SQLException e) {
        throw new DataAccessException("SQL Exception", e);
    }
}
```

我们重新抛出包装在我们自己的名为`DataAccessException. `的异常中的`SQLException`,下面的测试验证了一切:

```java
@Test
void whenSQLExceptionIsThrown_thenShouldBeRethrownWithWrappedException() {
    assertThrows(DataAccessException.class,
      () -> simpleService.wrappingException());
}
```

这样做有两个原因。首先，我们使用异常包装，因为代码的其余部分不需要知道系统中每个可能的异常。

此外，较高级别的组件不需要知道底层组件，也不需要知道它们抛出的异常。

### 3.3.用 Java 实现多重捕捉

有时，我们使用的方法会抛出许多不同的异常。

让我们来看看更广泛的 try-catch 块:

```java
try {
    tryCatch.execute();
} catch (ConnectionException | SocketException ex) {
    System.out.println("IOException");
} catch (Exception ex) {
    System.out.println("General exception");
}
```

`execute`方法可以抛出三个异常:`SocketException, ConnectionException, Exception.`第一个 catch 块会捕捉到`ConnectionException`或者`SocketException`。第二个 catch 块将捕获`Exception `或`Exception. `的任何其他子类。记住，**我们应该总是首先捕获更详细的异常。**

我们可以交换 catch 块的顺序。然后，我们永远也抓不到`SocketException`和`ConnectionException`，因为一切都会被`Exception`抓住。

## 4.`Throws`在爪哇

我们将`throws `添加到方法声明中。

让我们看一下我们以前的一个方法声明:

```java
public static void execute() throws SocketException, ConnectionException, Exception
```

**该方法可能抛出多个异常。**它们在方法声明的末尾用逗号分隔。我们可以把检查过的和未检查过的异常都放在`throws. `中，我们已经在下面描述了它们之间的区别。

### 4.1.已检查和未检查的异常

被检查的异常意味着它是在编译时被检查的。注意，我们必须处理这个异常。否则，方法必须通过使用`throws`关键字来指定异常。

最常见的检查异常是当我们从`File. `创建`FileInputStream `时可能抛出的`IOException, FileNotFoundException, ParseException. FileNotFoundException `

有一个简短的例子:

```java
File file = new File("not_existing_file.txt");
try {
    FileInputStream stream = new FileInputStream(file);
} catch (FileNotFoundException e) {
    e.printStackTrace();
}
```

我们可以通过在方法声明中添加`throws `来避免使用 try-catch 块:

```java
private static void uncheckedException() throws FileNotFoundException {
    File file = new File("not_existing_file.txt");
    FileInputStream stream = new FileInputStream(file);
}
```

不幸的是，一个更高级的函数仍然必须处理这个异常。否则，我们必须用`throws keyword.`将这个异常放在方法声明中

相反，**未检查的异常在编译时不会被检查。**

最常见的未检查异常有:`ArrayIndexOutOfBoundsException, IllegalArgumentException, NullPointerException. `

**运行时抛出未检查的异常。**下面的代码将抛出一个`NullPointerException.` 可能这是 Java 中最常见的异常之一。

对空引用调用方法将导致以下异常:

```java
public void runtimeNullPointerException() {
    String a = null;
    a.length();
}
```

让我们在测试中验证这一行为:

```java
@Test
void whenCalled_thenNullPointerExceptionIsThrown() {
    assertThrows(NullPointerException.class,
      () -> simpleService.runtimeNullPointerException());
}
```

请记住，这个代码和测试没有任何意义。解释运行时异常只是为了学习。

在 Java 中，`Error`和`RuntimeException`的每个子类都是未检查的异常。被检查的例外是`Throwable `类下的其他所有东西。

## 5.结论

在本文中，我们讨论了两个 Java 关键字:`throw`和`throws. `之间的区别。我们已经过了基本用法，谈了一些好的实践`. `，然后我们讨论了检查和未检查的异常。

和往常一样，源代码可以在我们的 GitHub 上找到[。](https://web.archive.org/web/20221208143849/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-exceptions)

如果您想更深入地了解 Java 中的异常处理，请看看我们关于 Java 异常的文章[。](/web/20221208143849/https://www.baeldung.com/java-exceptions)