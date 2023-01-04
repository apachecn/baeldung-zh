# 字符#isAlphabetic 对字符#isLetter

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-character-isletter-isalphabetic>

## 1.概观

在本教程中，我们将首先简要介绍每个定义的 Unicode 码位或字符范围的一些通用类别类型，以便**理解字母和字母字符**之间的区别。

进一步，**我们来看看 Java 中 [`Character`](https://web.archive.org/web/20220630131429/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/Character.html) 类**的`isAlphabetic()`和`isLetter()`方法。最后，我们将讨论这些方法之间的相似之处和区别。

## 2.Unicode 字符的常规类别类型

[Unicode 字符集](https://web.archive.org/web/20220630131429/https://en.wikipedia.org/wiki/List_of_Unicode_characters) (UCS)包含 1114112 个码位:U+0000—U+10FFFF。字符和代码点范围按类别分组。

`Character`类提供了两个重载版本的**[`getType()`](https://web.archive.org/web/20220630131429/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/Character.html#getType(char))方法，该方法返回一个指示角色的一般类别类型**的值。

让我们看看第一个方法的签名:

```
public static int getType(char ch)
```

此方法不能处理补充字符。为了处理所有的 Unicode 字符，包括补充字符，Java 的`Character`类提供了一个重载的 [`getType`](https://web.archive.org/web/20220630131429/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/Character.html#getType(int)) 方法，该方法具有以下签名:

```
public static int getType(int codePoint)
```

接下来，让我们开始看看一些通用的类别类型。

### 2.1.`UPPERCASE_LETTER`

`UPPERCASE_LETTER`通用类别类型代表大写字母。

当我们对一个大写字母调用`Character` # `getType`方法时，例如，`U`，该方法返回值 1，这相当于`UPPERCASE_LETTER`枚举值:

```
assertEquals(Character.UPPERCASE_LETTER, Character.getType('U'));
```

### 2.2.`LOWERCASE_LETTER`

`LOWERCASE_LETTER`常规类别类型与小写字母相关联。

当对小写字母调用`Character` # `getType`方法时，例如，`u`，该方法将返回值 2，与`LOWERCASE_LETTER`的枚举值相同:

```
assertEquals(Character.LOWERCASE_LETTER, Character.getType('u'));
```

### 2.3.`TITLECASE_LETTER`

接下来，`TITLECASE_LETTER`一般类别表示标题大小写字符。

有些字符看起来像成对的拉丁字母。当我们对这样的 Unicode 字符调用`Character` # `getType`方法时，这将返回值 3，它等于`TITLECASE_LETTER`枚举值:

```
assertEquals(Character.TITLECASE_LETTER, Character.getType('\u01f2'));
```

在这里，Unicode 字符'`\u01f2`'代表拉丁文大写字母'`D`'，后跟一个带抑扬符的小写'`Z`'。

### 2.4.`MODIFIER_LETTER`

在 Unicode 标准中，[修饰字母](https://web.archive.org/web/20220630131429/https://en.wikipedia.org/wiki/Modifier_letter)是“一个字母或符号，通常写在它以某种方式修饰的另一个字母旁边”。

`MODIFIER_LETTER`通用类别类型代表这样的修饰字母。

例如，修饰符字母 small `H`、‘`ʰ`’在传递给`Character` # `getType`方法时，返回值 4，这与`MODIFIER_LETTER`的枚举值相同:

```
assertEquals(Character.MODIFIER_LETTER, Character.getType('\u02b0'));
```

Unicode 字符'`\u020b`'代表修饰字母 small `H`。

### 2.5.`OTHER_LETTER`

`OTHER_LETTER`通用类别类型代表一个表意文字或一个单播字母表中的一个字母。表意文字是代表一种思想或概念的图形符号，独立于任何特定的语言。

单字母字母表只有一个字母盒。例如，希伯来语是一种单一的书写系统。

让我们看一个希伯来字母 Alef，'`א`'的例子，当我们把它传递给`Character` # `getType`方法时，它返回值 5，等于`OTHER_LETTER`的枚举值:

```
assertEquals(Character.OTHER_LETTER, Character.getType('\u05d0'));
```

Unicode 字符'`\u05d0`'代表希伯来字母 Alef。

### 2.6.`LETTER_NUMBER`

最后，`LETTER_NUMBER` 类别与由字母或字母状符号组成的数字相关联。

例如，罗马数字属于`LETTER_NUMBER`大类。当我们用罗马数字 5 'ⅴ'调用`Character` # `getType`方法时，它返回值 10，等于枚举`LETTER_NUMBER`值:

```
assertEquals(Character.LETTER_NUMBER, Character.getType('\u2164'));
```

Unicode 字符'`\u2164`'代表罗马数字五。

接下来我们来看看`Character` # `isAlphabetic`的方法。

## 3.`Character` # `isAlphabetic`

首先，我们来看看 [`isAlphabetic`](https://web.archive.org/web/20220630131429/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/Character.html#isAlphabetic(int)) 的签名方法:

```
public static boolean isAlphabetic(int codePoint)
```

这将 Unicode 码位作为输入参数，如果指定的 Unicode 码位是字母，则**返回`true`，否则返回`false`。**

如果字符的一般类别类型是以下任一类型，则字符是字母:

*   `UPPERCASE_LETTER`
*   `LOWERCASE_LETTER`
*   `TITLECASE_LETTER`
*   `MODIFIER_LETTER`
*   `OTHER_LETTER`
*   `LETTER_NUMBER`

此外，如果一个字符具有 Unicode 标准定义的贡献属性`Other_Alphabetic` ，那么它就是字母。

让我们看几个字母字符的例子:

```
assertTrue(Character.isAlphabetic('A'));
assertTrue(Character.isAlphabetic('\u01f2'));
```

在上面的例子中，我们将代表拉丁大写字母'`D`'的`UPPERCASE_LETTER` `‘A'`和`TITLECASE_LETTER ‘\u01f2'`，后面跟着一个带有连字符的小'【T4 '】传递给`isAlphabetic`方法，它返回 true。

## 4.`Character` # `isLetter`

Java 的`Character`类提供了`isLetter()`方法来确定一个指定的字符是否是一个字母。让我们看看方法签名:

```
public static boolean isLetter(char ch)
```

它将一个字符作为输入参数，如果指定的字符是一个字母，**返回`true`，否则返回`false`。**

如果由`Character` # `getType`方法提供的字符的一般类别类型是以下任何一种，则该字符被认为是字母:

*   `UPPERCASE_LETTER`
*   `LOWERCASE_LETTER`
*   `TITLECASE_LETTER`
*   `MODIFIER_LETTER`
*   `OTHER_LETTER`

但是，此方法不能处理补充字符。为了处理所有的 Unicode 字符，包括补充字符，Java 的`Character`类提供了一个重载版本的`isLetter()`方法:

```
public static boolean isLetter(int codePoint)
```

这个方法可以处理所有的 Unicode 字符，因为它将一个 Unicode 码位作为输入参数。此外，如果指定的 Unicode 码位是我们前面定义的字母，它将返回`true`。

让我们来看几个字母字符的例子:

```
assertTrue(Character.isAlphabetic('a'));
assertTrue(Character.isAlphabetic('\u02b0'));
```

在上面的例子中，我们向`isLetter`方法输入代表修饰字母小`H`的`LOWERCASE_LETTER ‘a'`和`MODIFIER_LETTER ‘\u02b0'`，它返回 true。

## 5.比较和对比

最后，**我们可以看到，所有的字母都是字母字符，但不是所有的字母字符都是字母**。

换句话说，如果一个字符是一个字母或者具有常规类别`LETTER_NUMBER`，则`isAlphabetic`方法返回`true`。此外，如果字符具有 Unicode 标准定义的`Other_Alphabetic`属性，它也会返回`true`。

首先，让我们看一个既是字母又是字母表的字符的例子——字符'`a`':

```
assertTrue(Character.isLetter('a')); 
assertTrue(Character.isAlphabetic('a'));
```

字符'`a`'作为输入参数传递给`isLetter()`和`isAlphabetic()`方法时，返回`true`。

接下来，让我们看一个例子，一个字符是一个字母表，但不是一个字母。在这种情况下，我们将使用 Unicode 字符'`\u2164`'，它代表罗马数字五:

```
assertFalse(Character.isLetter('\u2164'));
assertTrue(Character.isAlphabetic('\u2164'));
```

**当传递给`isLetter()`方法时，Unicode 字符“`\u2164`”返回 false。另一方面，当传递给`isAlphabetic()`方法时，它返回`true`。**

当然，对于英语来说，这种区别并没有什么不同。因为英语中所有的字母都属于字母表的范畴。另一方面，其他语言中的一些字符可能有区别。

## 6.结论

在本文中，我们了解了 Unicode 码位的不同类别。此外，我们讨论了`isAlphabetic()`和`isLetter()`方法之间的相似之处和不同之处。

和往常一样，所有这些代码示例都可以在 GitHub 上的[处获得。](https://web.archive.org/web/20220630131429/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-char)