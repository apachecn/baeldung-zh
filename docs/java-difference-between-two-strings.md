# 在 Java 中寻找两个字符串的区别

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-difference-between-two-strings>

## 1。概述

这个快速教程将展示如何使用 Java 找到两个字符串之间的区别。

对于本教程，我们将使用**两个现有的 Java 库**并比较它们解决这个问题的方法。

## 2。问题

让我们考虑下面的需求:我们想找出字符串`“` ABCDELMN "和" ABCFGLMN "之间的区别。

根据我们需要的输出格式，忽略编写自定义代码来实现这一点的可能性，我们发现有两种主要的选择。

第一个是由谷歌编写的名为`[diff-match-patch](https://web.archive.org/web/20221127015501/https://github.com/google/diff-match-patch).`的库，正如他们所声称的，这个库提供了**用于同步纯文本**的健壮算法。

另一个选项是 Apache Commons Lang 的`[StringUtils](https://web.archive.org/web/20221127015501/https://github.com/apache/commons-lang/blob/master/src/main/java/org/apache/commons/lang3/StringUtils.java) `类。

下面就来探讨一下这两者的区别。

## 3。`diff-match-patch`

出于本文的目的，我们将使用原始 Google 库的一个分支[，因为原始库的工件没有在 Maven Central 上发布。此外，一些类名不同于原始代码库，更符合 Java 标准。](https://web.archive.org/web/20221127015501/https://search.maven.org/search?q=org.bitbucket.cowwoc%20diff-match-patch)

首先，我们需要在我们的`pom.xml `文件中包含它的依赖关系:

```java
<dependency>
    <groupId>org.bitbucket.cowwoc</groupId>
    <artifactId>diff-match-patch</artifactId>
    <version>1.2</version>
</dependency>
```

然后，让我们考虑这个代码:

```java
String text1 = "ABCDELMN";
String text2 = "ABCFGLMN";
DiffMatchPatch dmp = new DiffMatchPatch();
LinkedList<Diff> diff = dmp.diffMain(text1, text2, false);
```

如果我们运行上面的代码——它产生了`text1`和`text2`之间的差异——打印变量`diff`将产生以下输出:

```java
[Diff(EQUAL,"ABC"), Diff(DELETE,"DE"), Diff(INSERT,"FG"), Diff(EQUAL,"LMN")]
```

事实上，输出将是一个由`Diff`对象组成的**列表，每个对象都是由操作类型** ( `INSERT`、`DELETE`或`EQUAL`)和**与操作**相关联的文本部分构成的**。**

当运行`text2`和`text1,` 之间的 diff 时，我们将得到以下结果:

```java
[Diff(EQUAL,"ABC"), Diff(DELETE,"FG"), Diff(INSERT,"DE"), Diff(EQUAL,"LMN")]
```

## 4。`StringUtils`

来自`Apache Commons`的类有一个**更简单的方法**。

首先，我们将把 Apache Commons Lang 依赖关系添加到我们的`pom.xml `文件中:

```java
<dependency>
    <groupId>org.apache.commons</groupId>
    <artifactId>commons-lang3</artifactId>
    <version>3.12.0</version>
</dependency>
```

然后，为了找出 Apache Commons 的两个文本之间的区别，我们可以调用`StringUtils#Difference`:

```java
StringUtils.difference(text1, text2)
```

产生的输出**将是一个简单的字符串**:

```java
FGLMN
```

而运行`text2`和`text1`之间的差异将返回:

```java
DELMN
```

这个简单的方法**可以使用** **`StringUtils.indexOfDifference()`** 来增强，其中**将返回两个字符串开始不同的** **索引**(在我们的例子中，是字符串的第四个字符)。这个索引可用于**获得原始字符串**的子字符串，以向**显示两个输入**之间的共同之处，以及不同之处。

## 5。性能

对于我们的基准，我们生成一个 10，000 个字符串的列表，其中**是 10 个字符的固定部分**，后面是 **20 个随机字母字符**。

然后我们遍历列表，在列表的`n^(th)`元素和`n+1^(th)`元素之间执行 diff:

```java
@Benchmark
public int diffMatchPatch() {
    for (int i = 0; i < inputs.size() - 1; i++) {
        diffMatchPatch.diffMain(inputs.get(i), inputs.get(i + 1), false);
    }
    return inputs.size();
}
```

```java
@Benchmark
public int stringUtils() {
    for (int i = 0; i < inputs.size() - 1; i++) {
        StringUtils.difference(inputs.get(i), inputs.get(i + 1));
    }
    return inputs.size();
}
```

最后，让我们运行基准测试并比较这两个库:

```java
Benchmark                                   Mode  Cnt    Score   Error  Units
StringDiffBenchmarkUnitTest.diffMatchPatch  avgt   50  130.559 ± 1.501  ms/op
StringDiffBenchmarkUnitTest.stringUtils     avgt   50    0.211 ± 0.003  ms/op
```

## 6。结论

就纯粹的执行速度而言， **`StringUtils`显然更具性能**，尽管它只返回两个字符串开始不同的子串。

同时，`Diff-Match-Patch`提供了一个**更彻底的比较结果**，代价是性能。

GitHub 上的[提供了这些例子和片段的实现。](https://web.archive.org/web/20221127015501/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-string-algorithms-2)