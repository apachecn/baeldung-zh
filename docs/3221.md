# Java objects . hash()vs objects . hashcode()

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-objects-hash-vs-objects-hashcode>

## 1.介绍

hashcode 是对象内容的数字表示。

在 Java 中，有几种不同的方法可以用来获取对象的 hashcode:

*   `Object.hashCode() `
*   `Objects.hashCode() –` 在 Java 7 中引入
*   `Objects.hash()`–在 Java 7 中引入

在本教程中，我们将研究其中的每一种方法。首先，我们将从定义和基本示例开始。在我们了解了基本用法之后，我们将深入了解它们之间的差异以及这些差异可能带来的后果。

## 2.基本用法

### 2.1.`Object.hashCode()`

我们可以使用 [`Object.hashCode()`](/web/20221129003257/https://www.baeldung.com/java-hashcode) 方法来检索一个对象的 hashcode。它和`Objects.hashCode()`非常相似，除了如果我们的对象是`null`我们就不能使用它。

也就是说，让我们在两个相同的`Double`对象上调用`Object.hashCode()`:

```
Double valueOne = Double.valueOf(1.0012);
Double valueTwo = Double.valueOf(1.0012);

int hashCode1 = valueOne.hashCode();
int hashCode2 = valueTwo.hashCode();

assertEquals(hashCode1, hashCode2);
```

正如所料，我们收到相同的 hashcodes。

相比之下，现在让我们对一个`null`对象调用`Object.hashCode()`,期望抛出一个`NullPointerException`:

```
Double value = null;
value.hashCode();
```

### 2.2.`Objects.hashCode()`

`Objects.hashCode()`是一个空安全的方法，我们可以用它来获得一个对象的 [hashcode](/web/20221129003257/https://www.baeldung.com/java-hashcode) 。Hashcodes 对于哈希表和`equals()`的正确实现是必要的。

在 [JavaDoc](https://web.archive.org/web/20221129003257/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/Object.html#hashCode()) 中指定的 hashcode 的一般契约是:

*   在应用程序的同一次执行中，每次调用未更改的对象时，返回的整数都是相同的
*   对于根据它们的`equals()`方法相等的两个对象，返回相同的 hashcode

尽管这不是必需的，不同的对象尽可能返回不同的 hashcodes。

首先，让我们在两个相同的字符串上调用`Objects.hashCode()`:

```
String stringOne = "test";
String stringTwo = "test";
int hashCode1 = Objects.hashCode(stringOne);
int hashCode2 = Objects.hashCode(stringTwo);

assertEquals(hashCode1, hashCode2);
```

现在，我们希望返回的 hashcodes 是相同的。

另一方面，如果我们提供一个`null`到`Objects.hashCode()`，我们将得到零:

```
String nullString = null;
int hashCode = Objects.hashCode(nullString);
assertEquals(0, hashCode);
```

### 2.3.`Objects.hash()`

与只接受单个对象的`Objects.hashCode(),`不同，`Objects.hash()`可以接受一个或多个对象，并为它们提供一个 hashcode。在幕后，`hash()`方法的工作原理是将提供的对象放入一个数组，并对它们调用`Arrays.hashCode()`。如果我们只给`Objects.hash()`方法提供一个对象，我们不能期望得到与调用对象上的`Objects.hashCode()`相同的结果。

首先，让我们用两对相同的字符串调用`Objects.hash()`:

```
String strOne = "one";
String strTwo = "two";
String strOne2 = "one";
String strTwo2 = "two";

int hashCode1 = Objects.hash(strOne, strTwo);
int hashCode2 = Objects.hash(strOne2, strTwo2);

assertEquals(hashCode1, hashCode2);
```

接下来，让我们用一个字符串调用`Objects.hash()`和`Objects.hashCode()`:

```
String testString = "test string";
int hashCode1 = Objects.hash(testString);
int hashCode2 = Objects.hashCode(testString);

assertNotEquals(hashCode1, hashCode2);
```

正如所料，返回的两个 hashcodes 不匹配。

## 3.主要差异

在上一节中，我们谈到了`Objects.hash()`和`Objects.hashCode()`之间的主要区别。现在，让我们更深入地挖掘一下，这样我们就能理解一些分支。

如果我们需要覆盖我们类的一个`[equals()](/web/20221129003257/https://www.baeldung.com/java-equals-hashcode-contracts)`方法，正确地覆盖`hashCode()`也是很重要的。

让我们首先为我们的例子创建一个简单的`Player`类:

```
public class Player {
    private String firstName;
    private String lastName;
    private String position;

    // Standard getters/setters
}
```

### 3.1.多字段 Hashcode 实现

假设我们的`Player`类在所有三个字段中都被认为是唯一的:`firstName`、`lastName,`和`position`。

也就是说，让我们看看在 Java 7 之前[我们是如何实现](/web/20221129003257/https://www.baeldung.com/java-eclipse-equals-and-hashcode) `Player.hashCode()`的:

```
@Override
public int hashCode() {
    int result = 17;
    result = 31 * result + firstName != null ? firstName.hashCode() : 0;
    result = 31 * result + lastName != null ? lastName.hashCode() : 0;
    result = 31 * result + position != null ? position.hashCode() : 0;
    return result;
}
```

因为`Objects.hashCode()`和`Objects.hash()`都是在 Java 7 中引入的，所以我们必须在每个字段上调用`Object.hashCode()`之前显式检查`null`。

让我们确认我们都可以在同一个对象上调用`hashCode()`两次并得到相同的结果，并且我们可以在相同的对象上调用它并得到相同的结果:

```
Player player = new Player("Eduardo", "Rodriguez", "Pitcher");
Player indenticalPlayer = new Player("Eduardo", "Rodriguez", "Pitcher");

int hashCode1 = player.hashCode();
int hashCode2 = player.hashCode();
int hashCode3 = indenticalPlayer.hashCode();

assertEquals(hashCode1, hashCode2);
assertEquals(hashCode1, hashCode3);
```

接下来，让我们看看如何利用通过`Objects.hashCode()`获得的空安全来缩短这个时间:

```
int result = 17;
result = 31 * result + Objects.hashCode(firstName);
result = 31 * result + Objects.hashCode(lastName);
result = 31 * result + Objects.hashCode(position);
return result;
```

如果我们运行相同的单元测试，我们应该期待相同的结果。

因为我们的类依赖于多个字段来确定等式，所以让我们更进一步，使用`Objects.hash()`使我们的`hashCode()`方法非常简洁:

```
return Objects.hash(firstName, lastName, position);
```

更新之后，我们应该能够再次成功运行我们的单元测试。

### 3.2.`Objects.hash()`详情

在幕后，当我们调用`Objects.hash(),`时，值被放在一个数组中，然后在数组上调用`Arrays.hashCode()`。

也就是说，让我们创建一个`Player`并将它的 hashcode 与我们使用的值进行比较:

```
@Test
public void whenCallingHashCodeAndArraysHashCode_thenSameHashCodeReturned() {
    Player player = new Player("Bobby", "Dalbec", "First Base");
    int hashcode1 = player.hashCode();
    String[] playerInfo = {"Bobby", "Dalbec", "First Base"};
    int hashcode2 = Arrays.hashCode(playerInfo);

    assertEquals(hashcode1, hashcode2);
}
```

我们创建了一个`Player`,然后创建了一个`String[].` ,然后我们在`Player`上调用了`hashCode()`,在数组上使用了`Arrays.hashCode()`,收到了相同的 hashcode。

## 4.结论

在本文中，我们学习了如何以及何时使用`Object.hashCode()`、`Objects.hashCode()`和`Objects.hash()`。此外，我们还研究了它们之间的差异。

作为回顾，让我们快速总结一下它们的用法:

*   `Object.hashCode()`:用于获取单个非空对象的 hashcode
*   `Objects.hashCode()`:用于获取可能为空的单个对象的 hashcode
*   `Objects.hash()`:用于获取多个对象的 hashcode

与往常一样，GitHub 上的示例代码[可用。](https://web.archive.org/web/20221129003257/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-lang-4)