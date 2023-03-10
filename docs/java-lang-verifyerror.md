# java.lang.VerifyError 的成因及避免

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-lang-verifyerror>

## 1.介绍

在本教程中，我们将看看`java.lang.VerifyError`错误的原因以及避免它的多种方法。

## 2.原因

**[Java 虚拟机(JVM)](/web/20220926200514/https://www.baeldung.com/jvm-vs-jre-vs-jdk) 不信任所有加载的字节码，这是 Java 安全模型**的核心原则。在运行时，JVM 将加载`.class`文件，并试图将它们链接在一起形成一个可执行文件——但是这些加载的`.class`文件的有效性是未知的。

为了确保加载的`.class`文件不会对最终的可执行文件造成威胁，JVM 对`.class`文件执行[验证。此外，JVM 确保二进制文件是格式良好的。例如，JVM 将验证类没有子类型`final`类。](https://web.archive.org/web/20220926200514/https://docs.oracle.com/javase/specs/jvms/se13/html/jvms-4.html#jvms-4.10)

在许多情况下，对有效的、非恶意的字节码的验证会失败，因为新版本的 Java 比旧版本有更严格的验证过程。例如，JDK 协议 13 可能添加了 JDK 协议 7 中没有强制执行的验证步骤。因此，如果我们用 JVM 13 运行一个应用程序，并且包含用旧版本的 [Java 编译器(javac)](/web/20220926200514/https://www.baeldung.com/javac) 编译的依赖项，JVM 可能会认为过时的依赖项是无效的。

因此，当将旧的`.class`文件与新的**JVM 链接时，JVM 可能会抛出类似如下的`java.lang.VerifyError`** :

```java
java.lang.VerifyError: Expecting a stackmap frame at branch target X
Exception Details:
  Location:

com/example/baeldung.Foo(Lcom/example/baeldung/Bar:Baz;)Lcom/example/baeldung/Foo; @1: infonull
  Reason:
    Expected stackmap frame at this location.
  Bytecode:
    0000000: 0001 0002 0003 0004 0005 0006 0007 0008
    0000010: 0001 0002 0003 0004 0005 0006 0007 0008
    ...
```

有两种方法可以解决这个问题:

*   将依赖项更新到用更新的`javac`编译的版本
*   禁用 Java 验证

## 3.生产解决方案

验证错误最常见的原因是使用用旧版本的`javac`编译的新 JVM 版本链接二进制文件。当**依赖项拥有由 [Javassist](/web/20220926200514/https://www.baeldung.com/javassist)** 等工具生成的字节码时，这种情况更常见，如果工具过时，这些工具可能会生成过时的字节码。

为了解决这个问题，**将依赖项更新到使用 JDK 版本构建的** **版本，该版本与用于构建应用程序**的 JDK 版本相匹配。例如，如果我们使用 JDK 13 版构建应用程序，那么依赖项应该使用 JDK 13 版构建。

为了找到兼容的版本，检查依赖项的 [JAR 清单文件](/web/20220926200514/https://www.baeldung.com/java-jar-manifest)中的`Build-Jdk`,以确保它与用于构建应用程序的 JDK 版本相匹配。

## 4.调试和开发解决方案

当调试或开发应用程序时，我们可以禁用验证作为一种快速解决方案。

**不要将此解决方案用于生产代码`.`**

通过禁用验证，JVM 可以将恶意或错误的代码链接到我们的应用程序，从而导致安全危害或执行时崩溃。

还要注意，从 JDK 13 开始，[这个解决方案已经被弃用了](https://web.archive.org/web/20220926200514/https://bugs.openjdk.java.net/browse/JDK-8218003)，我们不应该期望这个解决方案能在未来的 Java 版本中工作。禁用验证将导致以下警告:

```java
Java HotSpot(TM) 64-Bit Server VM warning: Options -Xverify:none and -noverify were deprecated
  in JDK 13 and will likely be removed in a future release.
```

禁用字节码验证的机制因我们运行代码的方式而异。

### 4.1。命令行

要在命令行上禁用验证，请将`noverify`标志传递给`java`命令:

```java
java -noverify Foo.class
```

注意， **`-noverify`是`-Xverify:none`** 的快捷方式，两者可以互换使用**。**

### 4.2 版。肚子

要在 [Maven](/web/20220926200514/https://www.baeldung.com/maven) 构建中禁用验证，将`noverify`标志传递给任何所需的插件:

```java
<plugin>
    <groupId>com.example.baeldung</groupId>
    <artifactId>example-plugin</artifactId>
    <!-- ... -->
    <configuration>
        <argLine>-noverify</argLine>
        <!-- ... -->
    </configuration>
</plugin>
```

### 4.3 版。度

要在 [Gradle](/web/20220926200514/https://www.baeldung.com/gradle) 构建中禁用验证，请将`noverify`标志传递给任何所需的任务:

```java
someTask {
    // ...
    jvmArgs = jvmArgs << "-noverify"
}
```

## 5.结论

在这个快速教程中，我们学习了 JVM 为什么执行字节码验证，以及什么导致了`java.lang.VerifyError`错误。我们还探讨了两种解决方案:一种是生产解决方案，另一种是非生产解决方案。

可能的话，**使用最新版本的依赖关系**，而不是禁用验证。