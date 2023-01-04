# Java 中的检查异常和未检查异常

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-checked-unchecked-exceptions>

## 1.概观

Java 异常分为两大类:**检查异常和未检查异常。**

在本教程中，我们将提供一些关于如何使用它们的代码示例。

## 2.检查异常

一般来说，被检查的异常表示程序控制之外的错误。例如， [`FileInputStream`](https://web.archive.org/web/20221208143840/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/io/FileInputStream.html#%3Cinit%3E(java.io.File)) 的构造函数在输入文件不存在的情况下抛出 *FileNotFoundException* 。

Java 在编译时验证检查过的异常。

因此，我们应该使用 [`throws`](/web/20221208143840/https://www.baeldung.com/java-throw-throws) 关键字来声明一个被检查的异常:

```java
private static void checkedExceptionWithThrows() throws FileNotFoundException {
    File file = new File("not_existing_file.txt");
    FileInputStream stream = new FileInputStream(file);
}
```

我们还可以使用一个`try-catch`块来处理一个被检查的异常:

```java
private static void checkedExceptionWithTryCatch() {
    File file = new File("not_existing_file.txt");
    try {
        FileInputStream stream = new FileInputStream(file);
    } catch (FileNotFoundException e) {
        e.printStackTrace();
    }
}
```

Java 中常见的一些[检查异常](/web/20221208143840/https://www.baeldung.com/java-common-exceptions)有`IOException`、`SQLException`和`ParseException`。

[`Exception`](https://web.archive.org/web/20221208143840/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/Exception.html) 类是检查异常的超类，所以我们可以通过扩展`Exception`[创建一个定制的检查异常](/web/20221208143840/https://www.baeldung.com/java-new-custom-exception):

```java
public class IncorrectFileNameException extends Exception {
    public IncorrectFileNameException(String errorMessage) {
        super(errorMessage);
    }
} 
```

## 3.未检查的异常

如果一个程序抛出一个未检查的异常，它反映了程序逻辑内部的一些错误。

例如，如果我们将一个数除以 0，Java 会抛出`ArithmeticException`:

```java
private static void divideByZero() {
    int numerator = 1;
    int denominator = 0;
    int result = numerator / denominator;
} 
```

**Java 在编译时不验证未检查的异常。**此外，我们不必用`throws `关键字在方法中声明未检查的异常。尽管上面的代码在编译时没有任何错误，但它会在运行时抛出`ArithmeticException`。

Java 中常见的一些[未检查异常](/web/20221208143840/https://www.baeldung.com/java-common-exceptions)有`NullPointerException`、`ArrayIndexOutOfBoundsException`和`IllegalArgumentException`。

`[RuntimeException](https://web.archive.org/web/20221208143840/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/RuntimeException.html)` 类是所有未检查异常的超类，所以我们可以通过扩展`RuntimeException`来[创建一个定制的未检查异常](/web/20221208143840/https://www.baeldung.com/java-new-custom-exception):

```java
public class NullOrEmptyException extends RuntimeException {
    public NullOrEmptyException(String errorMessage) {
        super(errorMessage);
    }
}
```

## 4.何时使用已检查的异常和未检查的异常

在 Java 中使用异常是一个很好的实践，这样我们可以将错误处理代码与常规代码分开。然而，我们需要决定抛出哪种类型的异常。 [Oracle Java 文档](https://web.archive.org/web/20221208143840/https://docs.oracle.com/javase/tutorial/essential/exceptions/runtime.html)提供了何时使用检查异常和未检查异常的指导:

“如果可以合理地预期客户端会从异常中恢复，请将其设为已检查的异常。如果客户端无法从异常中恢复，请将其设为未检查的异常。

例如，在我们打开一个文件之前，我们可以首先验证输入文件名。如果用户输入的文件名无效，我们可以抛出一个自定义的检查异常:

```java
if (!isCorrectFileName(fileName)) {
    throw new IncorrectFileNameException("Incorrect filename : " + fileName );
} 
```

这样，我们可以通过接受另一个用户输入的文件名来恢复系统。

但是，如果输入文件名是一个空指针或者是一个空字符串，这意味着我们在代码中有一些错误。在这种情况下，我们应该抛出一个未检查的异常:

```java
if (fileName == null || fileName.isEmpty())  {
    throw new NullOrEmptyException("The filename is null or empty.");
} 
```

## 5.结论

在本文中，我们讨论了检查异常和未检查异常之间的区别。我们还提供了一些代码示例来展示何时使用检查过的或未检查过的异常。

和往常一样，本文中的所有代码都可以在 GitHub 上找到[。](https://web.archive.org/web/20221208143840/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-exceptions)