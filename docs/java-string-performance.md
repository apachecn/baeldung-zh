# 字符串性能提示

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-string-performance>

## 1。简介

在本教程中，**我们将关注 Java 字符串 API** 的性能方面。

我们将深入研究`String`创建、转换和修改操作，以分析可用选项并比较它们的效率。

我们将要提出的建议不一定适合每一种应用。但是毫无疑问，我们将展示如何在应用程序运行时间至关重要的情况下赢得性能。

## 2。构造一个新的字符串

如你所知，在 Java 中，字符串是不可变的。所以每次我们构造或连接一个`String`对象，Java 就会创建一个新的`String –` 如果在循环中完成，这可能会特别昂贵。

### 2.1 **。使用构造函数**

在大多数情况下，**我们应该避免使用构造函数创建`Strings`，除非我们知道自己在做什么**。

让我们首先在循环内部创建一个`newString `对象，使用`new String()`构造函数，然后使用`=`操作符。

为了编写我们的基准，我们将使用 [JMH](/web/20221129012725/https://www.baeldung.com/java-microbenchmark-harness) (Java 微基准测试工具)工具。

我们的配置:

```java
@BenchmarkMode(Mode.SingleShotTime)
@OutputTimeUnit(TimeUnit.MILLISECONDS)
@Measurement(batchSize = 10000, iterations = 10)
@Warmup(batchSize = 10000, iterations = 10)
public class StringPerformance {
}
```

这里，我们使用的是`SingeShotTime`模式，它只运行一次这个方法。因为我们想测量循环内部的`String`操作的性能，所以有一个`@Measurement`注释可用于此。

要知道，由于 JVM 应用的各种优化，直接在我们的测试中进行**基准测试循环可能会扭曲结果。**

所以我们只计算单个操作，让 JMH 来处理循环。简言之，JMH 通过使用`batchSize`参数来执行迭代。

现在，让我们添加第一个微基准:

```java
@Benchmark
public String benchmarkStringConstructor() {
    return new String("baeldung");
}

@Benchmark
public String benchmarkStringLiteral() {
    return "baeldung";
}
```

在第一个测试中，每次迭代都会创建一个新的对象。在第二个测试中，对象只被创建一次。对于剩余的迭代，从`String's`常量池返回相同的对象。

让我们用循环迭代计数 `= 1,000,000`来运行测试，看看结果:

```java
Benchmark                   Mode  Cnt  Score    Error     Units
benchmarkStringConstructor  ss     10  16.089 ± 3.355     ms/op
benchmarkStringLiteral      ss     10  9.523  ± 3.331     ms/op
```

从`Score`值中，我们可以清楚地看到差异是显著的。

### 2.2。+操作员

让我们来看看动态`String`拼接的例子:

```java
@State(Scope.Thread)
public static class StringPerformanceHints {
    String result = "";
    String baeldung = "baeldung";
}

@Benchmark
public String benchmarkStringDynamicConcat() {
    return result + baeldung;
} 
```

在我们的结果中，我们希望看到平均执行时间。输出数字格式设置为毫秒:

```java
Benchmark                       1000     10,000
benchmarkStringDynamicConcat    47.331   4370.411
```

现在，让我们分析结果。正如我们所见，将`1000`项添加到`state.result`项需要`47.331`毫秒。因此，迭代次数增加 10 次，运行时间增长到`4370.441`毫秒。

**总之，执行时间呈二次方增长。所以 n 次迭代的循环中动态串接的复杂度为`O(n^2)`。**

### 2.3。`String.concat()`

连接`Strings`的另一种方法是使用`concat()`方法:

```java
@Benchmark
public String benchmarkStringConcat() {
    return result.concat(baeldung);
} 
```

输出时间单位是毫秒，迭代次数是 100，000。结果表如下所示:

```java
Benchmark              Mode  Cnt  Score     Error     Units
benchmarkStringConcat    ss   10  3403.146 ± 852.520  ms/op
```

### 2.4。`String.format()`

另一种创建字符串的方法是使用`String.format()`方法。**在幕后，它使用正则表达式来解析输入。**

让我们编写 JMH 测试用例:

```java
String formatString = "hello %s, nice to meet you";

@Benchmark
public String benchmarkStringFormat_s() {
    return String.format(formatString, baeldung);
}
```

之后，我们运行它并查看结果:

```java
Number of Iterations      10,000   100,000   1,000,000
benchmarkStringFormat_s   17.181   140.456   1636.279    ms/op
```

虽然使用`String.format()`的代码看起来更干净，可读性更好，但是在性能方面我们并没有胜出。

### 2.5。`StringBuilder` 和`StringBuffer`

我们已经有一篇[文章](/web/20221129012725/https://www.baeldung.com/java-string-builder-string-buffer)解释`StringBuffer`和`StringBuilder`。所以在这里，我们将只显示关于它们性能的额外信息。`StringBuilder `使用一个可调整大小的数组和一个指示数组中最后一个单元格位置的索引。当数组已满时，它将其大小扩大一倍，并将所有字符复制到新数组中。

考虑到调整大小并不经常发生，**我们可以将每个`append()`操作视为`O(1)`常量时间**。考虑到这一点，整个过程具有`O(n) `的复杂性。

在对`StringBuffer`和`StringBuilder, `进行修改和运行动态连接测试后，我们得到:

```java
Benchmark               Mode  Cnt  Score   Error  Units
benchmarkStringBuffer   ss    10  1.409  ± 1.665  ms/op
benchmarkStringBuilder  ss    10  1.200  ± 0.648  ms/op
```

虽然分数相差不大，但我们可以注意到**比`StringBuilder`快**。

幸运的是，在简单的情况下，我们不需要将一个`String`和另一个`StringBuilder`放在一起。有时候，**和+的静态串联实际上可以代替`StringBuilder`。在幕后，最新的 Java 编译器将调用`StringBuilder.append()`来连接字符串**。

这意味着显著提高性能。

## 3。公用事业运营

### 3.1。`StringUtils.replace()`vs`String.replace()`

有趣的是，替换`String`的 **Apache Commons 版本比 String 自己的`replace()`方法**做得更好。这种差异的答案在于它们的实现。`String.replace()`使用正则表达式模式来匹配`String.`

相比之下，`StringUtils.replace()`在广泛使用`indexOf()`，速度更快。

现在，是时候进行基准测试了:

```java
@Benchmark
public String benchmarkStringReplace() {
    return longString.replace("average", " average !!!");
}

@Benchmark
public String benchmarkStringUtilsReplace() {
    return StringUtils.replace(longString, "average", " average !!!");
}
```

将`batchSize`设置为 100，000，我们给出结果:

```java
Benchmark                     Mode  Cnt  Score   Error   Units
benchmarkStringReplace         ss   10   6.233  ± 2.922  ms/op
benchmarkStringUtilsReplace    ss   10   5.355  ± 2.497  ms/op
```

虽然这两个数字相差不大，但`StringUtils.replace()`的分数更高。当然，这些数字和它们之间的差距可能会根据迭代次数、字符串长度甚至 JDK 版本等参数而变化。

在最新的 JDK 9+(我们的测试在 JDK 10 上运行)版本中，两种实现都有相当好的结果。现在，让我们将 JDK 版本降级到 8，并再次进行测试:

```java
Benchmark                     Mode  Cnt   Score    Error     Units
benchmarkStringReplace         ss   10    48.061   ± 17.157  ms/op
benchmarkStringUtilsReplace    ss   10    14.478   ±  5.752  ms/op
```

现在，性能差异是巨大的，这证实了我们在开始时讨论的理论。

### 3.2。`split()`

在我们开始之前，检查一下 Java 中可用的字符串分割方法会很有用。

当需要用分隔符分割字符串时，我们想到的第一个函数通常是`String.split(regex)`。但是，它会带来一些严重的性能问题，因为它接受正则表达式参数。或者，我们可以使用`StringTokenizer`类将字符串分解成标记。

另一个选择是番石榴的`Splitter` API。最后，如果我们不需要正则表达式的功能，也可以使用传统的`indexOf()`来提高应用程序的性能。

现在，是时候为`String.split()`选项编写基准测试了:

```java
String emptyString = " ";

@Benchmark
public String [] benchmarkStringSplit() {
    return longString.split(emptyString);
}
```

`Pattern.split()`:

```java
@Benchmark
public String [] benchmarkStringSplitPattern() {
    return spacePattern.split(longString, 0);
}
```

`StringTokenizer`:

```java
List stringTokenizer = new ArrayList<>();

@Benchmark
public List benchmarkStringTokenizer() {
    StringTokenizer st = new StringTokenizer(longString);
    while (st.hasMoreTokens()) {
        stringTokenizer.add(st.nextToken());
    }
    return stringTokenizer;
}
```

`String.indexOf()`:

```java
List stringSplit = new ArrayList<>();

@Benchmark
public List benchmarkStringIndexOf() {
    int pos = 0, end;
    while ((end = longString.indexOf(' ', pos)) >= 0) {
        stringSplit.add(longString.substring(pos, end));
        pos = end + 1;
    }
    stringSplit.add(longString.substring(pos));
    return stringSplit;
}
```

番石榴`Splitter`:

```java
@Benchmark
public List<String> benchmarkGuavaSplitter() {
    return Splitter.on(" ").trimResults()
      .omitEmptyStrings()
      .splitToList(longString);
}
```

最后，我们运行并比较`batchSize = 100,000`的结果:

```java
Benchmark                     Mode  Cnt    Score    Error    Units
benchmarkGuavaSplitter         ss   10    4.008  ± 1.836     ms/op
benchmarkStringIndexOf         ss   10    1.144  ± 0.322     ms/op
benchmarkStringSplit           ss   10    1.983  ± 1.075     ms/op
benchmarkStringSplitPattern    ss   10    14.891  ± 5.678    ms/op
benchmarkStringTokenizer       ss   10    2.277  ± 0.448     ms/op
```

正如我们所见，性能最差的是`benchmarkStringSplitPattern`方法，这里我们使用了`Pattern`类。结果，我们可以了解到使用带有`split()`方法的正则表达式类可能会导致多次性能损失。

同样，**我们注意到最快的结果是使用`indexOf() and split()`提供的例子。**

### 3.3。转换为`String`

在这一节中，我们将测量字符串转换的运行时得分。更具体地说，我们将检查`Integer.toString()`连接方法:

```java
int sampleNumber = 100;

@Benchmark
public String benchmarkIntegerToString() {
    return Integer.toString(sampleNumber);
}
```

`String.valueOf()`:

```java
@Benchmark
public String benchmarkStringValueOf() {
    return String.valueOf(sampleNumber);
}
```

`[some integer value] + “”`:

```java
@Benchmark
public String benchmarkStringConvertPlus() {
    return sampleNumber + "";
}
```

`String.format()`:

```java
String formatDigit = "%d";

@Benchmark
public String benchmarkStringFormat_d() {
    return String.format(formatDigit, sampleNumber);
}
```

运行测试后，我们将看到`batchSize = 10,000`的输出:

```java
Benchmark                     Mode  Cnt   Score    Error  Units
benchmarkIntegerToString      ss   10   0.953 ±  0.707  ms/op
benchmarkStringConvertPlus    ss   10   1.464 ±  1.670  ms/op
benchmarkStringFormat_d       ss   10  15.656 ±  8.896  ms/op
benchmarkStringValueOf        ss   10   2.847 ± 11.153  ms/op
```

在分析结果之后，我们看到`Integer.toString()`的测试**获得了`0.953`毫秒**的最佳成绩。相比之下，包含`String.format(“%d”)`的转换表现最差。

这是合乎逻辑的，因为解析格式`String`是一项开销很大的操作。

### 3.4。比较字符串

让我们评估比较`Strings.`的不同方法，迭代次数是`100,000`。

以下是我们对`String.equals()`操作的基准测试:

```java
@Benchmark
public boolean benchmarkStringEquals() {
    return longString.equals(baeldung);
}
```

`String.equalsIgnoreCase()`:

```java
@Benchmark
public boolean benchmarkStringEqualsIgnoreCase() {
    return longString.equalsIgnoreCase(baeldung);
}
```

`String.matches()`:

```java
@Benchmark
public boolean benchmarkStringMatches() {
    return longString.matches(baeldung);
} 
```

`String.compareTo()`:

```java
@Benchmark
public int benchmarkStringCompareTo() {
    return longString.compareTo(baeldung);
}
```

之后，我们运行测试并显示结果:

```java
Benchmark                         Mode  Cnt    Score    Error  Units
benchmarkStringCompareTo           ss   10    2.561 ±  0.899   ms/op
benchmarkStringEquals              ss   10    1.712 ±  0.839   ms/op
benchmarkStringEqualsIgnoreCase    ss   10    2.081 ±  1.221   ms/op
benchmarkStringMatches             ss   10    118.364 ± 43.203 ms/op
```

一如既往，数字说明了一切。由于使用正则表达式来比较相等性，`matches()`花费的时间最长。

相比之下，**`equals()`和`()`是最佳选择**。

### 3.5。`String.matches()`vs`Precompiled Pattern`

现在，让我们分别看看`String.matches()`和`Matcher.matches() `模式。第一个以一个 regexp 作为参数，并在执行前编译它。

所以每次我们调用`String.matches()`，它都会编译`Pattern:`

```java
@Benchmark
public boolean benchmarkStringMatches() {
    return longString.matches(baeldung);
}
```

第二种方法重用`Pattern`对象:

```java
Pattern longPattern = Pattern.compile(longString);

@Benchmark
public boolean benchmarkPrecompiledMatches() {
    return longPattern.matcher(baeldung).matches();
}
```

现在结果是:

```java
Benchmark                      Mode  Cnt    Score    Error   Units
benchmarkPrecompiledMatches    ss   10    29.594  ± 12.784   ms/op
benchmarkStringMatches         ss   10    106.821 ± 46.963   ms/op
```

正如我们所看到的，用预编译的 regexp 进行匹配要快三倍。

### 3.6。检查长度

最后，我们来对比一下`String.isEmpty()`的方法:

```java
@Benchmark
public boolean benchmarkStringIsEmpty() {
    return longString.isEmpty();
}
```

和`String.length()`方法:

```java
@Benchmark
public boolean benchmarkStringLengthZero() {
    return emptyString.length() == 0;
}
```

首先，我们称它们为`longString = “Hello baeldung, I am a bit longer than other Strings in average” String.``batchSize`是`10,000`:

```java
Benchmark                  Mode  Cnt  Score   Error  Units
benchmarkStringIsEmpty       ss   10  0.295 ± 0.277  ms/op
benchmarkStringLengthZero    ss   10  0.472 ± 0.840  ms/op
```

之后，让我们设置`longString = “”`空字符串并再次运行测试:

```java
Benchmark                  Mode  Cnt  Score   Error  Units
benchmarkStringIsEmpty       ss   10  0.245 ± 0.362  ms/op
benchmarkStringLengthZero    ss   10  0.351 ± 0.473  ms/op
```

正如我们注意到的，`benchmarkStringLengthZero()`和`benchmarkStringIsEmpty() `方法在两种情况下的得分大致相同。然而，调用 **`isEmpty()`比检查字符串长度是否为零**要快。

## 4。字符串重复数据删除

从 JDK 8 开始，字符串重复数据删除功能可用于消除内存消耗。简单地说，**这个工具寻找具有相同或重复内容的字符串，将每个不同字符串值的一个副本存储到字符串池**。

目前，有两种方法可以处理`String`副本:

*   手动使用`String.intern()`
*   启用字符串重复数据删除

让我们仔细看看每个选项。

### 4.1。`String.intern()`

在开始之前，阅读一下我们[的文章](/web/20221129012725/https://www.baeldung.com/java-string-pool)中关于手工实习的内容会很有用。**通过`String.intern()`我们可以在全局`String`池**中手动设置`String`对象的引用。

然后，JVM 可以在需要时使用返回引用。从性能的角度来看，我们的应用程序可以通过重用常量池中的字符串引用而受益匪浅。

重要的是要知道， **JVM `String`池对于线程来说不是本地的。我们添加到池中的每个`String`，对于其他线程也是可用的**。

然而，也有严重的缺点:

*   为了正确维护我们的应用程序，我们可能需要设置一个`-XX:StringTableSize` JVM 参数来增加池的大小。JVM 需要重新启动来扩展池大小
*   **手动调用`String.intern()`比较耗时**。它以线性时间算法增长，复杂度为`O(n)`
*   另外，**频繁调用长`String`对象可能会导致内存问题**

为了获得一些经过验证的数字，让我们进行一个基准测试:

```java
@Benchmark
public String benchmarkStringIntern() {
    return baeldung.intern();
}
```

此外，输出分数以毫秒为单位:

```java
Benchmark               1000   10,000  100,000  1,000,000
benchmarkStringIntern   0.433  2.243   19.996   204.373
```

这里的列标题表示从`1000`到`1,000,000`的不同的`iterations`计数。对于每个迭代次数，我们有测试性能分数。正如我们注意到的，除了迭代次数之外，分数也显著增加。

### 4.2。自动启用重复数据删除

首先，**这个选项是 G1 垃圾收集器的一部分。**默认情况下，此功能被禁用。因此，我们需要使用以下命令来启用它:

```java
 -XX:+UseG1GC -XX:+UseStringDeduplication
```

需要注意的是，**启用此选项并不保证`String`会发生重复数据删除**。此外，它不处理年轻的`Strings.`来管理处理的最小年龄`Strings, XX:StringDeduplicationAgeThreshold=3` JVM 选项可用。这里，`3`是默认参数。

## 5。总结

在本教程中，我们试图给出一些在日常编码生活中更有效地使用字符串的提示。

因此，**我们可以强调一些建议，以提高我们的应用程序性能**:

*   **当连接字符串时，`StringBuilder`是想到的最方便的选项**。然而，对于小字符串，`+ `操作具有几乎相同的性能。在幕后，Java 编译器可能会使用`StringBuilder `类来减少字符串对象的数量
*   为了将值转换成字符串，`[some type].toString()`(例如`Integer.toString()`)比`String.valueOf()`工作得更快。因为这种差异并不显著，所以我们可以自由地使用`String.valueOf()`来消除对输入值类型的依赖
*   **说到字符串比较，目前为止没有什么能打败`String.equals()`**
*   `String`重复数据删除提高了大型多线程应用的性能。但是过度使用`String.intern()`可能会导致严重的内存泄漏，降低应用程序的速度
*   **为了分裂琴弦，我们应该使用`indexOf()`来赢得演奏**。然而，在一些非关键的情况下，`String.split()`函数可能是一个很好的选择
*   使用`Pattern.match()`字符串可以显著提高性能
*   **`String.isEmpty()`比弦`.length() ==0`** 快

另外，**请记住，我们在这里给出的数字只是 JMH 基准测试结果**——因此您应该始终在您自己的系统和运行时范围内进行测试，以确定这些优化的影响。

最后，和往常一样，讨论中使用的代码可以在 GitHub 上找到[。](https://web.archive.org/web/20221129012725/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-strings)