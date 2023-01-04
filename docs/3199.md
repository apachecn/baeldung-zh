# 逻辑与按位 OR 运算符

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-logical-vs-bitwise-or-operator>

## 1.介绍

在计算机编程中，OR 的使用情况是，它或者是布尔逻辑的逻辑构造**，或者是在比特级别操作数据的逐位数学运算**。

逻辑运算符用于根据特定条件做出决策，而按位运算符用于快速二进制计算，包括 IP 地址屏蔽。

在本教程中，我们将学习逻辑和按位 OR 运算符，分别用||和|表示**。**

## 2.逻辑或的使用

### 2.1.它是如何工作的

逻辑 OR 运算符处理布尔操作数。当至少有一个操作数是 T1 时，它**返回`true `，否则，它返回`false`:**

*   真||真=真
*   真||假=真
*   假||真=真
*   假||假=假

### 2.2.例子

让我们借助一些`boolean`变量来理解:

```
boolean condition1 = true; 
boolean condition2 = true; 
boolean condition3 = false; 
boolean condition4 = false;
```

当我们对两个`true`操作数应用逻辑 OR 时，结果将是`true`:

```
boolean result = condition1 || condition2;
assertTrue(result);
```

当我们对一个`true`和一个`false`操作数应用逻辑 OR 时，结果将是`true`:

```
boolean result = condition1 || condition3; 
assertTrue(result);
```

当我们对两个`false`操作数应用逻辑 OR 时，结果将是`false`:

```
boolean result = condition3 || condition4; 
assertFalse(result);
```

当有多个操作数时，从左到右有效地执行**评估。因此，表达式`condition1 || condition2 || condition3 || condition4` 将产生与以下相同的逻辑:**

```
boolean result1 = condition1 || condition2; 
boolean result2 = result1 || condition3;
boolean finalResult = result2 || condition4;
assertTrue(finalResult);
```

但是实际上，Java 可以在上面的表达式中走捷径。

## 3.短路

逻辑 OR 运算符具有短路行为。这意味着只要一个操作数被评估为`true`，它就返回`true`，而不评估剩余的操作数。

让我们考虑下面的例子:

```
boolean returnAndLog(boolean value) { 
    System.out.println("Returning " + value); 
    return value; 
} 

if (returnAndLog(true) || returnAndLog(false)) { 
} 

Output:
Returning true

if (returnAndLog(false) || returnAndLog(true)) { 
}

Output:
Returning false
Returning true
```

这里我们看到，如果前面的条件是`true`，那么第二个逻辑条件不会被计算。

我们应该注意，如果调用的任何方法有副作用，这可能会导致意想不到的结果。如果我们重写第一个示例，在`if`语句之前捕获`boolean`值，我们会得到不同的结果:

```
boolean result1 = returnAndLog(true);
boolean result2 = returnAndLog(false);

if (result1 || result2) {
}

Output:
Returning true
Returning false
```

## 4.使用按位“或”

**4.1。工作原理**

按位“或”是一个二元运算符，**它** **对两个整数操作数**的每个对应位进行“或”运算。如果至少有一位为 1，则返回 1，否则返回 0。此外，该运算符总是计算两个操作数:

*   1 | 1 = 1
*   1 | 0 = 1
*   0 | 1 = 1
*   0 | 0 = 0

因此，当我们对两个整数进行按位“或”运算时，结果将是一个新的整数。

### 4.2.例子

让我们考虑一个例子:

```
int four = 4; //0100 = 4
int three = 3; //0011 = 3
int fourORthree = four | three;
assertEquals(7, fourORthree); // 0111 = 7
```

现在，我们来看看上面的操作是如何工作的。

首先，每个整数被转换成它的二进制表示:

*   `4`的二进制表示是`0100`
*   `3`的二进制表示是`0011`

然后，对各个位进行按位“或”运算，得到代表最终结果的二进制表示:

```
0100
0011
----
0111
```

现在，`0111`，当转换回它的十进制表示时，会给我们最后的结果:整数`7`。

**当有多个操作数时，求值从左到右进行**。因此，表达式`1 | 2 | 3 | 4` 将被计算为:

```
int result1 = 1 | 2; 
int result2 = result1 | 3;
int finalResult = result2 | 4;
assertEquals(finalResult,7);
```

## 5.兼容类型

在这一节中，我们将看看这些操作符兼容的数据类型。

### 5.1.逻辑或

逻辑 OR 运算符只能用于布尔操作数。并且，将它与整数操作数一起使用会导致编译错误:

`boolean result = 1 || 2;`

`Compilation error: Operator '||' cannot be applied to 'int', 'int`

### 5.2.按位或

除了整数操作数，按位 OR 也可以用于布尔操作数。如果至少有一个操作数是`true`，则返回`true`，否则返回`false`。

让我们借助示例中的一些`boolean`变量来理解:

```
boolean condition1 = true;
boolean condition2 = true;
boolean condition3 = false;
boolean condition4 = false;

boolean condition1_OR_condition2 = condition1 | condition2;
assertTrue(condition1_OR_condition2);

boolean condition1_OR_condition3 = condition1 | condition3;
assertTrue(condition1_OR_condition3);

boolean condition3_OR_condition4 = condition3 | condition4;
assertFalse(condition3_OR_condition4);
```

## 6.优先

让我们回顾一下逻辑和按位 OR 运算符以及其他运算符的优先级:

*   优先级较高的运算符:++––*+–/> ><< >< = =！=
*   按位 AND: &
*   按位或:|
*   逻辑与:&&
*   逻辑或:||
*   优先级较低的运算符:？: = += -= *= /= >>= <<=

一个简单的例子将帮助我们更好地理解这一点:

```
boolean result = 2 + 4 == 5 || 3 < 5;
assertTrue(result);
```

考虑到逻辑 OR 运算符的低优先级，上面的表达式将计算为:

*   `((2+4) == 5) || (3 < 5)`
*   `And then, (6 == 5) || (3 < 5)`
*   `And then, false || true`

这就产生了结果`true.`

现在，考虑另一个使用按位 OR 运算符的示例:

```
int result = 1 + 2 | 5 - 1;
assertEquals(7, result);
```

上面的表达式将计算为:

*   `(1+2) | (5-1)`
*   `And then, 3 | 4`

因此，结果将是`7`。

## 7.结论

在本文中，我们学习了对布尔和整数操作数使用逻辑和按位 OR 运算符。

我们还研究了这两个操作符之间的主要区别以及它们在其他操作符中的优先级。

与往常一样，GitHub 上的示例代码[可用。](https://web.archive.org/web/20220707143818/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-lang-operators-2)