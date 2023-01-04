# 将正则表达式模式预编译成模式对象

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-regex-pre-compile>

## 1.概观

在本教程中，我们将看到预编译正则表达式模式的好处以及 Java 8 和 11 中引入的新方法。

这不是一个正则表达式操作指南，但是我们有一个优秀的 Java 正则表达式 API 指南。

## 2.利益

重用不可避免地会带来性能提升，因为我们不需要一次又一次地创建和重新创建相同对象的实例。因此，我们可以假设重用和性能经常是联系在一起的。

让我们来看看这个原则，因为它与`Pattern#compile.` ****有关。我们将使用一个简单的基准**:**

 **1.  我们有一个从 1 到 5，000，000 的数字列表
2.  我们的正则表达式将匹配偶数

因此，让我们用下面的 Java 正则表达式来测试解析这些数字:

*   `String.matches(regex)`
*   `Pattern.matches(regex, charSequence)`
*   `Pattern.compile(regex).matcher(charSequence).matches()`
*   预编译的正则表达式有许多对`preCompiledPattern.matcher(value).matches()`的调用
*   带有一个`Matcher`实例和许多对`matcherFromPreCompiledPattern.reset(value).matches()`的调用的预编译正则表达式

实际上，如果我们看一下`String#matches`的实现:

```
public boolean matches(String regex) {
    return Pattern.matches(regex, this);
}
```

并且在`Pattern#matches`:

```
public static boolean matches(String regex, CharSequence input) {
    Pattern p = compile(regex);
    Matcher m = p.matcher(input);
    return m.matches();
}
```

然后，我们可以想象**前三个表达式会有类似的表现。**那是因为第一个表达式调用第二个，第二个调用第三个。

第二点是这些方法没有重用创建的`Pattern`和`Matcher`实例。正如我们将在基准测试中看到的，**这会使性能降低六倍**:

```
 @Benchmark
public void matcherFromPreCompiledPatternResetMatches(Blackhole bh) {
    for (String value : values) {
        bh.consume(matcherFromPreCompiledPattern.reset(value).matches());
    }
}

@Benchmark
public void preCompiledPatternMatcherMatches(Blackhole bh) {
    for (String value : values) {
        bh.consume(preCompiledPattern.matcher(value).matches());
    }
}

@Benchmark
public void patternCompileMatcherMatches(Blackhole bh) {
    for (String value : values) {
        bh.consume(Pattern.compile(PATTERN).matcher(value).matches());
    }
}

@Benchmark
public void patternMatches(Blackhole bh) {
    for (String value : values) {
        bh.consume(Pattern.matches(PATTERN, value));
    }
}

@Benchmark
public void stringMatchs(Blackhole bh) {
    Instant start = Instant.now();
    for (String value : values) {
        bh.consume(value.matches(PATTERN));
    }
} 
```

看看基准测试的结果，毫无疑问，**预编译的`Pattern`和重用的`Matcher`以比**快六倍多的结果胜出:

```
Benchmark                                                               Mode  Cnt     Score     Error  Units
PatternPerformanceComparison.matcherFromPreCompiledPatternResetMatches  avgt   20   278.732 ±  22.960  ms/op
PatternPerformanceComparison.preCompiledPatternMatcherMatches           avgt   20   500.393 ±  34.182  ms/op
PatternPerformanceComparison.stringMatchs                               avgt   20  1433.099 ±  73.687  ms/op
PatternPerformanceComparison.patternCompileMatcherMatches               avgt   20  1774.429 ± 174.955  ms/op
PatternPerformanceComparison.patternMatches                             avgt   20  1792.874 ± 130.213  ms/op
```

**除了性能时间，我们还有创建的对象数量**:

*   前三种形式:
    *   创建了 5，000，000 个`Pattern`实例
    *   创建了 5，000，000 个`Matcher`实例
*   `preCompiledPattern.matcher(value).matches()`
    *   1 个`Pattern`实例已创建
    *   创建了 5，000，000 个`Matcher`实例
*   `matcherFromPreCompiledPattern.reset(value).matches()`
    *   1 个`Pattern`实例已创建
    *   1 个`Matcher`实例已创建

