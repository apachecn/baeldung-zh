# 在 Java 的 If 条件中使用 Not 运算符

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-using-not-in-if-conditions>

## 1.介绍

在 Java 的 [if-else 语句](/web/20221126215603/https://www.baeldung.com/java-if-else)中，当表达式为`true`时，我们可以采取某个动作，当表达式为`false`时，我们可以选择另一个动作。在本教程中，我们将学习如何使用`not`操作符反转逻辑。

## 2.`if-else S`声明

让我们从一个简单的`if-else` 语句开始:

```
boolean isValid = true;

if (isValid) {
    System.out.println("Valid");
} else {
    System.out.println("Invalid");
}
```

如果我们的程序只需要处理否定的情况呢？我们如何重写上面的例子？

一种选择是简单地删除`if`块中的代码:

```
boolean isValid = true;

if (isValid) {

} else {
    System.out.println("Invalid");
}
```

然而，一个空的`if` 块看起来可能是不完整的代码，并且似乎是一种冗长的只处理负面情况的方式。我们可以试着测试我们的逻辑表达式是否等于`false`:

```
boolean isValid = true;

if (isValid == false) {
    System.out.println("Invalid");
}
```

上面的版本相对容易阅读，尽管如果逻辑表达式更复杂的话可能会更难。不过，Java 为我们提供了另一种选择，即`not`操作符:

```
boolean isValid = true;

if (!isValid) {
    System.out.println("Invalid");
}
```

## 3.`not`操作员

`not`操作符是一个逻辑操作符，在 Java 中用符号`!` 表示。它是一个一元运算符，以布尔值作为操作数。**`not`运算符通过反转(或否定)其操作数**的值来工作。

### 3.1.将`not`运算符应用于布尔值

当应用于布尔值时，`not`运算符将`true`转换为`false`，将`false` 转换为`true`。

例如:

```
System.out.println(!true);   // prints false 
System.out.println(!false);  // prints true 
System.out.println(!!false); // prints false
```

### 3.2.将`not`运算符应用于布尔表达式

由于`not`是一个一元运算符，**当你想要`not`一个表达式的结果时，你需要用括号**将该表达式括起来以得到正确的答案。首先计算括号中的表达式，然后`not`运算符反转其结果。

例如:

```
int count = 2;

System.out.println(!(count > 2));  // prints true
System.out.println(!(count <= 2)); // prints false
```

```
boolean x = true;
boolean y = false;

System.out.println(!(x && y));  // prints true
System.out.println(!(x || y));  // prints false 
```

我们应该注意到，当否定一个表达式时，[德摩根定律](https://web.archive.org/web/20221126215603/https://en.wikipedia.org/wiki/De_Morgan%27s_laws)开始发挥作用。换句话说，表达式中的每一项都被求反，并且运算符被反转。这可以帮助我们简化难以阅读的表达式。

例如:

```
!(x && y) is same as !x || !y
!(x || y) is same as !x && !y
!(a < 3 && b == 10) is same as a >= 3 || b != 10 
```

## 4.一些常见的陷阱

使用`not`操作符有时会损害我们代码的可读性。消极可能比积极更难理解。让我们看一些例子。

### 4.1.双重否定

因为`not`操作符是一个求反操作符，所以将它与具有负名称的变量或函数一起使用会导致代码难以阅读。这与自然语言相似，双重否定通常被认为难以理解。

例如:

```
if (product.isActive()) {...}
```

比更好读

```
if (!product.isNotActive()) {...}
```

虽然我们的 API 可能没有提供`isActive`方法，但是我们可以创建一个来提高可读性。

### 4.2.复杂条件

`not`操作符有时会使已经很复杂的表达式变得更加难以阅读和理解。当这种情况发生时，我们可以通过颠倒条件或提取方法来简化代码。让我们来看一些由`not`操作符变得复杂的条件的例子，以及我们如何通过反转条件来简化它们:

```
if (!true) // Complex
if (false) // Simplified

if (!myDate.onOrAfter(anotherDate)) // Complex 
if (myDate.before(anotherDate))     // Simplified

if (!(a >= b)) // Complex
if (a < b)     // Simplified

if (!(count >= 10 || total >= 1000))  // Complex
if (count < 10 && total < 1000)       // Simplified
```

## 5.结论

在本文中，我们讨论了*而非*运算符，以及它如何与布尔值、表达式和 in `if-else`语句一起使用。

我们还研究了一些常见的陷阱，这些陷阱是由反向编写表达式引起的，以及如何修复它们。

一如既往，本文中使用的示例的源代码 在 GitHub 上有[。](https://web.archive.org/web/20221126215603/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-lang-syntax-2)