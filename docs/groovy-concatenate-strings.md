# 用 Groovy 连接字符串

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/groovy-concatenate-strings>

## 1.概观

在本教程中，我们将看看使用 Groovy 连接`String`的几种方法。注意， [Groovy 在线解释器](https://web.archive.org/web/20221110163406/https://groovyconsole.appspot.com/)在这里派上了用场。

我们将从定义一个`numOfWonder`变量开始，我们将在整个例子中使用它:

```java
def numOfWonder = 'seven'
```

## 2.串联运算符

很简单，我们可以使用 [+运算符](/web/20221110163406/https://www.baeldung.com/groovy-strings#string-concatenation)来连接`String` s:

```java
'The ' + numOfWonder + ' wonders of the world' 
```

类似地，Groovy 也支持左移<

```java
'The ' << numOfWonder << ' wonders of ' << 'the world'
```

## 3.字符串插值

下一步，我们将尝试在字符串文字中使用 [Groovy 表达式来提高代码的可读性:](/web/20221110163406/https://www.baeldung.com/groovy-strings#string-interpolation)

```java
"The $numOfWonder wonders of the world\n"
```

这也可以使用花括号来实现:

```java
"The ${numOfWonder} wonders of the world\n" 
```

## 4.多行字符串

假设我们想要打印世界上所有的奇迹，那么我们可以使用[的三重双引号](/web/20221110163406/https://www.baeldung.com/groovy-strings#triple-quoted-string)来定义一个多行的`String`，仍然包括我们的`numOfWonder`变量:

```java
"""
There are $numOfWonder wonders of the world.
Can you name them all? 
1\. The Great Pyramid of Giza
2\. Hanging Gardens of Babylon
3\. Colossus of Rhode
4\. Lighthouse of Alexendra
5\. Temple of Artemis
6\. Status of Zeus at Olympia
7\. Mausoleum at Halicarnassus
"""
```

## 5.串联方法

作为最后一个选择，我们将看看`String`的`concat` 方法:

```java
'The '.concat(numOfWonder).concat(' wonders of the world')​​​​​​​​​​​​​​​​​​​​​​​​​​​​​​​​​​​​​​​​​​
```

对于非常长的文本，我们建议使用 [`StringBuilder`](/web/20221110163406/https://www.baeldung.com/java-string-builder-string-buffer) 或 [`StringBuffer`](/web/20221110163406/https://www.baeldung.com/java-string-builder-string-buffer) 来代替:

```java
new StringBuilder().append('The ').append(numOfWonder).append(' wonders of the world')
new StringBuffer().append('The ').append(numOfWonder).append(' wonders of the world')​​​​​​​​​​​​​​​
```

## 6.结论

在本文中，我们快速浏览了如何使用 Groovy 连接`String` s。

像往常一样，本教程的完整源代码可以在 GitHub 上找到[。](https://web.archive.org/web/20221110163406/https://github.com/eugenp/tutorials/tree/master/core-groovy-modules/core-groovy-2)