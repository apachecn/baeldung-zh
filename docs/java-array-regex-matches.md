# 从正则表达式匹配创建 Java 数组

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-array-regex-matches>

## 1.概观

在本教程中，我们将学习如何从正则表达式( [regex](/web/20221208143956/https://www.baeldung.com/regular-expressions-java) )输出中创建一个数组。

## 2.介绍

对于我们的例子，让我们解析一个长字符串。我们会找到 10 位数电话号码的模式。然后我们将输出生成一个数组。

Oracle 为其 regex 实现提供了`java.util.regex`包。在我们的演示中，我们将使用这个包中可用的类。一旦找到匹配项，我们将获取输出并创建一个数组。

数组是固定大小的变量。在使用它们之前，我们必须声明它们的大小。如果数组没有正确实现，也有可能浪费内存。出于这个原因，我们从一个`List`开始，然后动态地将`List`转换成一个数组。

## 3.履行

让我们通过代码一步一步地实现这个解决方案。首先，让我们创建一个 [`ArrayList`](/web/20221208143956/https://www.baeldung.com/java-arraylist) 来存储匹配:

```java
List<String> matchesList = new ArrayList<String>();
```

我们将存储一个嵌入了电话号码的长字符串，如下所示:

```java
String stringToSearch =
  "7801111111blahblah  780222222 mumbojumbo7803333333 thisnthat 7804444444";
```

我们使用`compile()`方法，一个静态工厂方法在 [`Pattern`](/web/20221208143956/https://www.baeldung.com/java-regex-pre-compile) 类中。它返回 regex 的一个等价的`Pattern`对象:

```java
Pattern p1 = Pattern.compile("780{1}\\d{7}");
```

一旦我们有了一个`Pattern`对象，我们使用` the matcher()`方法创建一个`Matcher`对象:

```java
Matcher m1 = p1.matcher(stringToSearch); 
```

这里我们可以使用 Matcher 类中的`find()`方法，如果找到匹配项，它将返回一个`boolean`:

```java
while (m1.find()) {
    matchesList.add(m1.group());
}
```

我们刚刚使用的 `group()`方法在`Matcher`类中。它产生一个代表匹配模式的`String`。

为了将`matchesList`转换成一个数组，我们需要找到匹配的项目数。然后，当我们创建一个新数组来存储结果时，我们使用它:

```java
int sizeOfNewArray = matchesList.size(); 
String newArrayOfMatches[] = new String[sizeOfNewArray]; 
matchesList.toArray(newArrayOfMatches);
```

现在让我们通过一些例子来看看我们的代码是如何工作的。如果我们传递一个带有四个匹配模式的`String`，我们的代码会产生一个带有这四个匹配的新的`String`数组:

```java
RegexMatches rm = new RegexMatches();
String actual[] = rm.regexMatch("7801111211fsdafasdfa  7802222222  sadfsadfsda7803333333 sadfdasfasd 7804444444");

assertArrayEquals(new String[] {"7801111211", "7802222222", "7803333333", "7804444444"}, actual, "success");
```

如果我们传递一个没有匹配的`String`，我们得到一个空的`String`数组:

```java
String actual[] = rm.regexMatch("78011111fsdafasdfa  780222222  sadfsadfsda78033333 sadfdasfasd 7804444");

assertArrayEquals(new String[] {}, actual, "success");
```

## 4.结论

在本教程中，我们学习了如何在 Java 中查找文本串中的模式。我们还找到了一种在数组中列出输出的方法。

GitHub 上的[提供了源代码。](https://web.archive.org/web/20221208143956/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-regex-2)