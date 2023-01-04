# 比较 Java 中的字符

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-compare-characters>

## 1.概观

在这个简短的教程中，我们将探索在 Java 中比较字符的不同方法。

我们先从讨论如何比较原始字符开始。然后，我们将看看比较`Character`对象的不同方法。

## 2.比较 P **原生** **人物**

首先，让我们开始强调如何比较原始字符。

### 2.1.使用关系操作符

 **通常，比较字符的最简单方法是使用[关系运算符](/web/20221023150043/https://www.baeldung.com/java-operators#relational-operators)。

**简而言之，Java 中的字符根据其** [**ASCII 码**](/web/20221023150043/https://www.baeldung.com/cs/ascii-code) 的顺序进行比较:

```java
assertFalse('a' == 'A');
assertTrue('a' < 'v');
assertTrue('F' > 'D'); 
```

### 2.2.使用`Character.compare() `方法

类似地，另一个解决方案是使用`Character` 类的`compare()`方法。

简单地说，*字符*类将原始类型`char`的值包装在一个对象中。**`compare()`方法接受两个`char`参数，并对它们进行数值比较**:

```java
assertTrue(Character.compare('C', 'C') == 0);
assertTrue(Character.compare('f', 'A') > 0);
assertTrue(Character.compare('Y', 'z') < 0); 
```

如上所示，`compare(char a, char b)`方法返回一个`int`值。**表示`a`和`b`的 ASCII 码的区别。**

如果两个字符值相同，返回值等于零，如果`a < b`小于零，否则大于零。

## 3.比较角色对象

现在我们知道了如何比较原始字符，让我们看看如何比较`Character` 对象。

### 3.1.使用`Character.compareTo()` 方法

`Character` 类提供了`compareTo()`方法来从数字上比较两个角色对象:

```java
Character chK = Character.valueOf('K');
assertTrue(chK.compareTo(chK) == 0);

Character chG = Character.valueOf('G');
assertTrue(chK.compareTo(chG) > 0);

Character chH = Character.valueOf('H');
assertTrue(chG.compareTo(chH) < 0); 
```

**在这里，我们使用了`valueOf()`方法来创建`Character`对象，因为 t** **构造函数从 Java 9** 开始就被弃用了。

### 3.2.使用`Object.equals()` 方法

此外，比较对象的一个常见解决方案是使用`equals()`方法。如果两个对象相等，则返回`true`，否则返回`false`。

那么，让我们看看如何用它来比较字符:

```java
Character chL = 'L';
assertTrue(chL.equals(chL));

Character chV = 'V';
assertFalse(chL.equals(chV)); 
```

### 3.3.使用`Objects.equals()` 方法

`Objects` 类由操作对象的实用方法组成。它提供了另一种通过`equals()`方法比较角色对象的方式:

```java
Character chA = 'A';
Character chB = 'B';

assertTrue(Objects.equals(chA, chA));
assertFalse(Objects.equals(chA, chB)); 
```

如果角色对象彼此相等，则`equals()` 方法返回`true`，否则返回`false`。

## 4.结论

在本文中，我们学习了在 Java 中比较原语和对象字符的各种方法。

和往常一样，本文中使用的代码可以在 GitHub 上找到[。](https://web.archive.org/web/20221023150043/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-string-operations-5)**