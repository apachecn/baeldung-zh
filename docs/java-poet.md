# JavaPoet 简介

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-poet>

## 1。概述

在本教程中，我们将探索 [JavaPoet](https://web.archive.org/web/20220703144239/https://github.com/square/javapoet) 库的基本功能。

**JavaPoet** 由 [Square](https://web.archive.org/web/20220703144239/https://square.github.io/) 开发，**提供 API 生成 Java 源代码**。它可以生成基本类型、引用类型及其变体(如类、接口、枚举类型、匿名内部类)、字段、方法、参数、注释和 Javadocs。

JavaPoet 自动管理依赖类的导入。它还使用 Builder 模式来指定生成 Java 代码的逻辑。

## 2。Maven 依赖关系

为了使用 JavaPoet，我们可以直接下载最新的 [JAR 文件](https://web.archive.org/web/20220703144239/https://search.maven.org/artifact/com.squareup/javapoet/1.11.1/jar)，或者在我们的`pom.xml:`中定义如下依赖关系

```java
<dependency>
    <groupId>com.squareup</groupId>
    <artifactId>javapoet</artifactId>
    <version>1.10.0</version>
</dependency>
```

## 3。方法说明

首先，让我们看一下方法规范。要生成一个方法，我们只需调用`MethodSpec`类的`methodBuilder()`方法。我们将生成的方法名指定为`methodBuilder()`方法的`String`参数。

我们可以使用`addStatement()`方法**生成任何一个以分号**结尾的逻辑语句。同时，我们可以在一个控制流中定义一个用花括号括起来的控制流，比如`if-else`块，或者`for`循环。

这里有一个简单的例子——生成`sumOfTen()`方法来计算从 0 到 10 的数字之和:

```java
MethodSpec sumOfTen = MethodSpec
  .methodBuilder("sumOfTen")
  .addStatement("int sum = 0")
  .beginControlFlow("for (int i = 0; i <= 10; i++)")
  .addStatement("sum += i")
  .endControlFlow()
  .build();
```

这将产生以下输出:

```java
void sumOfTen() {
    int sum = 0;
    for (int i = 0; i <= 10; i++) {
        sum += i;
    }
}
```

## 4.码组

我们还可以**将一个或多个控制流和逻辑语句打包到一个代码块**:

```java
CodeBlock sumOfTenImpl = CodeBlock
  .builder()
  .addStatement("int sum = 0")
  .beginControlFlow("for (int i = 0; i <= 10; i++)")
  .addStatement("sum += i")
  .endControlFlow()
  .build();
```

这会产生:

```java
int sum = 0;
for (int i = 0; i <= 10; i++) {
    sum += i;
}
```

我们可以通过调用`addCode()`并提供`sumOfTenImpl`对象来简化`MethodSpec`中的早期逻辑:

```java
MethodSpec sumOfTen = MethodSpec
  .methodBuilder("sumOfTen")
  .addCode(sumOfTenImpl)
  .build();
```

代码块也适用于其他规范，如类型和 Javadocs。

## 5。字段规格

接下来，让我们探索一下字段规范逻辑。

为了生成一个字段，我们使用了`FieldSpec` 类的`builder()`方法:

```java
FieldSpec name = FieldSpec
  .builder(String.class, "name")
  .addModifiers(Modifier.PRIVATE)
  .build();
```

这将生成以下字段:

```java
private String name;
```

我们还可以通过调用`initializer()`方法来初始化字段的默认值:

```java
FieldSpec defaultName = FieldSpec
  .builder(String.class, "DEFAULT_NAME")
  .addModifiers(Modifier.PRIVATE, Modifier.STATIC, Modifier.FINAL)
  .initializer("\"Alice\"")
  .build();
```

这会产生:

```java
private static final String DEFAULT_NAME = "Alice";
```

## 6.参数说明

现在让我们来探索参数规范逻辑。

如果我们想给方法添加一个参数，我们可以在构建器的函数调用链中调用`addParameter()`。

对于更复杂的参数类型，我们可以使用`ParameterSpec` builder:

```java
ParameterSpec strings = ParameterSpec
  .builder(
    ParameterizedTypeName.get(ClassName.get(List.class), TypeName.get(String.class)), 
    "strings")
  .build();
```

我们还可以添加方法的修饰符，比如`public`和/或`static:`

```java
MethodSpec sumOfTen = MethodSpec
  .methodBuilder("sumOfTen")
  .addParameter(int.class, "number")
  .addParameter(strings)
  .addModifiers(Modifier.PUBLIC, Modifier.STATIC)
  .addCode(sumOfTenImpl)
  .build();
```

下面是生成的 Java 代码的样子:

```java
public static void sumOfTen(int number, List<String> strings) {
    int sum = 0;
    for (int i = 0; i <= 10; i++) {
        sum += i;
    }
}
```

## 7。型号规格

在探索了生成方法、字段和参数的方法之后，现在让我们来看看类型规范。

要声明一个类型，我们可以使用 **`TypeSpec`，它可以构建类、接口和枚举类型**。

### 7.1。生成一个类

为了生成一个类，我们可以使用`TypeSpec`类的`classBuilder()`方法。

我们还可以指定它的修饰符，例如，`public `和 `final`访问修饰符。除了类修饰符，我们还可以使用已经提到的`FieldSpec`和`MethodSpec`类来指定字段和方法。

注意，`addField()`和`addMethod()`方法在生成接口或匿名内部类时也是可用的。

让我们看看下面的类生成器示例:

```java
TypeSpec person = TypeSpec
  .classBuilder("Person")
  .addModifiers(Modifier.PUBLIC)
  .addField(name)
  .addMethod(MethodSpec
    .methodBuilder("getName")
    .addModifiers(Modifier.PUBLIC)
    .returns(String.class)
    .addStatement("return this.name")
    .build())
  .addMethod(MethodSpec
    .methodBuilder("setName")
    .addParameter(String.class, "name")
    .addModifiers(Modifier.PUBLIC)
    .returns(String.class)
    .addStatement("this.name = name")
    .build())
  .addMethod(sumOfTen)
  .build();
```

下面是生成的代码:

```java
public class Person {
    private String name;

    public String getName() {
        return this.name;
    }

    public String setName(String name) {
        this.name = name;
    }

    public static void sumOfTen(int number, List<String> strings) {
        int sum = 0;
        for (int i = 0; i <= 10; i++) {
            sum += i;
        }
    }
}
```

### 7.2。生成接口

为了生成 Java 接口，我们使用了`TypeSpec.` 的`interfaceBuilder()`方法

我们还可以通过在`addModifiers()`中指定`DEFAULT`修饰符值来定义一个默认方法:

```java
TypeSpec person = TypeSpec
  .interfaceBuilder("Person")
  .addModifiers(Modifier.PUBLIC)
  .addField(defaultName)
  .addMethod(MethodSpec
    .methodBuilder("getName")
    .addModifiers(Modifier.PUBLIC, Modifier.ABSTRACT)
    .build())
  .addMethod(MethodSpec
    .methodBuilder("getDefaultName")
    .addModifiers(Modifier.PUBLIC, Modifier.DEFAULT)
    .addCode(CodeBlock
      .builder()
      .addStatement("return DEFAULT_NAME")
      .build())
    .build())
  .build();
```

它将生成以下 Java 代码:

```java
public interface Person {
    private static final String DEFAULT_NAME = "Alice";

    void getName();

    default void getDefaultName() {
        return DEFAULT_NAME;
    }
}
```

### 7.3。生成枚举

要生成枚举类型，我们可以使用`TypeSpec`的`enumBuilder()`方法。要指定每个枚举值，我们可以调用`addEnumConstant()`方法:

```java
TypeSpec gender = TypeSpec
  .enumBuilder("Gender")
  .addModifiers(Modifier.PUBLIC)
  .addEnumConstant("MALE")
  .addEnumConstant("FEMALE")
  .addEnumConstant("UNSPECIFIED")
  .build();
```

前述`enumBuilder()`逻辑的输出为:

```java
public enum Gender {
    MALE,
    FEMALE,
    UNSPECIFIED
}
```

### 7.4。生成匿名内部类

要生成匿名内部类，我们可以使用`TypeSpec`类的`anonymousClassBuilder()`方法。注意，**我们必须在`addSuperinterface()`方法**中指定父类。否则，它将使用默认的父类，即`Object`:

```java
TypeSpec comparator = TypeSpec
  .anonymousClassBuilder("")
  .addSuperinterface(ParameterizedTypeName.get(Comparator.class, String.class))
  .addMethod(MethodSpec
    .methodBuilder("compare")
    .addModifiers(Modifier.PUBLIC)
    .addParameter(String.class, "a")
    .addParameter(String.class, "b")
    .returns(int.class)
    .addStatement("return a.length() - b.length()")
    .build())
  .build();
```

这将生成以下 Java 代码:

```java
new Comparator<String>() {
    public int compare(String a, String b) {
        return a.length() - b.length();
    }
});
```

## 8。注释规范

为了给生成的代码添加注释，我们可以在`MethodSpec`或`FieldSpec`构建器类中调用`addAnnotation()`方法:

```java
MethodSpec sumOfTen = MethodSpec
  .methodBuilder("sumOfTen")
  .addAnnotation(Override.class)
  .addParameter(int.class, "number")
  .addParameter(strings)
  .addModifiers(Modifier.PUBLIC, Modifier.STATIC)
  .addCode(sumOfTenImpl)
  .build();
```

这会产生:

```java
@Override
public static void sumOfTen(int number, List<String> strings) {
    int sum = 0;
    for (int i = 0; i <= 10; i++) {
        sum += i;
    }
}
```

如果我们需要指定成员值，我们可以调用`AnnotationSpec`类的`addMember()`方法:

```java
AnnotationSpec toString = AnnotationSpec
  .builder(ToString.class)
  .addMember("exclude", "\"name\"")
  .build();
```

这将生成以下注释:

```java
@ToString(
    exclude = "name"
)
```

## 9。生成 javadoc

可以使用`CodeBlock,`或通过直接指定值来生成 Javadoc:

```java
MethodSpec sumOfTen = MethodSpec
  .methodBuilder("sumOfTen")
  .addJavadoc(CodeBlock
    .builder()
    .add("Sum of all integers from 0 to 10")
    .build())
  .addAnnotation(Override.class)
  .addParameter(int.class, "number")
  .addParameter(strings)
  .addModifiers(Modifier.PUBLIC, Modifier.STATIC)
  .addCode(sumOfTenImpl)
  .build();
```

这将生成以下 Java 代码:

```java
/**
 * Sum of all integers from 0 to 10
 */
@Override
public static void sumOfTen(int number, List<String> strings) {
    int sum = 0;
    for (int i = 0; i <= 10; i++) {
        sum += i;
    }
}
```

## 10。格式化

让我们重新检查一下[第 5 节](#fieldspec)中的`FieldSpec`初始化器的例子，它包含一个用于转义“Alice”`String`值的转义符:

```java
initializer("\"Alice\"")
```

在[第 8 节](#annotationspec)中也有一个类似的例子，当我们定义一个注释的被排除成员时:

```java
addMember("exclude", "\"name\"")
```

当我们的 JavaPoet 代码增长并有许多类似的`String`转义或`String`连接语句时，它就变得笨拙了。

JavaPoet 中的字符串格式化特性使得`beginControlFlow()`、 *addStatement()* 或`initializer()`方法中的`String`格式化更加容易。**语法类似于 Java 中的 *String.format()* 功能。它可以帮助格式化文字、字符串、类型和名称**。

### 10.1。文字格式

**JavaPoet 在输出中用文字值替换`$L`。**我们可以在参数中指定任何原始类型和`String`值:

```java
private MethodSpec generateSumMethod(String name, int from, int to, String operator) {
    return MethodSpec
      .methodBuilder(name)
      .returns(int.class)
      .addStatement("int sum = 0")
      .beginControlFlow("for (int i = $L; i <= $L; i++)", from, to)
      .addStatement("sum = sum $L i", operator)
      .endControlFlow()
      .addStatement("return sum")
      .build();
}
```

如果我们用下面指定的值调用`generateSumMethod()`:

```java
generateSumMethod("sumOfOneHundred", 0, 100, "+");
```

JavaPoet 将生成以下输出:

```java
int sumOfOneHundred() {
    int sum = 0;
    for (int i = 0; i <= 100; i++) {
        sum = sum + i;
    }
    return sum;
}
```

### 10.2。`String `格式化

`String`格式化生成一个带引号的值，这个值在 Java 中专门指`String`类型。 **JavaPoet 用输出**中的`String`值替换`$S`:

```java
private static MethodSpec generateStringSupplier(String methodName, String fieldName) {
    return MethodSpec
      .methodBuilder(methodName)
      .returns(String.class)
      .addStatement("return $S", fieldName)
      .build();
}
```

如果我们调用`generateGetter()`方法并提供这些值:

```java
generateStringSupplier("getDefaultName", "Bob");
```

我们将获得以下生成的 Java 代码:

```java
String getDefaultName() {
    return "Bob";
}
```

### 10.3。类型格式化

**JavaPoet 用生成的 Java 代码**中的类型替换`$T`。JavaPoet 自动处理导入语句中的类型。如果我们以文字的形式提供类型，JavaPoet 将不会处理导入。

```java
MethodSpec getCurrentDateMethod = MethodSpec
  .methodBuilder("getCurrentDate")
  .returns(Date.class)
  .addStatement("return new $T()", Date.class)
  .build();
```

JavaPoet 将生成以下输出:

```java
Date getCurrentDate() {
    return new Date();
}
```

### 10.4。名称格式

如果我们需要**引用变量/参数、字段或方法的名称，我们可以在 JavaPoet 的`String`格式化程序中使用`$N`** 。

我们可以将先前的`getCurrentDateMethod()`添加到新的引用方法中:

```java
MethodSpec dateToString = MethodSpec
  .methodBuilder("getCurrentDateAsString")
  .returns(String.class)
  .addStatement(
    "$T formatter = new $T($S)", 
    DateFormat.class, 
    SimpleDateFormat.class, 
    "MM/dd/yyyy HH:mm:ss")
  .addStatement("return formatter.format($N())", getCurrentDateMethod)
  .build();
```

这会产生:

```java
String getCurrentDateAsString() {
    DateFormat formatter = new SimpleDateFormat("MM/dd/yyyy HH:mm:ss");
    return formatter.format(getCurrentDate());
}
```

## 11.生成 Lambda 表达式

我们可以利用已经研究过的特性来生成 Lambda 表达式。例如，多次打印`name`字段或变量的代码块:

```java
CodeBlock printNameMultipleTimes = CodeBlock
  .builder()
  .addStatement("$T<$T> names = new $T<>()", List.class, String.class, ArrayList.class)
  .addStatement("$T.range($L, $L).forEach(i -> names.add(name))", IntStream.class, 0, 10)
  .addStatement("names.forEach(System.out::println)")
  .build();
```

该逻辑生成以下输出:

```java
List<String> names = new ArrayList<>();
IntStream.range(0, 10).forEach(i -> names.add(name));
names.forEach(System.out::println);
```

## 12。使用`JavaFile` 生成输出

**`JavaFile`类帮助配置和产生生成代码**的输出。要生成 Java 代码，我们只需构建`JavaFile,`，提供包名和`TypeSpec`对象的实例。

### 12.1。代码缩进

默认情况下，JavaPoet 使用两个空格进行缩进。为了保持一致性，本教程中的所有示例都使用 4 个空格缩进，这是通过`indent()`方法配置的:

```java
JavaFile javaFile = JavaFile
  .builder("com.baeldung.javapoet.person", person)
  .indent("    ")
  .build();
```

### 12.2。静态导入

如果我们需要添加一个静态导入，我们可以通过调用`addStaticImport()`方法在`JavaFile`中定义类型和具体的方法名:

```java
JavaFile javaFile = JavaFile
  .builder("com.baeldung.javapoet.person", person)
  .indent("    ")
  .addStaticImport(Date.class, "UTC")
  .addStaticImport(ClassName.get("java.time", "ZonedDateTime"), "*")
  .build();
```

它生成以下静态导入语句:

```java
import static java.util.Date.UTC;
import static java.time.ZonedDateTime.*;
```

### 12.3。输出

`writeTo()`方法提供了将代码写入多个目标的功能，比如标准输出流(`System.out`)和`File`。

要将 Java 代码写入标准输出流，我们只需调用`writeTo()`方法，并提供`System.out`作为参数:

```java
JavaFile javaFile = JavaFile
  .builder("com.baeldung.javapoet.person", person)
  .indent("    ")
  .addStaticImport(Date.class, "UTC")
  .addStaticImport(ClassName.get("java.time", "ZonedDateTime"), "*")
  .build();

javaFile.writeTo(System.out);
```

`writeTo()`方法也接受`java.nio.file.Path`和`java.io.File`。我们可以提供相应的`Path`或`File`对象，以便将 Java 源代码文件生成到目标文件夹/路径中:

```java
Path path = Paths.get(destinationPath);
javaFile.writeTo(path);
```

关于`JavaFile`的更多详细信息，请参考 [Javadoc](https://web.archive.org/web/20220703144239/https://square.github.io/javapoet/javadoc/javapoet/com/squareup/javapoet/JavaFile.html) 。

## 13。结论

本文介绍了 JavaPoet 的功能，比如生成方法、字段、参数、类型、注释和 Javadocs。

JavaPoet 只为代码生成而设计。如果我们想用 Java 进行元编程，Java poet 1 . 10 . 0 版不支持代码编译和运行。

与往常一样，GitHub 上的[提供了示例和代码片段。](https://web.archive.org/web/20220703144239/https://github.com/eugenp/tutorials/tree/master/libraries-6)