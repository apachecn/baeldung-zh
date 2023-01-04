# Java 运算符

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-operators>

## 1.概观

运算符是任何编程语言的基本构件。我们使用运算符对值和变量进行运算。

Java 提供了许多组操作符。它们按功能分类。

在本教程中，我们将遍历所有的 Java 操作符，了解它们的功能以及如何使用它们。

## 2.算术运算符

我们使用算术运算符来执行简单的数学运算。我们要注意的是，**算术运算符只作用于本原数类型和它们的装箱类型**，比如`int`和`Integer`。

接下来，让我们看看算术运算符组中有哪些运算符。

### 2.1.加法运算符

加法运算符(+)允许我们将两个值相加或连接两个字符串:

```
int ten = 5 + 5;
String youAndMe = "You " + "and" + " Me";
```

### 2.2.减法运算符

通常，我们使用减法运算符(-)从一个值中减去另一个值:

```
int five = 10 - 5;
int negative80 = 20 - 100;
```

### 2.3.乘法运算符

乘法运算符(*)用于将两个值或变量相乘:

```
int hundred = 20 * 5;
int fifteen = -5 * -3;
```

### 2.4.除法运算符

除法运算符(/)允许我们将左边的值除以右边的值:

```
int four = 20 / 5;
int seven = 15 / 2;
```

当我们对两个整数值(`byte`、`short`、`int`和 `long`)使用除法运算符时，我们应该注意到**的结果是商值。余数不包括**。

如上例所示，如果我们计算`15 / 2`，商是`7,`，余数是`1`。因此，我们有了`15 / 2 = 7`。

### 2.5.模运算符

