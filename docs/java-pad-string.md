# 在 Java 中用零或空格填充字符串

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-pad-string>

## 1。概述

在这个简短的教程中，我们将看到如何在 Java 中填充一个`String`。我们将主要关注左填充，这意味着我们将添加前导空格或零，直到它达到所需的长度。

右侧填充的方法`String`非常相似，所以我们只指出不同之处。

## 2.使用自定义方法填充 a `String`

Java 中的`String`类没有提供方便的填充方法，所以让我们自己创建几个方法。

首先，让我们设定一些期望:

```java
assertEquals("    123456", padLeftZeros("123456", 10));
assertEquals("0000123456", padLeftZeros("123456", 10));
```

### 2.1.使用`StringBuilder`

我们可以通过`StringBuilder`和一些程序逻辑来实现这一点:

```java
public String padLeftZeros(String inputString, int length) {
    if (inputString.length() >= length) {
        return inputString;
    }
    StringBuilder sb = new StringBuilder();
    while (sb.length() < length - inputString.length()) {
        sb.append('0');
    }
    sb.append(inputString);

    return sb.toString();
}
```

我们在这里可以看到，如果原始文本的长度等于或大于所需的长度，我们返回它的未更改版本。否则，我们创建一个新的`String`，从空格开始，添加原来的。

**当然，如果我们想用不同的字符填充** **，我们可以用它来代替`0`。**

同样，如果我们想要右填充，我们只需要做`new` `StringBuilder(inputString)`来代替，然后在末尾添加空格。

### 2.2.使用`substring`

另一种做左填充的方法是**创建一个只包含填充字符的期望长度的`String`，然后使用`substring()`方法**:

```java
StringBuilder sb = new StringBuilder();
for (int i = 0; i < length; i++) {
    sb.append(' ');
}

return sb.substring(inputString.length()) + inputString;
```

### 2.3.使用`String.format`

最后，从 Java 5 开始，我们可以使用`String` `.format()`:

```java
return String.format("%1$" + length + "s", inputString).replace(' ', '0');
```

**我们应该注意，缺省情况下，填充操作将使用空格来执行。这就是为什么我们需要使用`replace()`方法来填充零或任何其他字符。**

对于正确的 pad，我们只需使用不同的标志:`%1$-`。

## 3.使用库填充 a `String`

此外，还有一些已经提供填充功能的外部库。

### 3.1.Apache Commons Lang

Apache Commons Lang 提供了一个 Java 实用程序类包。其中最受欢迎的是 [`StringUtils`](https://web.archive.org/web/20220707143819/https://commons.apache.org/proper/commons-lang/apidocs/org/apache/commons/lang3/StringUtils.html) 。

要使用它，我们需要通过将[的依赖项](https://web.archive.org/web/20220707143819/https://search.maven.org/classic/#search%7Cga%7C1%7Cg%3A%22org.apache.commons%22%20AND%20a%3A%22commons-lang3%22)添加到我们的`pom.xml`文件中来将它包含到我们的项目中:

```java
<dependency>
    <groupId>org.apache.commons</groupId>
    <artifactId>commons-lang3</artifactId>
    <version>3.12.0</version>
</dependency>
```

**然后我们传递`inputString`和`length`，就像我们创建的方法一样。**

我们也可以传递填充字符:

```java
assertEquals("    123456", StringUtils.leftPad("123456", 10));
assertEquals("0000123456", StringUtils.leftPad("123456", 10, "0"));
```

同样，默认情况下，`String`将用空格填充，或者我们需要显式设置另一个填充字符。

也有相应的`rightPad()`方法。

**要探索 Apache Commons Lang 3 的更多特性，请查看[我们的入门教程](/web/20220707143819/https://www.baeldung.com/java-commons-lang-3)。**要查看**使用`StringUtils`类操纵`String`的其他方式，请参考[这篇文章](/web/20220707143819/https://www.baeldung.com/string-processing-commons-lang)。**

### 3.2.谷歌番石榴

我们可以使用的另一个库是谷歌的[番石榴](https://web.archive.org/web/20220707143819/https://github.com/google/guava)。

当然，我们首先需要通过添加[的依赖项](https://web.archive.org/web/20220707143819/https://search.maven.org/classic/#search%7Cga%7C1%7Cg%3A%22com.google.guava%22%20AND%20a%3A%22guava%22)将它添加到项目中:

```java
<dependency>
    <groupId>com.google.guava</groupId>
    <artifactId>guava</artifactId>
    <version>31.0.1-jre</version>
</dependency>
```

**然后我们用`Strings`类**:

```java
assertEquals("    123456", Strings.padStart("123456", 10, ' '));
assertEquals("0000123456", Strings.padStart("123456", 10, '0'));
```

**这个方法没有默认的填充字符，所以每次都需要传递。**

为了正确的垫，我们可以使用`padEnd()`的方法。

番石榴图书馆提供了更多的功能，我们已经介绍了其中的许多功能。在这里寻找与番石榴相关的文章。

## 4.结论

在这篇简短的文章中，我们展示了如何在 Java 中填充一个`String`。我们使用我们自己的实现或现有的库给出了例子。

像往常一样，完整的源代码可以在 GitHub 上找到[。](https://web.archive.org/web/20220707143819/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-string-algorithms-2)