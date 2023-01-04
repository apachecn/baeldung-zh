# 从字符串中删除开始和结束双引号

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-remove-start-end-double-quote>

## 1.概观

在本文中，我们将研究在 Java 中**从`String`中移除开始和结束双引号的不同方法。**

我们将在这里探讨的内容对于处理从文件中提取的或从其他来源接收的文本可能是有用的。

## 2.简单的方法:`substring`法

让我们先从一个简单的方法开始，**使用`substring`方法**。可以在一个`String`对象上调用这个方法来返回一个特定的子字符串。

该方法需要两个参数:

1.  `beginIndex` —子字符串应该开始的字符的索引
2.  `endIndex` —子字符串结束位置之后的索引

因此，考虑到我们的输入`String`包含在双引号中，我们可以使用`substring`方法:

```
String input = "\"text wrapped in double quotes\"";
String result = input.substring(1, input.length() - 1);
System.out.println("Input: " + input);
System.out.println("Result: " + result);
```

通过执行上面的代码，我们得到了以下输出:

```
Input: "text wrapped in double quotes"
Result: text wrapped in double quotes
```

当我们不确定`String`是否会用双引号括起来时，我们应该在运行`substring`方法之前检查一下:

```
if (input != null && input.length() >= 2 
      && input.charAt(0) == '\"' && input.charAt(input.length() - 1) == '\"') {
    result = input.substring(1, input.length() - 1);
}
```

在上面的例子中，我们检查了`String`至少有两个字符，并且以双引号开始和结束。

## 3.使用`replaceAll`方法

除了`substring`法，我们还可以使用`replaceAll`法。**该方法替换`String`中与给定正则表达式**匹配的所有部分。使用`replaceAll`，我们可以用空字符串替换所有出现的双引号:

```
String result = input.replaceAll("\"", "");
```

一方面，这种方法的优点是可以删除所有出现的双引号，即使字符串有多行。另一方面，使用这种方法，我们不能只删除`String.`开头和结尾的双引号

**要从**的开头和结尾去掉双引号`**String**,`我们可以使用一个更具体的正则表达式:

```
String result = input.replaceAll("^\"|\"$", "");
```

在执行了这个例子之后，出现在`String`开头或结尾的双引号将被空字符串替换。

为了理解这种方法，我们来分解一下正则表达式。

首先，我们有一个脱字符号(^)，后跟转义双引号(\ ")，用于匹配`String`开头的双引号。然后，有一个管道符号(|)来表示匹配的替代项，类似于 OR 逻辑运算符。

最后，我们避免了双引号后面跟一个美元符号($)来匹配在`String`末尾的双引号。

## 4.用番石榴

从`String`的开头和结尾去掉双引号的另一种可能的方法是**使用来自 Guava 库**的`CharMatcher`类:

```
String result = CharMatcher.is('\"').trimFrom(input);
```

**这种方法更容易理解，只删除了`String`** 的开始和结束引号。然而，为了让这种方法工作，我们需要将 [`guava`](https://web.archive.org/web/20221128235332/https://search.maven.org/search?q=g:com.google.guava%20a:guava) 库添加到我们的项目中:

```
<dependency>
    <groupId>com.google.guava</groupId>
    <artifactId>guava</artifactId>
    <version>${guava-version}</version>
</dependency>
```

在这种情况下，我们需要将`${guava-version}`属性设置为我们想要使用的版本。

## 5.结论

在本文中，我们探索了在 `**String**.`的开头和结尾去除双引号的不同方法，我们可以在实践中应用这些方法中的任何一种。各有利弊。

例如，在使用番石榴库时，我们有一个简单而优雅的解决方案。然而，如果我们的项目中不包括 Guava，这个解决方案将需要添加一个新的依赖项。

和往常一样，本文中的代码可以在 [GitHub](https://web.archive.org/web/20221128235332/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-string-operations-3) 上找到。