# Java Switch 语句

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-switch>

## 1。概述

在本教程中，我们将学习什么是`switch`语句以及如何使用它。

**`switch`语句允许我们替换几个嵌套的 `if-else`结构，从而提高代码的可读性。**

随着时间的推移而演变。增加了新的支持类型，特别是在 Java 5 和 7 中。此外，它还在继续发展，Java 12 中可能会引入`switch`表达式。

下面我们将给出一些代码示例来演示`switch`语句的使用、`break`语句的作用、`switch`参数/ `case`值的要求以及`switch`语句中`String`的比较。

让我们继续看这个例子。

## 2。使用示例

假设我们有以下嵌套的 `if-else`语句:

```java
public String exampleOfIF(String animal) {
    String result;
    if (animal.equals("DOG") || animal.equals("CAT")) {
        result = "domestic animal";
    } else if (animal.equals("TIGER")) {
        result = "wild animal";
    } else {
        result = "unknown animal";
    }
    return result;
}
```

上面的代码看起来不太好，很难维护和推理。

为了提高可读性，我们可以使用一个`switch`语句:

```java
public String exampleOfSwitch(String animal) {
    String result;
    switch (animal) {
        case "DOG":
            result = "domestic animal"; 
            break;
        case "CAT":
            result = "domestic animal";
            break;
        case "TIGER":
            result = "wild animal";
            break;
        default:
            result = "unknown animal";
            break;
    }
    return result;
}
```

我们将`switch`参数`animal`与几个`case`值进行比较。如果`case`值都不等于参数，则执行`default`标签下的块。

**简单来说，`break`语句用于退出`switch`语句。**

## 3。`break`声明

尽管现实生活中的大多数`switch`语句都暗示只应该执行`case`块中的一个，但是在块完成后，要退出一个`switch`，就必须使用 `break`语句。

**如果我们忘记写一个`break`，下面的块将被执行。**

为了演示这一点，让我们省略`break`语句，并将每个块的输出添加到控制台:

```java
public String forgetBreakInSwitch(String animal) {
    switch (animal) {
    case "DOG":
        System.out.println("domestic animal");
    default:
        System.out.println("unknown animal");
    }
}
```

让我们执行代码`forgetBreakInSwitch` ( `“DOG”)`)并检查输出，以证明所有的块都被执行了:

```java
domestic animal
unknown animal
```

所以，我们应该小心，在每个块的末尾添加`break`语句，除非需要传递到下一个标签下的代码。

唯一不需要`break`的代码块是最后一个，但是在最后一个代码块中添加一个`break`会使代码不容易出错。

当我们希望对几个 case 语句执行相同的代码时，我们也可以利用这种行为来省略`break`。

让我们将前两种情况组合在一起，重写上一节中的示例:

```java
public String exampleOfSwitch(String animal) {
    String result;
    switch (animal) {
        case "DOG":
        case "CAT":
            result = "domestic animal";
            break;
        case "TIGER":
            result = "wild animal";
            break;
        default:
            result = "unknown animal";
            break;
    }
    return result;
}
```

## 4。`switch`自变量和`case`值

现在让我们讨论允许的`switch`参数和`case`值的类型、对它们的要求以及`switch`语句如何处理字符串。

### 4.1。数据类型

我们不能在`switch`语句中比较所有类型的对象和原语。**一个`switch`只能处理四个原语和它们的包装器，以及`enum type `和`String`类**:

*   `byte`和`Byte`
*   `short`和`Short`
*   `int`和`Integer`
*   `char`和`Character`
*   `enum`
*   `String`

从 Java 7 开始，`switch`语句中提供了`String `类型。

`enum` type 是在 Java 5 中引入的，从那以后就可以在`switch`语句中使用了。

从 Java 5 开始，包装类也已经可用。

当然，`switch`参数和`case`值应该是同一类型。

### 4.2.**无`null`值**

**我们不能将`null`值作为参数传递给`switch`语句。**

如果我们这样做，程序将抛出`NullPointerException`，使用我们的第一个`switch `例子:

```java
@Test(expected=NullPointerException.class)
public void whenSwitchAgumentIsNull_thenNullPointerException() {
    String animal = null;
    Assert.assertEquals("domestic animal", s.exampleOfSwitch(animal));
}
```

当然，我们也不能将`null`作为一个值传递给`switch`语句的`case`标签。如果这样做，代码将无法编译。

### 4.3。`Case`作为编译时常量的值

如果我们试图用变量 dog 替换`DOG`的 case 值，代码将不会编译，直到我们将`dog`变量标记为`final`:

```java
final String dog="DOG";
String cat="CAT";

switch (animal) {
case dog: //compiles
    result = "domestic animal";
case cat: //does not compile
    result = "feline"
}
```

### 4.4。`String`对比