因此，不要将我们的正则表达式委托给`String#matches`或`Pattern#matches`，这样总是会创建`Pattern`和`Matcher` 实例。我们应该预编译我们的正则表达式以获得更好的性能，并且创建更少的对象。

要了解更多关于正则表达式的性能，请查看我们的[Java 正则表达式性能概述。](/web/20220625072553/https://www.baeldung.com/java-regex-performance)

## 3.新方法

自从引入功能接口和流之后，重用变得更加容易。

**`Pattern`类在新的 Java 版本**中得到了发展，提供了与 streams 和 lambdas 的集成。

### 3.1.Java 8

**Java 8 引入了两个新方法:`splitAsStream`和`asPredicate`。**

让我们看一下`splitAsStream`的一些代码，它从给定的输入序列中围绕模式匹配创建一个流:

```
@Test
public void givenPreCompiledPattern_whenCallSplitAsStream_thenReturnArraySplitByThePattern() {
    Pattern splitPreCompiledPattern = Pattern.compile("__");
    Stream<String> textSplitAsStream = splitPreCompiledPattern.splitAsStream("My_Name__is__Fabio_Silva");
    String[] textSplit = textSplitAsStream.toArray(String[]::new);

    assertEquals("My_Name", textSplit[0]);
    assertEquals("is", textSplit[1]);
    assertEquals("Fabio_Silva", textSplit[2]);
}
```

`asPredicate`方法创建了一个谓词，其行为就好像它从输入序列中创建了一个匹配器，然后调用 find:

```
string -> matcher(string).find();
```

让我们创建一个模式，该模式匹配一个列表中的姓名，该列表中的姓名至少包含三个字母:

```
@Test
public void givenPreCompiledPattern_whenCallAsPredicate_thenReturnPredicateToFindPatternInTheList() {
    List<String> namesToValidate = Arrays.asList("Fabio Silva", "Mr. Silva");
    Pattern firstLastNamePreCompiledPattern = Pattern.compile("[a-zA-Z]{3,} [a-zA-Z]{3,}");

    Predicate<String> patternsAsPredicate = firstLastNamePreCompiledPattern.asPredicate();
    List<String> validNames = namesToValidate.stream()
        .filter(patternsAsPredicate)
        .collect(Collectors.toList());

    assertEquals(1,validNames.size());
    assertTrue(validNames.contains("Fabio Silva"));
}
```

### 3.2.Java 11

**Java 11 引入了`asMatchPredicate`方法**,该方法创建一个谓词，其行为就像从输入序列创建一个匹配器，然后调用 matches:

```
string -> matcher(string).matches();
```

让我们创建一个模式，该模式匹配一个列表中的名字，该列表只有名字和姓氏，每个名字至少有三个字母:

```
@Test
public void givenPreCompiledPattern_whenCallAsMatchPredicate_thenReturnMatchPredicateToMatchesPattern() {
    List<String> namesToValidate = Arrays.asList("Fabio Silva", "Fabio Luis Silva");
    Pattern firstLastNamePreCompiledPattern = Pattern.compile("[a-zA-Z]{3,} [a-zA-Z]{3,}");

    Predicate<String> patternAsMatchPredicate = firstLastNamePreCompiledPattern.asMatchPredicate();
    List<String> validatedNames = namesToValidate.stream()
        .filter(patternAsMatchPredicate)
        .collect(Collectors.toList());

    assertTrue(validatedNames.contains("Fabio Silva"));
    assertFalse(validatedNames.contains("Fabio Luis Silva"));
}
```

## 4.结论

在本教程中，我们看到**使用预编译模式给我们带来了更好的性能**。

我们还了解了 JDK 8 号和 JDK 11 号推出的三种新方法，它们让我们的生活变得更加轻松。

这些例子的代码可以在 GitHub 上找到，JDK 11 的代码片段在`[core-java-11](https://web.archive.org/web/20220625072553/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-11)`中，其他的在`[core-java-regex](https://web.archive.org/web/20220625072553/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-regex)`中。**