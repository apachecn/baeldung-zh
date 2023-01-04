# 移除 Java StringBuilder 的最后一个字符

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-remove-last-character-stringbuilder>

## 1.概观

当我们想在 Java 中构建一个字符串时，我们通常选择方便的`[StringBuilder](/web/20221022090111/https://www.baeldung.com/java-string-builder-string-buffer)`来完成这项工作。

假设我们有一个包含一些字符串段的`StringBuilder`序列，我们想从中删除最后一个字符。在这个快速教程中，我们将探索三种方法来做到这一点。

## 2.使用`StringBuilder`的`deleteCharAt()`方法

`StringBuilder`类有 [`deleteCharAt()`](https://web.archive.org/web/20221022090111/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/StringBuilder.html#deleteCharAt(int)) 方法。它允许我们在期望的位置删除字符`.`

**`deleteCharAt()`方法只有一个参数:我们想要删除的`char`索引。**

因此，如果我们将最后一个字符的索引传递给方法，就可以移除该字符。为了简单起见，我们将使用单元测试断言来验证它是否按预期工作。

接下来，让我们创建一个测试来检查它是否有效:

```java
StringBuilder sb = new StringBuilder("Using the sb.deleteCharAt() method!");
sb.deleteCharAt(sb.length() - 1);
assertEquals("Using the sb.deleteCharAt() method", sb.toString()); 
```

如上面的测试所示，我们将最后一个字符的索引(`sb.length() -1`)传递给`deleteCharAt()`方法，并期望删除结尾的感叹号(`!`)。

如果我们进行测试，就会通过。于是，`deleteCharAt()`解决了问题。

## 3.使用`StringBuilder`的`replace()`方法

`StringBuilder`的`[replace()](https://web.archive.org/web/20221022090111/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/StringBuilder.html#replace(int,int,java.lang.String))`方法允许我们用给定的字符串替换序列子串中的字符。该方法接受三个参数:

*   `start`索引–开始索引，包含开始索引
*   `end`索引–结束索引，不含
*   `replacement`–用于替换的字符串

假设序列中最后一个字符的索引是`lastIdx.` **，如果要删除最后一个字符，可以将`lastIdx`作为起始索引，`lastIdx+1`作为结束索引，空字符串`“”`作为对`replace()`** 的替换:

```java
StringBuilder sb = new StringBuilder("Using the sb.replace() method!");
int last = sb.length() - 1;
sb.replace(last, last + 1, "");
assertEquals("Using the sb.replace() method", sb.toString()); 
```

现在，如果我们运行一下，上面的测试就通过了。所以，可以用`replace()`法来解决问题。

## 4.使用`StringBuilder`的 `substring()`方法

我们可以使用`StringBuilder`的`[substring()](https://web.archive.org/web/20221022090111/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/StringBuilder.html#substring(int,int))`方法从字符串的给定`start`和`end`索引中获得一个子序列。该方法需要两个参数，即开始索引(包含)和结束索引(不包含)。

值得一提的是，`substring()`方法返回一个新的`String`对象。换句话说，**`substring()`方法不会修改`StringBuilder`对象**。

我们可以将 0 作为`start`索引，将最后一个字符的索引作为`end`索引传递给`substring()`方法，以获得去掉最后一个字符的字符串:

```java
StringBuilder sb = new StringBuilder("Using the sb.substring() method!");
assertEquals("Using the sb.substring() method", sb.substring(0, sb.length() - 1));
//the stringBuilder object is not changed
assertEquals("Using the sb.substring() method!", sb.toString()); 
```

如果我们执行它，测试就通过了。

正如我们在测试中看到的，尽管 `substring()`返回的`String`没有最后一个字符(`!`)，但原来的`StringBuilder`没有改变。

## 5.结论

在这篇简短的文章中，我们学习了如何从一个`StringBuilder`序列中移除最后一个字符。

像往常一样，这里展示的所有代码片段都可以在 GitHub 上获得。