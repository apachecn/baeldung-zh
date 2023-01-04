# 在 Java 中使用(未知源)堆栈跟踪

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-unknown-source-stack-trace>

## 1.概观

在这篇短文中，我们将探讨为什么我们可能会在 Java 异常堆栈跟踪中看到未知来源，以及我们如何修复它。

## 2.类调试信息

Java 类文件包含可选的调试信息，以便于调试。我们可以在编译时选择是否将所有调试信息添加到类文件中，以及添加什么。这将决定在运行时有哪些调试信息可用。

让我们研究一下 Java 编译器的帮助文档，看看各种可用的选项:

```java
javac -help

Usage: javac <options> <source files>
where possible options include:
  -g                         Generate all debugging info
  -g:none                    Generate no debugging info
  -g:{lines,vars,source}     Generate only some debugging info
```

Java 编译器的默认行为是将行和源信息添加到类文件中，这相当于`-g:lines,source.`

### 2.1.使用调试选项编译

让我们看看用上面的选项编译 Java 类时会发生什么。我们有一个`Main`类，它有意生成一个`StringIndexOutOfBoundsException`。

根据使用的编译机制，我们必须相应地指定编译选项。这里，我们将使用 Maven 及其编译器插件来定制编译器选项:

```java
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-compiler-plugin</artifactId>
    <version>3.8.1</version>
    <configuration>
        <compilerArgs>
            <arg>-g:none</arg>
        </compilerArgs>
    </configuration>
</plugin>
```

我们已经将`-g`设置为`none`,这意味着不会为我们编译的类生成调试信息。运行我们有缺陷的`Main`类会生成堆栈跟踪，在那里我们会看到未知的来源，而不是异常发生的行号。

```java
Exception in thread "main" java.lang.StringIndexOutOfBoundsException: begin 0, end 10, length 5
  at java.base/java.lang.String.checkBoundsBeginEnd(String.java:3751)
  at java.base/java.lang.String.substring(String.java:1907)
  at com.baeldung.unknownsourcestacktrace.Main.getShortenedName(Unknown Source)
  at com.baeldung.unknownsourcestacktrace.Main.getGreetingMessage(Unknown Source)
  at com.baeldung.unknownsourcestacktrace.Main.main(Unknown Source) 
```

让我们看看生成的类文件包含什么。我们将使用`javap `来完成这项工作，它是 [Java 类文件反汇编器](/web/20220627172553/https://www.baeldung.com/java-class-view-bytecode):

```java
javap -l -p Main.class

public class com.baeldung.unknownsourcestacktrace.Main {
    private static final org.slf4j.Logger logger;
    private static final int SHORT_NAME_LIMIT;
    public com.baeldung.unknownsourcestacktrace.Main();
    public static void main(java.lang.String[]);
    private static java.lang.String getGreetingMessage(java.lang.String);
    private static java.lang.String getShortenedName(java.lang.String);
    static {};
}
```

可能很难知道我们在这里应该得到什么样的调试信息，所以让我们更改编译选项，看看会发生什么。

### 2.3.修复

现在让我们将编译选项改为`-g:lines,vars,source` ，这将把`LineNumberTable,` `LocalVariableTable` 和`Source` 信息放入我们的类文件中。这也相当于只有`-g `来存放所有的调试信息:

```java
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-compiler-plugin</artifactId>
    <version>3.8.1</version>
    <configuration>
        <compilerArgs>
            <arg>-g</arg>
        </compilerArgs>
    </configuration>
</plugin>
```

再次运行我们的 buggy `Main` 类现在会产生:

```java
Exception in thread "main" java.lang.StringIndexOutOfBoundsException: begin 0, end 10, length 5
  at java.base/java.lang.String.checkBoundsBeginEnd(String.java:3751)
  at java.base/java.lang.String.substring(String.java:1907)
  at com.baeldung.unknownsourcestacktrace.Main.getShortenedName(Main.java:23)
  at com.baeldung.unknownsourcestacktrace.Main.getGreetingMessage(Main.java:19)
  at com.baeldung.unknownsourcestacktrace.Main.main(Main.java:15)
```

瞧，我们在堆栈跟踪中看到了行号信息。让我们看看我们的类文件中发生了什么变化:

```java
javap -l -p Main

Compiled from "Main.java"
public class com.baeldung.unknownsourcestacktrace.Main {
  private static final org.slf4j.Logger logger;

  private static final int SHORT_NAME_LIMIT;

  public com.baeldung.unknownsourcestacktrace.Main();
    LineNumberTable:
      line 7: 0
    LocalVariableTable:
      Start  Length  Slot  Name   Signature
          0       5     0  this   Lcom/baeldung/unknownsourcestacktrace/Main;

  public static void main(java.lang.String[]);
    LineNumberTable:
      line 12: 0
      line 13: 8
      line 15: 14
      line 16: 29
    LocalVariableTable:
      Start  Length  Slot  Name   Signature
          0      30     0  args   [Ljava/lang/String;
          8      22     1  user   Lcom/baeldung/unknownsourcestacktrace/dto/User;

  private static java.lang.String getGreetingMessage(java.lang.String);
    LineNumberTable:
      line 19: 0
    LocalVariableTable:
      Start  Length  Slot  Name   Signature
          0      28     0  name   Ljava/lang/String;

  private static java.lang.String getShortenedName(java.lang.String);
    LineNumberTable:
      line 23: 0
    LocalVariableTable:
      Start  Length  Slot  Name   Signature
          0       8     0  name   Ljava/lang/String;

  static {};
    LineNumberTable:
      line 8: 0
}
```

我们的类文件现在包含三条重要的信息:

1.  **Source** ，顶部头表示`.java`文件，从该文件生成了`.class`文件。在堆栈跟踪的上下文中，它提供发生异常的类名。
2.  **LineNumberTable** 将 JVM 实际运行的代码中的行号映射到我们的源代码文件中的行号。在堆栈跟踪的上下文中，它提供发生异常的行号。我们还需要它能够在我们的调试器中使用断点。
3.  **LocalVariableTable** 包含了获取局部变量的值的细节。调试器可以使用它来读取局部变量的值。在堆栈跟踪的上下文中，这无关紧要。

## 3.结论

我们现在已经熟悉了 Java 编译器生成的调试信息。操作它们的方式，`-g` 编译器选项。我们看到了如何用 Maven 编译器插件做到这一点。

因此，如果我们在堆栈跟踪中发现未知来源，我们可以调查我们的类文件，检查调试信息是否可用。接下来我们可以根据我们的构建工具选择正确的编译选项来解决这个问题。

和往常一样，完整的代码和 Maven 配置可以在 GitHub 上的[处获得。](https://web.archive.org/web/20220627172553/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-exceptions-3)