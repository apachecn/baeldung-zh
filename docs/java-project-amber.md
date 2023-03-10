# 琥珀计划简介

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-project-amber>

## 1。什么是琥珀项目

**[Amber 项目](https://web.archive.org/web/20220926192256/https://openjdk.java.net/projects/amber/)是 Java 和 OpenJDK 开发人员目前的一项计划，旨在对 JDK 进行一些微小但必要的更改，使开发过程变得更好**。这项工作自 2017 年以来一直在进行，并已在 Java 10 和 11 中进行了一些更改，其他更改计划包含在 Java 12 中，还将在未来的版本中推出更多更改。

这些更新都以 [JEPs](https://web.archive.org/web/20220926192256/https://en.wikipedia.org/wiki/JDK_Enhancement_Proposal) 的形式打包——JDK 增强提案计划。

## 2。交付的更新

到目前为止，琥珀计划已经成功地将一些变化交付给了目前发布的 JDK 版本——[JEP-286](https://web.archive.org/web/20220926192256/https://openjdk.java.net/jeps/286)和 [JEP-323](https://web.archive.org/web/20220926192256/https://openjdk.java.net/jeps/323) 。

### 2.1。局部变量类型推理

Java 7 引入了[菱形操作符](/web/20220926192256/https://www.baeldung.com/java-diamond-operator)，以使泛型更容易与一起工作。这个特性意味着，当我们定义变量时，不再需要在同一个语句中多次编写通用信息:

```java
List<String> strings = new ArrayList<String>(); // Java 6
List<String> strings = new ArrayList<>(); // Java 7
```

**Java 10 包含了 JEP-286 的全部工作，允许我们的 Java 代码定义局部变量，而不需要在编译器已经可用的地方复制类型信息**。这在更广泛的社区中被称为`var`关键字，它为 Java 带来了许多其他语言中可用的类似功能。

有了这项工作，**每当我们定义一个局部变量时，我们可以使用`var`关键字代替完整的类型定义**，编译器将自动计算出要使用的正确类型信息:

```java
var strings = new ArrayList<String>();
```

**在上面，变量`strings`被确定为类型`ArrayList<String>()`** ，但是不需要在同一行上复制信息。

无论值是如何确定的，我们都可以在使用局部变量的任何地方使用它。这包括返回类型和表达式，以及如上所述的简单赋值。

单词`var`是一个特例，因为它不是一个保留字。相反，它是一个特殊的类型名。这意味着这个词可以用于代码的其他部分，包括变量名。强烈建议不要这样做，以免混淆。

只有当我们提供一个实际类型作为声明的一部分时，我们才能使用局部类型推断。当值显式为`null,` 时，当根本没有提供值时，或者当提供的值不能确定确切的类型时，它被故意设计为不起作用-例如，Lambda 定义:

```java
var unknownType; // No value provided to infer type from
var nullType = null; // Explicit value provided but it's null
var lambdaType = () -> System.out.println("Lambda"); // Lambda without defining the interface
```

然而，**的值可以是`null`，如果它是某个其他调用**的返回值，因为调用本身提供了类型信息:

```java
Optional<String> name = Optional.empty();
var nullName = name.orElse(null);
```

在这种情况下，`nullName`将推断出类型`String`，因为这就是`name.orElse()`的返回类型。

**以这种方式定义的变量可以像任何其他变量**一样拥有任何其他修饰符——例如，`transitive, synchronized,`和`final`。

### 2.2。Lambdas 的局部变量类型推理

上述工作允许我们在不需要复制类型信息的情况下声明局部变量。然而，这对参数列表不起作用，尤其是对 lambda 函数的参数不起作用，这似乎令人惊讶。

在 Java 10 中，我们可以用两种方式定义 Lambda 函数——要么显式声明类型，要么完全省略它们:

```java
names.stream()
  .filter(String name -> name.length() > 5)
  .map(name -> name.toUpperCase());
```

这里，第二行有一个显式的类型声明——`String`——而第三行完全省略了它，编译器会计算出正确的类型。**我们不能在这里使用`var`类型**。

**Java 11 允许这种情况发生**，所以我们可以改为写:

```java
names.stream()
  .filter(var name -> name.length() > 5)
  .map(var name -> name.toUpperCase());
```

**这与我们代码**中其他地方对`var`类型的使用是一致的。

Lambdas 总是限制我们要么对每个参数使用完整的类型名，要么不使用完整的类型名。这没有改变，并且**`var`的使用必须用于每个参数或者不用于任何参数**:

```java
numbers.stream()
    .reduce(0, (var a, var b) -> a + b); // Valid

numbers.stream()
    .reduce(0, (var a, b) -> a + b); // Invalid

numbers.stream()
    .reduce(0, (var a, int b) -> a + b); // Invalid
```

这里，第一个例子完全有效——因为两个 lambda 参数都使用了`var`。**第二个和第三个是非法的，因为只有一个参数使用了** `**var**`，尽管在第三种情况下我们也有一个显式的类型名。

## 3。即将更新

除了已经发布的 JDK 中的更新，即将发布的 JDK 12 版本还包括一个更新—[JEP-325。](https://web.archive.org/web/20220926192256/https://openjdk.java.net/jeps/325)

### 3.1。开关表达式

**JEP-325 支持简化`switch`语句的工作方式，并允许它们被用作表达式**，以进一步简化使用它们的代码。

目前，`switch`语句的工作方式非常类似于 C 或 C++等语言中的语句。**这些变化使得它更类似于 Kotlin 中的`when`语句或 Scala** 中的`match`语句。

有了这些变化，**定义 switch 语句的语法看起来类似于 lambdas** 的语法，只是使用了`->`符号。这位于大小写匹配和要执行的代码之间:

```java
switch (month) {
    case FEBRUARY -> System.out.println(28);
    case APRIL -> System.out.println(30);
    case JUNE -> System.out.println(30);
    case SEPTEMBER -> System.out.println(30);
    case NOVEMBER -> System.out.println(30);
    default -> System.out.println(31);
}
```

**注意，`break`关键字是不需要的，更重要的是，我们不能在这里使用**。这自然意味着每一个匹配都是不同的，失败是不可避免的。相反，我们可以在需要的时候继续使用旧的样式。

**箭头的右边必须是表达式、块或 throws 语句**。其他的都是错误。这也解决了在 switch 语句中定义变量的问题——这只能发生在一个块中，这意味着它们会自动作用于该块:

```java
switch (month) {
    case FEBRUARY -> {
        int days = 28;
    }
    case APRIL -> {
        int days = 30;
    }
    ....
}
```

在旧风格的 switch 语句中，这将是一个错误，因为重复的变量 `**days**. `使用块的要求避免了这一点。

**箭头的左边可以是任意数量的逗号分隔值** **。**这是为了实现一些与 fallthrough 相同的功能，但仅限于整场比赛，绝不是偶然的:

```java
switch (month) {
    case FEBRUARY -> System.out.println(28);
    case APRIL, JUNE, SEPTEMBER, NOVEMBER -> System.out.println(30);
    default -> System.out.println(31);
}
```

到目前为止，所有这些都可以通过当前的方式实现，即`switch`语句的工作方式，并使其更加整洁。然而，**这次更新也带来了使用`switch`语句作为表达式**的能力。对于 Java 来说，这是一个重大的变化，但是这与许多其他语言——包括其他 JVM 语言——开始工作是一致的。

**这允许`switch`表达式解析为一个值，然后在其他语句**中使用该值——例如，赋值:

```java
final var days = switch (month) {
    case FEBRUARY -> 28;
    case APRIL, JUNE, SEPTEMBER, NOVEMBER -> 30;
    default -> 31;
}
```

这里，我们使用一个`switch`表达式来生成一个数字，然后我们将这个数字直接赋给一个变量。

**以前，这只能通过将变量`days`定义为`null`，然后在`switch`用例**中为其赋值来实现。这意味着`days`不可能是最终的，如果我们错过了一个案例，它可能会被取消分配。

## 4。即将到来的变化

到目前为止，所有这些变化要么已经可用，要么将在即将到来的版本。作为琥珀项目的一部分，有一些提议的变更尚未发布。

### 4.1。原始字符串文字

**目前，Java 只有一种方法来定义字符串——用双引号将内容括起来**。这很容易使用，但是在更复杂的情况下会遇到问题。

具体来说，**很难写出包含某些字符**的字符串——包括但不限于:换行符、双引号和反斜杠字符。这在文件路径和正则表达式中尤其成问题，因为这些字符可能比典型字符更常见。

**[JEP-326](https://web.archive.org/web/20220926192256/https://openjdk.java.net/jeps/326) 引入了一种新的字符串文字类型，称为原始字符串文字**。这些字符用反勾号而不是双引号括起来，可以包含任何字符。

这意味着可以编写跨多行的字符串，以及包含引号或反斜杠的字符串，而无需对它们进行转义。因此，它们变得更容易阅读。

例如:

```java
// File system path
"C:\\Dev\\file.txt"
`C:\Dev\file.txt`

// Regex
"\\d+\\.\\d\\d"
`\d+\.\d\d`

// Multi-Line
"Hello\nWorld"
`Hello
World`
```

在所有这三种情况下，**更容易看到带有反勾号的版本中发生了什么，这也大大减少了键入**的错误。

**新的原始字符串文字也允许我们在不增加复杂性的情况下包含反勾号。用于开始和结束字符串的反勾号的数量可以根据需要任意长，不必只有一个反勾号。只有当我们达到相同长度的反斜线时，字符串才会结束。比如说:**

```java
``This string allows a single "`" because it's wrapped in two backticks``
```

这些允许我们准确地输入字符串，而不需要特殊的序列来使某些字符工作。

### 4.2。λ剩菜

JEP-302 对 lambdas 的工作方式进行了一些小的改进。

主要的变化是参数的处理方式。首先，**这个变化引入了对未使用的参数使用下划线的能力，这样我们就不会生成不需要的名字**。这在以前是可能的，但只适用于单个参数，因为下划线是有效的名称。

Java 8 引入了一个变化，因此使用下划线作为名称是一个警告。Java 9 随后把这变成了一个错误，阻止我们使用它们。这个即将到来的变化允许他们为 lambda 参数而不引起任何冲突。例如，这将允许以下代码:

```java
jdbcTemplate.queryForObject("SELECT * FROM users WHERE user_id = 1", (rs, _) -> parseUser(rs))
```

在这种增强下，**我们用两个参数定义了λ，但是只有第一个参数被命名为**。第二个是不可访问的，但是同样，我们这样写是因为我们不需要使用它。

这个增强的另一个主要变化是允许 lambda 参数隐藏当前上下文中的名字。目前这是不允许的，这会导致我们编写一些不太理想的代码。例如:

```java
String key = computeSomeKey();
map.computeIfAbsent(key, key2 -> key2.length());
```

**没有真正的需要，除了编译器，为什么`key `和`key2`不能共用一个名字**。lambda 从不需要引用变量`key`，强迫我们这么做会让代码更难看。

相反，这种增强允许我们以更明显和简单的方式编写它:

```java
String key = computeSomeKey();
map.computeIfAbsent(key, key -> key.length());
```

此外，**当重载方法有 lambda 参数**时，此增强中有一个提议的更改可能会影响重载解析。目前，在一些情况下，由于重载决策的工作规则，这可能会导致不确定性，JEP 可能会对这些规则稍作调整，以避免这种不确定性。

例如**目前编译器认为以下方法是二义性的**:

```java
m(Predicate<String> ps) { ... }
m(Function<String, String> fss) { ... }
```

这两种方法都采用一个 lambda，该 lambda 有一个参数和一个非 void 返回类型。**对开发人员来说，它们显然是不同的——一个返回一个`String`，另一个返回一个`boolean`，但是编译器会将这些视为不明确的**。

这个 JEP 可以解决这个缺点，并允许显式地处理这个重载。

### 4.3。模式匹配

**[JEP-305](https://web.archive.org/web/20220926192256/https://openjdk.java.net/jeps/305) 引入了对我们使用`instanceof`操作符和自动类型强制的改进。**

目前，在 Java 中比较类型时，我们必须使用 `instanceof`运算符来查看值是否属于正确的类型，然后，我们需要将值强制转换为正确的类型:

```java
if (obj instanceof String) {
    String s = (String) obj;
    // use s
}
```

这是可行的，并且很容易理解，但是它比必要的要复杂得多。我们的代码中有一些非常明显的重复，因此，有可能会出现错误。

**这个增强对`instanceof`操作符进行了类似的调整，就像之前在 Java 7** 的`try-with-resources`中所做的一样。随着这一变化，比较、造型和变量声明变成了一条语句:

```java
if (obj instanceof String s) {
    // use s
}
```

**这给了我们一个单一的语句，没有重复，也没有在**中出现错误的风险，但是执行起来和上面的一样。

这也将跨分支正确工作，允许以下工作:

```java
if (obj instanceof String s) {
    // can use s here
} else {
    // can't use s here
}
```

**在适当的情况下，增强功能还可以跨不同的范围正确工作**。如预期的那样，由`instanceof`子句声明的变量将正确地隐藏在其外部定义的变量。不过，这只会发生在适当的区块中:

```java
String s = "Hello";
if (obj instanceof String s) {
    // s refers to obj
} else {
    // s refers to the variable defined before the if statement
}
```

**这也在同一个`if`子句**中起作用，与我们依赖`null`检查的方式相同:

```java
if (obj instanceof String s && s.length() > 5) {
    // s is a String of greater than 5 characters
}
```

**目前，这只是为`if`语句**设计的，但是未来的工作可能会将它扩展到与`switch expressions`一起工作。

### 4.4。简洁的方法体

[**JEP 草案 8209434**](https://web.archive.org/web/20220926192256/https://openjdk.java.net/jeps/8209434) **是一个支持简化方法定义**的提案，其方式类似于 lambda 定义的工作方式。

**现在，我们可以用三种不同的方式定义 Lambda**:用一个体，作为一个单独的表达式，或者作为一个方法引用:

```java
ToIntFunction<String> lenFn = (String s) -> { return s.length(); };
ToIntFunction<String> lenFn = (String s) -> s.length();
ToIntFunction<String> lenFn = String::length;
```

然而，**当涉及到编写实际的类方法体时，我们目前必须完整地写出它们**。

在适用的情况下，本提案也支持这些方法的表达式和方法引用形式。这将有助于保持某些方法比现在简单得多。

例如，getter 方法不需要完整的方法体，但可以替换为一个表达式:

```java
String getName() -> name;
```

同样，我们可以用一个方法引用调用来替换简单地包装其他方法的方法，包括通过以下方式传递参数:

```java
int length(String s) = String::length
```

这些将允许在有意义的情况下使用更简单的方法，这意味着它们不太可能在类的其余部分模糊真正的业务逻辑。

请注意，这仍处于草稿状态，因此在交付之前可能会有重大更改。

## 5。增强型枚举

JEP-301 之前被安排为琥珀计划的一部分。**这会给枚举带来一些改进，显式地允许单个枚举元素拥有不同的泛型类型信息**。

例如，它将允许:

```java
enum Primitive<X> {
    INT<Integer>(Integer.class, 0) {
       int mod(int x, int y) { return x % y; }
       int add(int x, int y) { return x + y; }
    },
    FLOAT<Float>(Float.class, 0f)  {
       long add(long x, long y) { return x + y; }
    }, ... ;

    final Class<X> boxClass;
    final X defaultValue;

    Primitive(Class<X> boxClass, X defaultValue) {
       this.boxClass = boxClass;
       this.defaultValue = defaultValue;
    }
}
```

不幸的是，**在 Java 编译器应用程序中进行的这种增强的实验已经证明，它比以前认为的更不可行**。将泛型类型信息添加到枚举元素中，使得无法将这些枚举用作其他类的泛型类型，例如，`EnumSet`。这大大降低了增强的有用性。

因此，**这一改进目前被搁置，直到这些细节得到解决**。

## 6。总结

我们在这里已经讨论了许多不同的特性。其中一些已经推出，另一些也将很快推出，还有更多计划在未来发布。这些如何改善你当前和未来的项目？