我们可以用除法运算符得到商。但是，**如果我们只想得到除法运算的余数**，可以使用[模运算符](/web/20220707143821/https://www.baeldung.com/modulo-java) (%):

```
int one = 15 % 2;
int zero = 10 % 5;
```

## 3.一元运算符

顾名思义， **[一元运算符](/web/20220707143821/https://www.baeldung.com/java-unary-operators)只需要一个单操作数**。例如，我们通常使用一元运算符对变量或值进行递增、递减或求反。

现在，让我们来看看 Java 中一元运算符的细节。

### 3.1.一元加号运算符

一元加号运算符(+)表示正值。如果数字是正数，我们可以省略'+'运算符:

```
int five = +5; // same as: int five = 5
```

### 3.2.一元减运算符

与一元加号运算符相反，一元减号运算符(-)对值或表达式求反:

```
int negativeFive = -5;
int eighty = -(20 - 100);
```

### 3.3.逻辑补码运算符

逻辑补码运算符(！)也被称为[【非】运算符](/web/20220707143821/https://www.baeldung.com/java-using-not-in-if-conditions)。我们可以用它来反转一个`boolean`变量或值的值:

```
boolean aTrue = true;
boolean bFalse = !aTrue;
```

### 3.4.增量运算符

增量运算符(++)允许我们将变量的值增加 1:

```
int number = 5;
number++; // number = 6
```

### 3.5.减量操作器

减量运算符(–)的作用与增量运算符相反。它将变量的值减少 1:

```
int number = 5;
number--; // number = 4
```

我们应该记住**递增和递减运算符只能用在变量**上。比如“`int a = 5; a++;`”就可以了。不过“`5++`这个表达式就不编了。

## 4.关系运算符

关系运算符也可以称为“比较运算符”。基本上，我们使用这些运算符来比较两个值或变量。

### 4.1.“等于”运算符

我们使用“等于”运算符(==)来比较两边的值。如果它们相等，操作返回`true`:

```
int number1 = 5;
int number2 = 5;
boolean theyAreEqual = number1 == number2;
```

“等于”运算符非常简单。另一方面，`Object`类已经提供了`equals()`方法。由于`Object`类是所有 Java 类的超类，所有 Java 对象都可以使用`equals()`方法进行相互比较。

当我们想要[比较两个对象](/web/20220707143821/https://www.baeldung.com/java-comparing-objects)时——例如，当我们[比较`Long`对象](/web/20220707143821/https://www.baeldung.com/java-compare-long-values)或[比较`String`s](/web/20220707143821/https://www.baeldung.com/java-compare-strings)—**时，我们应该明智地在`equals()` 方法和“等于”运算符**的比较方法之间进行选择。

### 4.2.“不等于”运算符

“不等于”运算符(！=)与' == '运算符相反。如果两边的值不相等，操作返回`true`:

```
int number1 = 5;
int number2 = 7;
boolean notEqual = number1 != number2;
```

### 4.3.“大于”运算符

当我们用“大于”运算符(>)比较两个值时，如果左边的值大于右边的值，则返回`true`:

```
int number1 = 7;
int number2 = 5;
boolean greater = number1 > number2;
```

### 4.4。“大于或等于”运算符

“大于或等于”运算符(> =)比较两边的值，如果左边的操作数大于或等于右边的操作数，则返回`true`:

```
int number1 = 7;
int number2 = 5;
boolean greaterThanOrEqualTo = number1 >= number2;
number1 = 5;
greaterThanOrEqualTo = number1 >= number2;
```

### 4.5.“小于”运算符

“小于”运算符(true:

```
int number1 = 4;
int number2 = 5;
boolean lessThan = number1 < number2;
```

### 4.6。“小于或等于”运算符

类似地，“小于或等于”运算符(< =)比较两边的值，如果左边的操作数小于或等于右边的操作数，则返回`true`:

```
int number1 = 4;
int number2 = 5;
boolean lessThanOrEqualTo = number1 <= number2;
number1 = 5;
lessThanOrEqualTo = number1 <= number2;
```

## 5.逻辑运算符

Java 中有两个逻辑操作符:逻辑 AND 和 OR 操作符。基本上，它们的功能与数字电子中的[与门](https://web.archive.org/web/20220707143821/https://en.wikipedia.org/wiki/AND_gate)和[或门](https://web.archive.org/web/20220707143821/https://en.wikipedia.org/wiki/OR_gate)非常相似。

通常我们使用一个逻辑运算符，有两个操作数，操作数是变量或表达式，可以求值为`boolean`。

接下来，让我们仔细看看它们。

### 5.1.逻辑 AND 运算符

仅当两个操作数都是`true`时，逻辑 AND 运算符(`&&`)才返回`true`:

```
int number1 = 7;
int number2 = 5;
boolean resultTrue = (number1 > 0) && (number1 > number2);
```

### 5.2.逻辑“或”运算符

与'`&&`'运算符不同，如果至少有一个操作数为`true`，逻辑 or 运算符(`||`)将返回`true`:

```
int number1 = 7;
int number2 = 5;
boolean resultTrue = (number1 > 100) || (number1 > number2);
```

**我们应该注意到，逻辑 OR 运算符具有[短路效应](/web/20220707143821/https://www.baeldung.com/java-logical-vs-bitwise-or-operator#short-circuit) :** 只要其中一个操作数被求值为`true,`，它就返回`true`，而不会对其余操作数求值。

## 6.三元运算符

一个[三元运算符](/web/20220707143821/https://www.baeldung.com/java-ternary-operator)是`if-then-else`语句的缩写形式。它有三个操作数，所以叫三元数。首先，让我们看看标准的`if-then-else`语句语法:

```
if ( condition ) {
    expression1
} else {
    expression2
}
```

我们可以使用三元运算符将上面的`if-then-else`语句转换成一个精简版本:

让我们看看它的语法:

```
condition ? expression1 : expression2
```

接下来，让我们通过一个简单的例子来理解三元运算符是如何工作的:

```
int number = 100;
String greaterThan50 = number > 50 ? "The number is greater than 50" : "The number is NOT greater than 50";
```

## 7.按位和位移运算符

由于文章“ [Java 位操作符](/web/20220707143821/https://www.baeldung.com/java-bitwise-operators)”涵盖了位和位移操作符的细节，我们将在本教程中简要总结这些操作符。

### 7.1.按位 AND 运算符

按位 AND 运算符(&)返回输入值的逐位 AND:

```
int number1 = 12;
int number2 = 14;
int twelve = number1 & number2; // 00001100 & 00001110 = 00001100 = 12
```

### 7.2.按位 OR 运算符

按位 or 运算符(|)返回输入值的逐位 OR 值:

```
int number1 = 12;
int number2 = 14;
int fourteen = number1 | number2; // 00001100 | 00001110 = 00001110 = 14
```

### 7.3.按位异或运算符

[位异或](/web/20220707143821/https://www.baeldung.com/java-xor-operator)(异或)运算符(^)返回输入值的逐位异或:

```
int number1 = 12;
int number2 = 14;
int two = number1 ^ number2; // 00001100 ^ 00001110 = 00000010 = 2
```

### 7.4.按位求补运算符

按位求补运算符(~)是一元运算符。它返回值的补码表示，将输入值的所有位反转:

```
int number = 12;
int negative13 = ~number; // ~00001100 = 11110011 = -13
```

### 7.5.左移运算符

移位运算符将位向左或向右移位给定的次数。

左移运算符(<

接下来，让我们将数字 12 左移两次:

```
int fourtyeight = 12 << 2; // 00001100 << 2 = 00110000 = 48
```

**`n << x`与数字`n`乘以`x`的 2 次方具有相同的效果。**

### 7.6.有符号右移位运算符

带符号的右移运算符(>>)将位向右移动由右侧操作数定义的次数，结果在左边的空位上填充 0。

我们应该注意到，移位后最左边的位置取决于符号扩展。

接下来，让我们对数字 12 和-12 进行两次“有符号右移”来看看区别:

```
int three = 12 >> 2; // 00001100 >> 2 = 00000011 = 3
int negativeThree = -12 >> 2; // 11110100 >> 2 = 11111101 = -3
```

如上面的第二个例子所示，如果数字是负数，每次移位后最左边的位置将由符号扩展设置。

**`n >> x`具有将数字`n`除以`x`的 2 次方的相同效果。**

### 7.7.无符号右移位运算符

无符号右移位运算符(>>>)的工作方式与“> >”运算符类似。唯一的区别是移位后，**最左边的位被设置为 0** 。

接下来，让我们对数字 12 和-12 进行两次无符号右移，看看有什么不同:

```
int three = 12 >>> 2; // 00001100 >> 2 = 00000011 = 3
int result = -12 >>> 2; // result = 1073741821 (11111111111111111111111111110100 >>> 2 = 00111111111111111111111111111101)
```

正如我们在上面的第二个例子中看到的，**>>>操作符用 0 填充左边的空白，不管数字是正还是负**。

## 8.“`instanceof`”运算符

有时，当我们有一个对象时，我们想测试它是否是给定类型的[实例。“`instanceof`”运算符可以帮助我们做到这一点:](/web/20220707143821/https://www.baeldung.com/java-instanceof)

```
boolean resultTrue = Long.valueOf(20) instanceof Number;
```

## 9.赋值运算符

我们使用赋值操作符给变量赋值。接下来，让我们看看在 Java 中可以使用哪些赋值操作符。

### 9.1.简单赋值运算符

简单赋值操作符(=)是 Java 中一个简单但重要的操作符。实际上，在前面的例子中，我们已经使用过很多次了。它将右边的值赋给左边的操作数:

```
int seven = 7;
```

### 9.2.复合赋值

我们学过算术运算符。我们可以将算术运算符和简单赋值运算符结合起来，创建复合赋值。

比如我们可以把“`a = a + 5`”写成复合式:“`a += 5`”。

最后，让我们通过例子来了解 Java 中所有受支持的复合赋值:

```
// Assuming all variables (a,b,c,d,e) have the initial value 10
a += 4; // a = 14, same as a = a + 4
b -= 4; // b = 6, same as b = b - 4
c *= 4; // c = 40, same as c = c * 4
d /= 4; // d = 2, same as d = d / 4
e %= 4; // e = 2, same as e = e % 4
```

## 10.结论

Java 为不同的功能提供了许多组操作符。在本文中，我们已经讨论了 Java 中的操作符。