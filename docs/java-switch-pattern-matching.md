# 开关的模式匹配

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-switch-pattern-matching>

## 1.概观

Java SE 17 版本引入了针对`switch`表达式和语句的模式匹配( [JEP 406](https://web.archive.org/web/20220910004922/https://openjdk.java.net/jeps/406) )作为预览特性。模式匹配**在为`switch`案例**定义条件时为我们提供了更多的灵活性。

除了现在可以包含模式的 case 标签之外，选择器表达式不再仅限于几种类型。在模式匹配之前，`switch` cases 只支持需要精确匹配一个常量值的选择器表达式的简单测试。

在本教程中，我们将介绍三种不同的模式类型，它们可以应用在`switch`语句中。我们还将探索一些`switch`细节，比如覆盖所有值、对子类排序以及处理空值。

## 2.交换语句

我们在 Java 中使用 [`switch`](/web/20220910004922/https://www.baeldung.com/java-switch) 将控制转移到几个预定义的 case 语句之一。哪个语句被选中取决于`switch`选择器表达式的值。

在 Java 的早期版本中，**选择器表达式必须是数字、字符串或常量**。此外，案例标签只能包含常量:

```java
final String b = "B";
switch (args[0]) {
    case "A" -> System.out.println("Parameter is A");
    case b -> System.out.println("Parameter is b");
    default -> System.out.println("Parameter is unknown");
};
```

在我们的例子中，如果变量 b 不是`final`，编译器将抛出常量表达式要求错误。

## 3.模式匹配

一般来说，模式匹配最初是作为 Java SE 14 中的预览特性引入的。

它仅限于模式的一种形式——类型模式。典型的模式由类型名和将结果绑定到的变量组成。

**将类型模式应用于`[instanceof](/web/20220910004922/https://www.baeldung.com/java-instanceof#:~:text=instanceof%20is%20a%20binary%20operator,check%20should%20always%20be%20used.)` 操作符简化了类型检查和类型转换**。此外，它使我们能够将两者结合成一个单一的表达式:

```java
if (o instanceof String s) {
    System.out.printf("Object is a string %s", s);
} else if (o instanceof Number n) {
    System.out.printf("Object is a number %n", n);
}
```

这种内置的语言增强功能帮助我们编写更少的代码，同时增强可读性。

## 4.开关模式

[`instanceof`](/web/20220910004922/https://www.baeldung.com/java-pattern-matching-instanceof)的模式匹配成为 Java SE 16 中的永久特性。

**在 Java 17 中，模式匹配的应用现在也扩展到了`switch`表达式**。

然而，它仍然是一个[预览功能](/web/20220910004922/https://www.baeldung.com/java-preview-features)，所以我们需要启用预览来使用它:

```java
java --enable-preview --source 17 PatternMatching.java
```

### 4.1.类型模式

让我们看看如何在`switch`语句中应用类型模式和`instanceof` 操作符。

例如，我们将创建一个使用`if-else`语句将不同类型转换为`double` 的方法。如果不支持该类型，我们的方法将简单地返回零:

```java
static double getDoubleUsingIf(Object o) {
    double result;
    if (o instanceof Integer) {
        result = ((Integer) o).doubleValue();
    } else if (o instanceof Float) {
        result = ((Float) o).doubleValue();
    } else if (o instanceof String) {
        result = Double.parseDouble(((String) o));
    } else {
        result = 0d;
    }
    return result;
}
```

我们可以使用`switch`中的类型模式用更少的代码解决同样的问题:

```java
static double getDoubleUsingSwitch(Object o) {
    return switch (o) {
        case Integer i -> i.doubleValue();
        case Float f -> f.doubleValue();
        case String s -> Double.parseDouble(s);
        default -> 0d;
    };
}
```

在 Java 的早期版本中，选择器表达式仅限于几种类型。然而，对于类型模式，**`switch`选择器表达式可以是任何类型。**

### 4.2.防护图案

类型模式帮助我们基于特定的类型转移控制。然而，有时，我们也需要对传递的值执行额外的检查。

例如，我们可以使用一个`if`语句来检查一个`String`的长度:

```java
static double getDoubleValueUsingIf(Object o) {
    return switch (o) {
        case String s -> {
            if (s.length() > 0) {
                yield Double.parseDouble(s);
            } else {
                yield 0d;
            }
        }
        default -> 0d;
    };
}
```

我们可以使用保护模式来解决同样的问题。他们使用模式和布尔表达式的组合:

```java
static double getDoubleValueUsingGuardedPatterns(Object o) {
    return switch (o) {
        case String s && s.length() > 0 -> Double.parseDouble(s);
        default -> 0d;
    };
}
```

**保护模式使我们能够在`switch`语句中避免额外的`if`条件。相反，我们可以** **将我们的条件逻辑移到案例标签**。

### 4.3.圆括号模式

除了 cases 标签中的条件逻辑之外，**带括号的模式使我们能够对它们进行分组**。

在执行附加检查时，我们可以在布尔表达式中简单地使用括号:

```java
static double getDoubleValueUsingParenthesizedPatterns(Object o) {
    return switch (o) {
        case String s && s.length() > 0 && !(s.contains("#") || s.contains("@")) -> Double.parseDouble(s);
        default -> 0d;
    };
}
```

通过使用括号，我们可以避免额外的 `if-else`语句。

## 5.开关细节

现在让我们看看在`switch`中使用模式匹配时要考虑的几个具体情况。

### 5.1.涵盖所有价值

在`switch`中使用模式匹配时，Java **编译器会检查类型覆盖**。

让我们考虑一个示例`switch`条件，它接受任何对象，但仅涵盖`String`情况:

```java
static double getDoubleUsingSwitch(Object o) {
    return switch (o) {
        case String s -> Double.parseDouble(s);
    };
}
```

我们的示例将导致以下编译错误:

```java
[ERROR] Failed to execute goal ... on project core-java-17: Compilation failure
[ERROR] /D:/Projects/.../HandlingNullValuesUnitTest.java:[10,16] the switch expression does not cover all possible input values
```

这是因为 `switch` **案例标签需要包含选择器表达式**的类型。

也可以使用`default`案例标签来代替特定的选择器类型。

### 5.2.子类排序

当在`switch`中使用带有模式匹配的子类时，案例的**顺序很重要**。

让我们考虑一个例子，其中`String`案例在`CharSequence`案例之后。

```java
static double getDoubleUsingSwitch(Object o) {
    return switch (o) {
        case CharSequence c -> Double.parseDouble(c.toString());
        case String s -> Double.parseDouble(s);
        default -> 0d;
    };
}
```

由于`String`是`CharSequence,` 的子类，我们的例子将导致以下编译错误:

```java
[ERROR] Failed to execute goal ... on project core-java-17: Compilation failure
[ERROR] /D:/Projects/.../HandlingNullValuesUnitTest.java:[12,18] this case label is dominated by a preceding case label
```

这个错误背后的原因是**没有机会执行第二个案例**，因为传递给该方法的任何字符串对象都将在第一个案例中处理。

### 5.3.处理空值

在 Java 的早期版本中，每次将一个`null`值传递给一个`switch`语句都会导致一个`NullPointerException`。

然而，使用类型模式，现在可以将空检查作为单独的 case 标签应用于**:**

```java
static double getDoubleUsingSwitch(Object o) {
    return switch (o) {
        case String s -> Double.parseDouble(s);
        case null -> 0d;
        default -> 0d;
    };
}
```

如果没有特定于空值的案例标签，总类型的**模式标签将**匹配空值:

```java
static double getDoubleUsingSwitchTotalType(Object o) {
    return switch (o) {
        case String s -> Double.parseDouble(s);
        case Object ob -> 0d;
    };
}
```

我们应该注意到一个`switch`表达式不能同时有一个`null` case 和一个 total 类型 case。

这样的`switch`语句将导致如下编译错误:

```java
[ERROR] Failed to execute goal ... on project core-java-17: Compilation failure
[ERROR] /D:/Projects/.../HandlingNullValuesUnitTest.java:[14,13] switch has both a total pattern and a default label
```

最后，使用模式匹配的`switch`语句仍然可以抛出一个`NullPointerException`。

然而，只有当`switch`块没有空匹配的 case 标签时，它才能这样做。

## 6.结论

在本文中，**我们探索了`switch`表达式和语句的模式匹配，这是`Java SE 17`** `.`中的一个预览特性，我们看到，通过在 case 标签中使用模式，选择是由模式匹配而不是简单的相等性检查决定的。

在示例中，我们涵盖了可以在`switch`语句中应用的三种不同的模式类型。最后，我们探讨了几个特定的案例，包括覆盖所有值、对子类排序和处理空值。

和往常一样，完整的源代码可以在 GitHub 上找到[。](https://web.archive.org/web/20220910004922/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-17)