如果一个`switch`语句使用相等操作符来比较字符串，我们就不能正确地比较用`new`操作符创建的`String`参数和`String` case 值。

幸运的是， **`switch`操作符使用了幕后的`equals()`方法。**

让我们来演示一下:

```java
@Test
public void whenCompareStrings_thenByEqual() {
    String animal = new String("DOG");
    assertEquals("domestic animal", s.exampleOfSwitch(animal));
}
```

## 5。`switch`表情

[JDK 13](https://web.archive.org/web/20220910004941/https://openjdk.java.net/jeps/354) 现已上市，带来了在 [JDK 12](https://web.archive.org/web/20220910004941/https://openjdk.java.net/jeps/325) 中首次引入的新功能的改进版本:`switch`表情。

**为了启用它，我们需要将`–enable-preview`传递给编译器。**

### 5.1.新的`switch`表情

让我们看看新的`switch`表达式在月份之间切换时是什么样子的:

```java
var result = switch(month) {
    case JANUARY, JUNE, JULY -> 3;
    case FEBRUARY, SEPTEMBER, OCTOBER, NOVEMBER, DECEMBER -> 1;
    case MARCH, MAY, APRIL, AUGUST -> 2;
    default -> 0; 
}; 
```

发送一个像`Month.JUNE`这样的值会将`result` 设置为`3`。

注意，新的语法使用了`->` 操作符，而不是我们习惯的使用`switch`语句的冒号。此外，没有`break` 关键字:`switch`表达式没有通过`case` s

另一个补充是，我们现在可以有逗号分隔的标准。

### 5.2.`yield` 关键字

更进一步，通过使用代码块，可以获得对表达式右侧发生的事情的细粒度控制。

在这种情况下，我们需要使用关键字`yield`:

```java
var result = switch (month) {
    case JANUARY, JUNE, JULY -> 3;
    case FEBRUARY, SEPTEMBER, OCTOBER, NOVEMBER, DECEMBER -> 1;
    case MARCH, MAY, APRIL, AUGUST -> {
        int monthLength = month.toString().length();
        yield monthLength * 4;
    }
    default -> 0;
};
```

虽然我们的例子有点武断，但重点是我们在这里可以接触到更多的 Java 语言。

### 5.3.返回到`switch`表达式中

由于`switch`语句和`switch`表达式之间的区别，**可以从`switch`语句内部进行`return`，但是不允许从`switch`表达式内部进行。**

以下示例完全有效，并且可以编译:

```java
switch (month) {
    case JANUARY, JUNE, JULY -> { return 3; }
    default -> { return 0; }
}
```

但是，下面的代码将无法编译，因为我们试图在封闭的 switch 表达式之外执行`return`:

```java
var result = switch (month) {
    case JANUARY, JUNE, JULY -> { return 3; }
    default -> { return 0; }
};
```

### 5.4.耗尽度

当使用`switch`语句时，**是否涵盖所有情况并不重要。**

例如，下面的代码完全有效，并且可以编译:

```java
switch (month) { 
    case JANUARY, JUNE, JULY -> 3; 
    case FEBRUARY, SEPTEMBER -> 1;
}
```

然而对于`switch`表达式，编译器坚持认为**涵盖了所有可能的情况。**

以下代码片段无法编译，因为没有默认情况，也没有涵盖所有可能的情况:

```java
var result = switch (month) {
    case JANUARY, JUNE, JULY -> 3;
    case FEBRUARY, SEPTEMBER -> 1;
}
```

然而，当所有可能的情况都包括在内时,`switch`表达式将有效:

```java
var result = switch (month) {
    case JANUARY, JUNE, JULY -> 3;
    case FEBRUARY, SEPTEMBER, OCTOBER, NOVEMBER, DECEMBER -> 1;
    case MARCH, MAY, APRIL, AUGUST -> 2;
}
```

请注意，上面的代码片段没有`default`案例。只要覆盖了所有情况，`switch`表达式就有效。

## 6。结论

在本文中，我们讨论了在 Java 中使用`switch`语句的微妙之处。我们可以根据可读性和比较值的类型来决定是否使用`switch `。

当我们在一个预定义的集合中有有限数量的选项时(例如，一周中的几天)，switch 语句是一个很好的选择。

否则，我们必须在每次添加或删除新值时修改代码，这可能是不可行的。对于这些情况，我们应该考虑其他方法，如[多态](/web/20220910004941/https://www.baeldung.com/java-polymorphism)或其他设计模式，如[命令](/web/20220910004941/https://www.baeldung.com/java-command-pattern)。

和往常一样，完整的 [JDK 8 代码](https://web.archive.org/web/20220910004941/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-lang-syntax)和 [JDK 13 代码](https://web.archive.org/web/20220910004941/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-13)可以在 GitHub 上获得。