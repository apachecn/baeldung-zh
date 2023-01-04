# Java 流的字符串操作

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-stream-operations-on-strings>

## 1。概述

Java 8 引入了新的`Stream` API，让我们以声明的方式处理数据。

在这篇简短的文章中，我们将学习如何使用`Stream` API 将逗号分隔的`String`拆分成一列`Strings`，以及如何将`String`数组加入逗号分隔的`String`。

我们还将看看如何使用`Stream` API 将字符串数组转换成 map。

几乎所有时间我们都面临这样的情况，我们需要迭代一些`Java Collections`并基于一些过滤逻辑过滤`Collection`。在这种情况下的传统方法中，我们会使用许多循环和 if-else 操作来获得想要的结果。

如果你想了解更多关于`Stream` API 的内容，请查看[这篇文章](/web/20221207100640/https://www.baeldung.com/java-8-streams)。

## 2。用`Stream` API 连接字符串

让我们使用`Stream` API 来创建一个函数，它将把一个`String`数组加入到一个逗号分隔的`String`中:

```
public static String join(String[] arrayOfString){
    return Arrays.asList(arrayOfString)
      .stream()
      //.map(...)
      .collect(Collectors.joining(","));
}
```

此处需要注意的要点:

*   `stream()`函数将任何`Collection`转换成数据流
*   `map()`功能用于处理数据
*   还有另一个名为`filter()`的函数，我们可以在其中包含过滤标准

可能会有这样的情况，我们可能想要加入一个带有固定前缀和后缀的`String`。使用`Stream` API，我们可以通过以下方式实现:

```
public static String joinWithPrefixPostfix(String[] arrayOfString){
    return Arrays.asList(arrayOfString)
      .stream()
      //.map(...)
      .collect(Collectors.joining(",","[","]"));
}
```

正如我们在`Collectors.joining()`方法中看到的，我们将前缀声明为`‘[‘`，后缀声明为`‘]'`；因此生成的`String`将用声明的`[…..]`格式创建。

## 3。使用`Stream` API 拆分`Strings`

现在，让我们创建一个函数，它使用`Stream` API 将逗号分隔的`String`拆分成一系列的`String`:

```
public static List<String> split(String str){
    return Stream.of(str.split(","))
      .map (elem -> new String(elem))
      .collect(Collectors.toList());
}
```

也可以使用`Stream` API 直接将`String`转换成`Character`列表:

```
public static List<Character> splitToListOfChar(String str) {
    return str.chars()
      .mapToObj(item -> (char) item)
      .collect(Collectors.toList());
}
```

这里要注意的一个有趣的事实是，`chars()`方法将`String`转换成一个`Integer`流，其中每个`Integer`值表示每个`Char`序列的`ASCII`值。这就是为什么我们需要在`mapToObj()`方法中显式地对 mapper 对象进行类型转换。

## 4.使用`Stream` API 从`String` 数组到`Map`

我们还可以使用`split `和`Collectors.toMap`将`String`数组转换为 map，假设数组中的每一项都包含一个由分隔符连接的键值实体:

```
public static Map<String, String> arrayToMap(String[] arrayOfString) {
	return Arrays.asList(arrayOfString)
	  .stream()
	  .map(str -> str.split(":"))
	  .collect(toMap(str -> str[0], str -> str[1]));
}
```

这里，`“:”`是字符串数组中所有元素的键值分隔符。

**请记住，为了避免编译错误，我们需要确保代码使用 Java 1.8** 编译。为此，我们需要在`pom.xml`中添加以下插件:

```
<build>
    <plugins>
      <plugin>
        <groupId>org.apache.maven.plugins</groupId>
        <artifactId>maven-compiler-plugin</artifactId>
        <version>3.3</version>
        <configuration>
          <source>1.8</source>
          <target>1.8</target>
        </configuration>
      </plugin>
    </plugins>        
</build>
```

## 5。测试

既然我们已经完成了函数的创建，让我们创建测试用例来验证结果。

首先，让我们测试一下简单的连接方法:

```
@Test
public void givenArray_transformedToStream_convertToString() {
    String[] programmingLanguages = {"java", "python", "nodejs", "ruby"};
    String expectation = "java,python,nodejs,ruby";

    String result  = JoinerSplitter.join(programmingLanguages);
    assertEquals(result, expectation);
}
```

接下来，让我们创建另一个来测试我们简单的分割功能:

```
@Test
public void givenString_transformedToStream_convertToList() {
    String programmingLanguages = "java,python,nodejs,ruby";

    List<String> expectation = new ArrayList<>();
    expectation.add("java");
    expectation.add("python");
    expectation.add("nodejs");
    expectation.add("ruby");

    List<String> result  = JoinerSplitter.split(programmingLanguages);

    assertEquals(result, expectation);
}
```

最后，让我们测试一下我们的`String`数组的映射功能:

```
@Test
public void givenStringArray_transformedToStream_convertToMap() {

    String[] programming_languages = new String[] {"language:java","os:linux","editor:emacs"};

    Map<String,String> expectation=new HashMap<>();
    expectation.put("language", "java");
    expectation.put("os", "linux");
    expectation.put("editor", "emacs");

    Map<String, String> result = JoinerSplitter.arrayToMap(programming_languages);
    assertEquals(result, expectation);

}
```

同样，我们需要创建其余的测试用例。

## 6。结论

API 为我们提供了复杂的数据处理技术。就多线程环境中的堆内存管理而言，这种新的代码编写方式非常高效。

像往常一样，完整的源代码可以在 Github 上获得[。](https://web.archive.org/web/20221207100640/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-string-operations)