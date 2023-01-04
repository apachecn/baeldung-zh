# 什么是 Java 中的编译时常量？

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-compile-time-constants>

## 1.概观

Java 语言规范没有定义甚至没有使用编译时常量这个术语。然而，开发人员经常使用这个术语来描述在编译之后没有改变的值**。**

在本教程中，我们将探索类常量和编译时常量之间的区别。我们将查看常量表达式，并了解哪些数据类型和运算符可用于定义编译时常量。最后，我们将看几个编译时常量常用的例子。

## 2.类别常数

当我们在 Java 中使用术语[常量](/web/20220526153741/https://www.baeldung.com/java-constants-good-practices)时，大多数时候，我们指的是`[static](/web/20220526153741/https://www.baeldung.com/java-static)` 和`[final](/web/20220526153741/https://www.baeldung.com/java-final)` 类变量。我们不能在编译后改变类常量的值。因此，**所有原始类型的类常量或`String`也是编译时常量**:

```
public static final int MAXIMUM_NUMBER_OF_USERS = 10;
public static final String DEFAULT_USERNAME = "unknown"; 
```

有可能创建不是`static`的常量。但是，Java 会在类的每个对象中为该常量分配内存。所以，如果常量真的只有一个值，就应该声明为`static`。

Oracle 已经为类常量定义了命名约定。我们将它们命名为大写字母，单词之间用下划线隔开。然而，并不是所有的`static`和`final`变量都是常量。如果一个物体的状态可以改变，它就不是一个常数:

```
public static final Logger log = LoggerFactory.getLogger(ClassConstants.class);
public static final List<String> contributorGroups = Arrays.asList("contributor", "author");
```

虽然这些是常量引用，但它们引用的是可变对象。

## 3.常量表达式

Java 编译器能够在代码编译期间**计算包含常量变量和某些运算符的表达式**:

```
public static final int MAXIMUM_NUMBER_OF_GUESTS = MAXIMUM_NUMBER_OF_USERS * 10;
public String errorMessage = ClassConstants.DEFAULT_USERNAME + " not allowed here.";
```

像这样的表达式被称为常量表达式，因为编译器会计算它们并生成一个编译时常量。根据 Java 语言规范的定义，以下运算符和表达式可用于常量表达式:

*   一元运算符:+、-、~、！
*   乘法运算符:*、/、%
*   加法运算符:+，-
*   移位操作符:<>，>>>
*   关系运算符:，> =
*   相等运算符:==，！=
*   按位和逻辑运算符:&，^，|
*   条件“与”和条件“或”运算符:&&，||
*   三元条件运算符:？：
*   括号中的表达式，其包含的表达式是常数表达式
*   引用常量变量的简单名称

## 4.编译与运行时常数

如果一个变量的值是在编译时计算的，那么它就是一个编译时常数。另一方面，运行时常数值将在执行期间计算。

### 4.1.编译时常数

如果 Java 变量是基元类型的**或`String`，声明为`final`，在其声明中初始化，并带有常量表达式**，那么它就是编译时常量。

`Strings`是基元类型之上的一个特例，因为它们是不可变的，并且位于 [`String`池](/web/20220526153741/https://www.baeldung.com/java-string-pool)中。因此，应用程序中运行的所有类都可以共享`String`值。

术语编译时常量包括类常量，但也包括使用常量表达式定义的实例和局部变量:

```
public final int maximumLoginAttempts = 5;

public static void main(String[] args) {
    PrintWriter printWriter = System.console().writer();
    printWriter.println(ClassConstants.DEFAULT_USERNAME);

    CompileTimeVariables instance = new CompileTimeVariables();
    printWriter.println(instance.maximumLoginAttempts);

    final String username = "baeldung" + "-" + "user";
    printWriter.println(username);
}
```

只有第一个打印的变量是类常量。然而，所有三个打印出来的变量都是编译时常量。

### 4.2.运行时常数

当程序运行时，运行时常数值不能改变。然而，**每次当我们运行应用程序时，它可以有一个不同的值**:

```
public static void main(String[] args) {
    Console console = System.console();

    final String input = console.readLine();
    console.writer().println(input);

    final double random = Math.random();
    console.writer().println("Number: " + random);
}
```

在我们的示例中打印了两个运行时常量，一个用户定义的值和一个随机生成的值。

## 5.静态代码优化

Java 编译器在编译过程中静态地优化所有编译时常量。因此，**编译器将所有编译时常量引用替换为它们的实际值**。编译器对任何使用编译时常数的类执行这种优化。

让我们看一个引用另一个类的常量的例子:

```
PrintWriter printWriter = System.console().writer();
printWriter.write(ClassConstants.DEFAULT_USERNAME);
```

接下来，我们将编译该类，并观察上面两行代码生成的字节码:

```
LINENUMBER 11 L1
ALOAD 1
LDC "unknown"
INVOKEVIRTUAL java/io/PrintWriter.write (Ljava/lang/String;)V
```

请注意，编译器用变量的实际值替换了变量引用。因此，为了改变编译时常量，我们需要重新编译所有使用它的类。否则，将继续使用旧值。

## 6.用例

让我们看一下 Java 中编译时常量的两个常见用例。

### 6.1.交换语句

在定义 switch 语句的情况时，我们需要遵守 Java 语言规范中定义的规则:

*   switch 语句的 case 标签需要常量表达式或枚举常量的值
*   与 switch 语句关联的 case 常量表达式中不能有两个具有相同的值

这背后的原因是编译器将 switch 语句编译成字节码`tableswitch` 或`lookupswitch.`，它们要求 case 语句中使用的**值既是编译时常数又是唯一的**:

```
private static final String VALUE_ONE = "value-one"

public static void main(String[] args) {
    final String valueTwo = "value" + "-" + "two";
    switch (args[0]) {
        case VALUE_ONE:
            break;
        case valueTwo:
            break;
        }
}
```

如果我们不在 switch 语句中使用常量值，编译器将抛出一个错误。然而，它将接受一个`final String`或任何其他编译时常数。

### 6.2.释文

Java 中的注释处理发生在[编译时间](/web/20220526153741/https://www.baeldung.com/cs/compile-load-execution-time)。实际上，这意味着**注释参数只能使用编译时常量**来定义:

```
private final String deprecatedDate = "20-02-14";
private final String deprecatedTime = "22:00";

@Deprecated(since = deprecatedDate + " " + deprecatedTime)
public void deprecatedMethod() {}
```

虽然在这种情况下使用类常量更常见，但编译器允许这样实现，因为它将值识别为不可变的常量。

## 7.结论

在本文中，我们探讨了 Java 中的术语编译时常数。我们看到术语**包括原始类型的类、实例和局部变量或`String`，声明为`final`，在其声明中初始化，并用常量表达式**定义。

在示例中，我们看到了编译时常数和运行时常数之间的差异。我们还看到编译器使用编译时常量来执行静态代码优化。

最后，我们看了 switch 语句和 Java 注释中编译时常量的用法。

和往常一样，源代码可以在 GitHub 上的[处获得。](https://web.archive.org/web/20220526153741/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-lang-4)