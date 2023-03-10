# Java 中的方法

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-methods>

## 1.介绍

在 Java 中，[方法](https://web.archive.org/web/20220630004458/https://docs.oracle.com/javase/tutorial/java/javaOO/methods.html)是我们定义应用程序业务逻辑的地方。它们定义了包含在对象中的数据之间的交互。

在本教程中，**我们将介绍 Java 方法的语法，方法签名的定义，以及如何调用和重载方法**。

## 2.方法语法

首先，一个方法由六个部分组成:

*   可选地，我们可以指定从哪个代码可以访问该方法
*   `Return type:`该方法返回的值的类型，如果有的话
*   我们给这个方法起的名字
*   `Parameter list:`可选的逗号分隔的方法输入列表
*   方法可以抛出的异常的可选列表
*   `Body:`逻辑的定义(可以为空)

让我们看一个例子:

[![method structure 3](img/836c3c975f71f27a8c8b73e8ed44829d.png)](/web/20220630004458/https://www.baeldung.com/wp-content/uploads/2019/09/method-structure-3-1024x131.png)

让我们仔细看看 Java 方法的这六个部分。

### 2.1.访问修饰符

[访问修饰符](/web/20220630004458/https://www.baeldung.com/java-access-modifiers)允许我们指定哪些对象可以访问该方法。有四种可能的访问修饰符:`public, protected, private`和 default(也称为`package-private`)。

方法**也可以在访问修饰符之前或之后包含`static`关键字**。这意味着该方法属于类，而不属于实例，因此，我们可以在不创建类实例的情况下调用该方法。不带`static`关键字的方法被称为实例方法，只能在类的实例上调用。

就性能而言，静态方法只需加载到内存中一次——在类加载期间——因此更节省内存。

### 2.2.返回类型

方法可以将数据返回到调用它们的代码中。**一个方法可以返回一个原始值或者一个对象引用，或者如果我们使用`void`关键字作为返回类型，它可以不返回任何东西**。

让我们看一个`void `方法的例子:

```java
public void printFullName(String firstName, String lastName) {
    System.out.println(firstName + " " + lastName);
}
```

**如果我们声明一个返回类型，那么我们必须在方法体中指定一个`return`语句。**一旦执行了`return`语句，方法体的执行也就结束了，如果有更多的语句，这些语句将不会被处理。

另一方面，`void`方法不返回值，因此没有`return`语句。

### 2.3.方法标识符

方法标识符是我们分配给方法规范的名称。使用信息性和描述性的名称是一个好习惯。值得一提的是，方法标识符最多可以有 65536 个字符(尽管名称很长)。

### 2.4.参数列表

我们可以**在其参数列表中为方法**指定输入值，参数列表用圆括号括起来。一个方法可以有 0 到 255 个由逗号分隔的参数。参数可以是一个[对象](/web/20220630004458/https://www.baeldung.com/java-classes-objects)、[一个原语](/web/20220630004458/https://www.baeldung.com/java-primitives-vs-objects)或一个[枚举](/web/20220630004458/https://www.baeldung.com/a-guide-to-java-enums)。我们可以在方法参数级别使用 Java 注释(例如 [Spring 注释`@RequestParam`](/web/20220630004458/https://www.baeldung.com/spring-request-param) )。

### 2.5.例外清单

我们可以通过使用`throws`子句来指定一个方法抛出哪些异常。在[检查异常的情况下，](/web/20220630004458/https://www.baeldung.com/java-checked-unchecked-exceptions)要么我们必须将代码放在[的`try-catch`子句](/web/20220630004458/https://www.baeldung.com/java-exceptions)中，要么我们必须在方法签名中提供一个`throws`子句。

因此，让我们来看一下前面方法的一个更复杂的变体，它抛出一个检查过的异常:

```java
public void writeName(String name) throws IOException {
    PrintWriter out = new PrintWriter(new FileWriter("OutFile.txt"));
    out.println("Name: " + name);
    out.close();
}
```

### 2.6.方法体

**Java 方法的最后一部分是方法体，它包含了我们想要执行的逻辑。**在方法体中，我们可以想写多少行代码就写多少行——或者在`static`方法中根本不用写。如果我们的方法声明了一个返回类型，那么方法体必须包含一个返回语句。

## 3.方法签名

根据其定义，方法签名只由两部分组成— **方法名和参数列表**。

所以，让我们写一个简单的方法:

```java
public String getName(String firstName, String lastName) {
  return firstName + " " + middleName + " " + lastName;
}
```

这个方法的签名是`getName(String firstName, String lastName)`。

方法标识符可以是任何标识符。然而，如果我们遵循常见的 Java 编码约定，方法标识符应该是小写的动词，后面可以跟形容词和/或名词。

## 4.调用方法

现在，让我们探索如何用 Java 调用方法。根据前面的例子，让我们假设这些方法包含在一个名为`PersonName`的 [Java 类](/web/20220630004458/https://www.baeldung.com/java-classes-objects)中:

```java
public class PersonName {
  public String getName(String firstName, String lastName) {
    return firstName + " " + middleName + " " + lastName;
  }
}
```

由于我们的`getName`方法是一个实例方法而不是一个`static`方法，为了调用方法`getName`，我们需要**创建一个类** `PersonName`的实例:

```java
PersonName personName = new PersonName();
String fullName = personName.getName("Alan", "Turing");
```

正如我们所看到的，我们使用创建的对象来调用`getName`方法。

最后，我们来看看**如何调用一个`static`方法**。在静态方法的情况下，我们不需要类实例来进行调用。取而代之的是，我们调用以类名为前缀的方法。

让我们使用前一个示例的变体来演示:

```java
public class PersonName {
  public static String getName(String firstName, String lastName) {
    return firstName + " " + middleName + " " + lastName;
  }
}
```

在这种情况下，方法调用是:

```java
String fullName = PersonName.getName("Alan", "Turing");
```

## 5.方法重载

Java 允许我们拥有**两个或更多具有相同标识符但不同参数列表的方法——不同的方法签名**。在这种情况下，我们说**方法是重载的**。让我们举个例子:

```java
public String getName(String firstName, String lastName) {
  return getName(firstName, "", lastName);
}

public String getName(String firstName, String middleName, String lastName) {
  if (!middleName.isEqualsTo("")) {
    return firstName + " " + lastName;
  }
  return firstName + " " + middleName + " " + lastName;
}
```

方法重载对于例子中的情况很有用，我们可以用一个方法实现相同功能的简化版本。

最后，一个好的设计习惯是确保[重载方法](/web/20220630004458/https://www.baeldung.com/java-method-overload-override)以相似的方式运行。否则，如果具有相同标识符的方法以不同的方式运行，代码将会混乱。

## 6.结论

在本教程中，我们探索了在 Java 中指定方法时涉及的 Java 语法部分。

特别是，我们检查了访问修饰符、返回类型、方法标识符、参数列表、异常列表和方法体。然后我们看到了方法签名的定义，如何调用方法，如何重载方法。

像往常一样，这里看到的代码可以在 GitHub 上找到[。](https://web.archive.org/web/20220630004458/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-lang-oop-methods)