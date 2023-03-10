# Java 编译器错误:表达式的非法开始

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-illegal-start-of-expression>

## 1.概观

“表达式的非法开始”是我们在编译时可能会遇到的一个常见错误。

在本教程中，我们将看到说明这个错误的主要原因以及如何修复它的例子。

## 2.缺少花括号

缺少花括号可能会导致“表达式的非法开始”错误。我们先来看一个例子:

```java
package com.baeldung;

public class MissingCurlyBraces {
    public void printSum(int x, int y) {
        System.out.println("Calculation Result:" + calcSum(x, y));

    public int calcSum(int x, int y) {
        return x + y;
    }
}
```

如果我们编译上面的类:

```java
$ javac MissingCurlyBraces.java
MissingCurlyBraces.java:7: error: illegal start of expression
        public int calcSum(int x, int y) {
        ^
MissingCurlyBraces.java:7: error: ';' expected
        public int calcSum(int x, int y) {
   ..... 
```

缺少`printSum()`的右花括号是问题的根本原因。

解决这个问题很简单——给`printSum()`方法添加右花括号:

```java
package com.baeldung;

public class MissingCurlyBraces {
    public void printSum(int x, int y) {
        System.out.println("Calculation Result:" + calcSum(x, y));
    }
    public int calcSum(int x, int y) {
        return x + y;
    }
}
```

在我们进入下一节之前，让我们回顾一下编译器错误。

编译器报告第 7 行导致了“表达式非法开始”错误。事实上，我们知道问题的根源在第 6 行。从这个例子中，我们了解到**有时编译器错误并不指向具有根本原因**的行，我们需要修正前一行的语法。

## 3.方法内部的访问修饰符

在 Java 中，**我们只能在方法或构造函数**内部声明局部变量。我们不能对方法内部的局部变量使用任何[访问修饰符](/web/20220626201502/https://www.baeldung.com/java-access-modifiers)，因为它们的可访问性是由方法范围定义的。

如果我们违反了规则，在方法中使用了访问修饰符，就会出现“表达式非法开始”的错误。

让我们来看看实际情况:

```java
package com.baeldung;

public class AccessModifierInMethod {
    public void printSum(int x, int y) {
        private int sum = x + y; 
        System.out.println("Calculation Result:" + sum);
    }
}
```

如果我们尝试编译上面的代码，我们会看到编译错误:

```java
$ javac AccessModifierInMethod.java 
AccessModifierInMethod.java:5: error: illegal start of expression
        private int sum = x + y;
        ^
1 error
```

移除`private`访问修饰符很容易解决这个问题:

```java
package com.baeldung;

public class AccessModifierInMethod {
    public void printSum(int x, int y) {
        int sum = x + y;
        System.out.println("Calculation Result:" + sum);
    }
}
```

## 4.嵌套方法

一些编程语言，比如 Python，支持嵌套方法。但是，Java 不支持一个方法包含在另一个方法中。

如果我们创建嵌套方法，我们将面临“表达式的非法开始”编译器错误:

```java
package com.baeldung;

public class NestedMethod {
    public void printSum(int x, int y) {
        System.out.println("Calculation Result:" + calcSum(x, y));
        public int calcSum ( int x, int y) {
            return x + y;
        }
    }
} 
```

让我们编译上面的源文件，看看 Java 编译器报告了什么:

```java
$ javac NestedMethod.java
NestedMethod.java:6: error: illegal start of expression
        public int calcSum ( int x, int y) {
        ^
NestedMethod.java:6: error: ';' expected
        public int calcSum ( int x, int y) {
                          ^
NestedMethod.java:6: error: <identifier> expected
        public int calcSum ( int x, int y) {
                                   ^
NestedMethod.java:6: error: not a statement
        public int calcSum ( int x, int y) {
                                        ^
NestedMethod.java:6: error: ';' expected
        public int calcSum ( int x, int y) {
                                         ^
5 errors
```

Java 编译器报告了五个编译错误。在某些情况下，一个错误可能会在编译期间导致多个进一步的错误。

找出根本原因对我们解决问题至关重要。在本例中，第一个“表达式非法开始”错误是根本原因。

我们可以通过将`calcSum() `方法移出`printSum()`方法来快速解决问题:

```java
package com.baeldung;

public class NestedMethod {
    public void printSum(int x, int y) {
        System.out.println("Calculation Result:" + calcSum(x, y));
    }
    public int calcSum ( int x, int y) {
        return x + y;
    }
} 
```

## 5.`char`或`String`不带引号

我们知道`String`文字应该用双引号括起来，而*字符*值应该用单引号括起来。

如果我们忘记用正确的引号把它们括起来，**Java 编译器会把它们当作变量名**。

如果没有声明“变量”,我们可能会看到“找不到符号”错误。

然而，**如果我们忘记用双引号将一个不是[有效 Java 变量名](https://web.archive.org/web/20220626201502/https://docs.oracle.com/javase/tutorial/java/nutsandbolts/variables.html)的`String` 括起来，Java 编译器就会报告“表达式非法开始”错误**。

让我们通过一个例子来看一下:

```java
package com.baeldung;

public class ForgetQuoting {
    public int calcSumOnly(int x, int y, String operation) {
        if (operation.equals(+)) {
            return x + y;
        }
        throw new UnsupportedOperationException("operation is not supported:" + operation);
    }
} 
```

我们忘了在对`equals`方法的调用中引用`String` `+`，而`+`显然不是一个有效的 Java 变量名。

现在，让我们试着编译它:

```java
$ javac ForgetQuoting.java 
ForgetQuoting.java:5: error: illegal start of expression
        if (operation.equals(+)) {
                              ^
1 error 
```

这个问题的解决方案很简单——用双引号将`String`文字括起来:

```java
package com.baeldung;

public class ForgetQuoting {
    public int calcSumOnly(int x, int y, String operation) {
        if (operation.equals("+")) {
            return x + y;
        }
        throw new UnsupportedOperationException("operation is not supported:" + operation);
    }
} 
```

## 6.结论

在这篇短文中，我们讨论了五种不同的场景，它们会引发“表达式非法开始”错误。

大多数情况下，当开发 Java 应用程序时，我们会使用 IDE 在检测到错误时发出警告。这些不错的 IDE 特性可以帮助我们避免这种错误。

然而，我们仍然会不时地遇到错误。因此，对错误的良好理解将有助于我们快速定位和修复错误。