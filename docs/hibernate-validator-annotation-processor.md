# 深度 Hibernate 验证器注释处理器

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/hibernate-validator-annotation-processor>

## 1。概述

很容易误用 [bean 验证](/web/20221206025029/https://www.baeldung.com/javax-validation)约束。例如，我们可能不小心用`@Future`约束修饰了一个`String`属性。这样的错误会导致运行时不可预测的错误。

幸运的是， [Hibernate Validator 注释处理器](https://web.archive.org/web/20221206025029/https://hibernate.org/validator/tooling/)有助于在编译时检测这些问题。多亏了它抛出的错误，我们可以更早地发现这些错误。

在本教程中，我们将探索如何配置处理器，我们将看看它能为我们找到的一些常见问题。

## 2。配置

### 2.1。安装

让我们从将[注释处理器依赖关系](https://web.archive.org/web/20221206025029/https://search.maven.org/artifact/org.hibernate.validator/hibernate-validator-annotation-processor)添加到 pom.xml 开始:

```java
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-compiler-plugin</artifactId>
    <version>3.6.1</version>
    <configuration>
        <source>1.8</source>
        <target>1.8</target>
        <compilerArgs>
            <arg>-Averbose=true</arg>
            <arg>-AmethodConstraintsSupported=true</arg>
            <arg>-AdiagnosticKind=ERROR</arg>
        </compilerArgs>
        <annotationProcessorPaths>
            <path>
                <groupId>org.hibernate.validator</groupId>
                <artifactId>hibernate-validator-annotation-processor</artifactId>
                <version>6.2.0.Final</version>
            </path>
        </annotationProcessorPaths>
    </configuration>
</plugin>
```

我们需要注意的是**版本 7 的这个工具只兼容 [`jakarta.validation`](https://web.archive.org/web/20221206025029/https://search.maven.org/search?q=g:jakarta.validation%20AND%20a:jakarta.validation-api) 约束**:

```java
<dependency>
    <groupId>jakarta.validation</groupId>
    <artifactId>jakarta.validation-api</artifactId>
    <version>3.0.1</version>
</dependency>
```

处理器还提供关于如何为主要 Java IDEs 设置它的[指导](https://web.archive.org/web/20221206025029/https://docs.jboss.org/hibernate/stable/validator/reference/en-US/html_single/?v=7.0#validator-annotationprocessor-ide)。

### 2.2。编译器选项

让我们设置处理器编译器选项:

```java
<compilerArgs>
    <arg>-Averbose=true</arg>
    <arg>-AmethodConstraintsSupported=true</arg>
    <arg>-AdiagnosticKind=ERROR</arg>
</compilerArgs> 
```

首先，`diagnosticKind`选项针对日志记录级别。最好保留默认的`ERROR`值，以便在编译时发现问题。所有允许的值都在 [`Diagnostic.Kind`](https://web.archive.org/web/20221206025029/https://docs.oracle.com/en/java/javase/12/docs//api/java.compiler/javax/tools/Diagnostic.Kind.html) 枚举中引用。

接下来，如果我们想将注释验证限制在 getters 上，我们应该将`methodConstraintsSupported`选项设置为`false`。

这里，我们将`verbose`设置为`true`以获得更多的输出，但是如果我们不想要大量的日志输出，我们可以将它设置为`false`。

## 3。常见约束问题

**注释处理器带有一组预定义的错误来检查**。让我们以一个简单的`Message`类为例，仔细看看其中的三个:

```java
public class Message {
    // constructor omitted
}
```

### 3.1。只有 Getters 可以被注释

首先，处理器的默认选项不应该存在这个问题。顾名思义，当我们注释一个非 getter 方法时，它会弹出来。我们需要将`methodConstraintsSupported`选项设置为`true`来允许这样做。

让我们给我们的`Message`类添加三个带注释的方法:

```java
@Min(3)
public boolean broadcast() {
    return true;
}

@NotNull
public void archive() {
}

@AssertTrue
public boolean delete() {
    return false;
} 
```

接下来，我们在配置中将`methodConstraintsSupported`选项设置为`false`:

```java
<compilerArgs>
    <arg>AmethodConstraintsSupported=false</arg>
</compilerArgs>
```

最后，这三种方法将使处理器检测到我们的问题:

```java
[ERROR] COMPILATION ERROR :
[INFO] -------------------------------------------------------------
[ERROR] ${home}\baeldung\tutorials\javaxval\src\main\java\com\baeldung\javaxval\methodvalidation\model\ReservationManagement.java:[25,4] error: Constraint annotations must not be specified at methods, which are no valid JavaBeans getter methods.
[ERROR] ${home}\baeldung\tutorials\javaxval\src\main\java\com\baeldung\javaxval\hibernate\validator\ap\Message.java:[55,4] error: Constraint annotations must not be specified at methods, which are no valid JavaBeans getter methods.
[ERROR] ${home}\baeldung\tutorials\javaxval\src\main\java\com\baeldung\javaxval\hibernate\validator\ap\Message.java:[38,5] error: Constraint annotations must not be specified at methods, which are no valid JavaBeans getter methods.
[ERROR] ${home}\baeldung\tutorials\javaxval\src\main\java\com\baeldung\javaxval\hibernate\validator\ap\Message.java:[47,4] error: Constraint annotations must not be specified at methods, which are no valid JavaBeans getter methods.
[INFO] 4 errors
[INFO] -------------------------------------------------------------
[INFO] ------------------------------------------------------------------------
[INFO] BUILD FAILURE
[INFO] ------------------------------------------------------------------------
[INFO] Total time:  5.457 s
[INFO] Finished at: 2022-01-20T21:42:47Z
[INFO] ------------------------------------------------------------------------
[ERROR] Failed to execute goal org.apache.maven.plugins:maven-compiler-plugin:3.6.1:compile (default-compile) on project javaxval: Compilation failure: Compilation failure:
[ERROR] ${home}\baeldung\tutorials\javaxval\src\main\java\com\baeldung\javaxval\methodvalidation\model\ReservationManagement.java:[25,4] error: Constraint annotations must not be specified at methods, which are no valid JavaBeans getter methods.
[ERROR] ${home}\baeldung\tutorials\javaxval\src\main\java\com\baeldung\javaxval\hibernate\validator\ap\Message.java:[55,4] error: Constraint annotations must not be specified at methods, which are no valid JavaBeans getter methods.
[ERROR] ${home}\baeldung\tutorials\javaxval\src\main\java\com\baeldung\javaxval\hibernate\validator\ap\Message.java:[38,5] error: Constraint annotations must not be specified at methods, which are no valid JavaBeans getter methods.
[ERROR] ${home}\baeldung\tutorials\javaxval\src\main\java\com\baeldung\javaxval\hibernate\validator\ap\Message.java:[47,4] error: Constraint annotations must not be specified at methods, which are no valid JavaBeans getter methods. 
```

**有趣的是注意到`delete`方法受到这个问题的影响，尽管从技术上来说，它被适当地注释了。**

对于接下来的部分，我们将把`methodConstraintsSupported`选项设置回`true`。

### 3.2。只有非 Void 方法可以被注释

这个问题表明我们不应该用约束验证来修饰`void`方法。我们可以通过在我们的`Message`类中注释一个`archive`方法来看到它的作用:

```java
@NotNull
public void archive() {
} 
```

它会导致处理器产生一个错误:

```java
[ERROR] COMPILATION ERROR :
[INFO] -------------------------------------------------------------
[ERROR] ${home}\baeldung\tutorials\javaxval\src\main\java\com\baeldung\javaxval\hibernate\validator\ap\Message.java:[45,4] error: Void methods may not be annotated with constraint annotations.
[INFO] 1 error
[INFO] -------------------------------------------------------------
[INFO] ------------------------------------------------------------------------
[INFO] BUILD FAILURE
[INFO] ------------------------------------------------------------------------
[INFO] Total time:  4.078 s
[INFO] Finished at: 2022-01-20T21:35:08Z
[INFO] ------------------------------------------------------------------------
[ERROR] Failed to execute goal org.apache.maven.plugins:maven-compiler-plugin:3.6.1:compile (default-compile) on project javaxval: Compilation failure
[ERROR] ${home}\baeldung\tutorials\javaxval\src\main\java\com\baeldung\javaxval\hibernate\validator\ap\Message.java:[45,4] error: Void methods may not be annotated with constraint annotations. 
```

### 3.3。不支持的注释类型

最后一个问题是最常见的。当批注目标数据类型与目标属性不匹配时，会出现这种情况。为了在我们的`Message`类中看到它的运行，让我们给我们的`Message`类添加一个错误注释的`String`属性:

```java
@Past 
private String createdAt; 
```

由于`@Past`注释，将会出现错误。事实上，只有日期类型可以使用此约束:

```java
[ERROR] COMPILATION ERROR :
[INFO] -------------------------------------------------------------
[ERROR] ${home}\baeldung\tutorials\javaxval\hibernate\validator\ap\Message.java:[20,5] error: The annotation @Past is disallowed for this data type.
[INFO] 1 error
[INFO] -------------------------------------------------------------
[INFO] ------------------------------------------------------------------------
[INFO] BUILD FAILURE
[INFO] ------------------------------------------------------------------------
[INFO] Total time:  4.892 s
[INFO] Finished at: 2022-01-20T21:29:15Z
[INFO] ------------------------------------------------------------------------
[ERROR] Failed to execute goal org.apache.maven.plugins:maven-compiler-plugin:3.6.1:compile (default-compile) on project javaxval: Compilation failure
[ERROR] ${home}\baeldung\tutorials\javaxval\src\main\java\com\baeldung\javaxval\hibernate\validator\ap\Message.java:[20,5] error: The annotation @Past is disallowed for this data type. 
```

如果我们将错误的注释应用于具有不支持的返回类型的方法，我们会得到类似的错误:

```java
@Min(3)
public boolean broadcast() { 
    return true;
} 
```

处理器错误信息与上一条相同:

```java
[ERROR] COMPILATION ERROR :
[INFO] -------------------------------------------------------------
[ERROR] ${home}\baeldung\tutorials\javaxval\src\main\java\com\baeldung\javaxval\hibernate\validator\ap\Message.java:[37,5] error: The annotation @Min is disallowed for the return type of this method.
[INFO] 1 error
[INFO] -------------------------------------------------------------
[INFO] ------------------------------------------------------------------------
[INFO] BUILD FAILURE
[INFO] ------------------------------------------------------------------------
[INFO] Total time:  3.761 s
[INFO] Finished at: 2022-01-20T21:38:28Z
[INFO] ------------------------------------------------------------------------
[ERROR] Failed to execute goal org.apache.maven.plugins:maven-compiler-plugin:3.6.1:compile (default-compile) on project javaxval: Compilation failure
[ERROR] ${home}\baeldung\tutorials\javaxval\src\main\java\com\baeldung\javaxval\hibernate\validator\ap\Message.java:[37,5] error: The annotation @Min is disallowed for the return type of this method. 
```

## 4。结论

在本文中，我们尝试了 Hibernate Validator 注释处理器。

首先，我们安装了它并配置了它的选项。然后，我们用三个常见的约束问题探讨了它的行为。

和往常一样，示例代码可以在 GitHub 上找到[。](https://web.archive.org/web/20221206025029/https://github.com/eugenp/tutorials/tree/master/javaxval-2)