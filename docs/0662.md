# Java StringTokenizer 快速指南

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-stringtokenizer>

## 1。概述

在这篇简短的文章中，我们将探索 Java 中的一个基本类——**`[StringTokenizer](https://web.archive.org/web/20221129004745/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/StringTokenizer.html)`**。

## 2。`StringTokenizer`

**`StringTokenizer`类帮助我们将 `Strings`分割成多个令牌。**

[`StreamTokenizer`](https://web.archive.org/web/20221129004745/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/io/StreamTokenizer.html) 提供了类似的功能，但是`StringTokenizer`的标记化方法比 [`StreamTokenizer`](https://web.archive.org/web/20221129004745/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/io/StreamTokenizer.html) 类使用的方法简单得多。`StringTokenizer`的方法不区分标识符、数字和引用的字符串，也不识别和跳过注释。

分隔符集(分隔标记的字符)可以在创建时指定，也可以基于每个标记指定。

## 3。使用`StringTokenizer`

使用`StringTokenizer`的最简单的例子是根据指定的分隔符分割`String`。

在这个简单的例子中，我们将分割参数字符串并将标记添加到一个列表中`:`

```
public List<String> getTokens(String str) {
    List<String> tokens = new ArrayList<>();
    StringTokenizer tokenizer = new StringTokenizer(str, ",");
    while (tokenizer.hasMoreElements()) {
        tokens.add(tokenizer.nextToken());
    }
    return tokens;
} 
```

请注意我们是如何根据分隔符'`,`'将`String`拆分成令牌列表的。然后在循环中，使用`tokens.add()`方法；我们将每个令牌添加到`ArrayList.`

例如，如果用户输入“`Welcome,to,baeldung.com`”，这个方法应该返回一个包含三个单词片段的列表，如“`Welcome`”、“`to`”和“`baeldung.com`”。

### 3.1。Java 8 方法

由于 `StringTokenizer`实现了`Enumeration<Object>`接口，我们可以用 J `ava`的`Collections`接口来使用它。

如果我们考虑前面的例子，我们可以使用`Collections.list()`方法和`Stream` API 检索相同的令牌集:

```
public List<String> getTokensWithCollection(String str) {
    return Collections.list(new StringTokenizer(str, ",")).stream()
      .map(token -> (String) token)
      .collect(Collectors.toList());
}
```

这里，我们将`StringTokenizer`本身作为参数传递给`Collections.list()`方法。

**这里要注意的一点是，由于`Enumeration`是一个`Object`类型，我们需要将令牌类型转换为`String` 类型**(即取决于实现；如果我们使用`Integer/Float`的`List`，那么我们将需要使用`Integer/Float`进行类型转换。

### 3.2。`StringTokenizer`的变种

除了默认的构造函数之外，`StringTokenizer`还有两个重载的构造函数:`StringTokenizer(String str)`和`StringTokenizer(String str, String delim, boolean returnDelims):`

**`StringTokenizer(String str, String delim, boolean returnDelims)`** 多带一个`boolean`输入。如果`boolean`的值是`true`，那么`StringTokenizer`将分隔符本身视为一个令牌，并将其添加到其内部令牌池中。

**`StringTokenizer(String`【str】**是前面例子的快捷方式；它在内部调用另一个构造函数，硬编码的分隔符为`” \t\n\r\f”`，布尔值为`false.`

### 3.3。令牌定制

`StringTokenizer`还附带了一个重载的`nextToken()`方法，它接受一个字符串片段作为输入。这个`String`片段充当了一组额外的分隔符；基于这些令牌再次进行重组。

例如，如果我们可以在`nextToken()`方法中传递“`e`”，根据分隔符“`e`”进一步拆分字符串:

```
tokens.add(tokenizer.nextToken("e"));
```

因此，对于给定的'`Hello,baeldung.com`'字符串，我们将产生以下令牌:

```
H
llo
ba
ldung.com
```

### 3.4。令牌长度

为了计算可用令牌的数量，我们可以使用`StringTokenizer`的`countTokens` 方法:

```
int tokenLength = tokens.countTokens();
```

### 3.5。从 CSV 文件中读取

现在，让我们尝试在一个真实的用例中使用`StringTokenizer`。

有些情况下，我们试图从 CSV 文件中读取数据，并根据用户给定的分隔符解析数据。

使用`StringTokenizer`，我们可以轻松到达那里:

```
public List<String> getTokensFromFile( String path , String delim ) {
    List<String> tokens = new ArrayList<>();
    String currLine = "";
    StringTokenizer tokenizer;
    try (BufferedReader br = new BufferedReader(
        new InputStreamReader(Application.class.getResourceAsStream( 
          "/" + path )))) {
        while (( currLine = br.readLine()) != null ) {
            tokenizer = new StringTokenizer( currLine , delim );
            while (tokenizer.hasMoreElements()) {
                tokens.add(tokenizer.nextToken());
            }
        }
    } catch (IOException e) {
        e.printStackTrace();
    }
    return tokens;
}
```

这里，函数有两个参数；一个作为 CSV 文件名(即从 resources `[src -> main -> resources]`文件夹中读取),另一个作为分隔符。

基于这两个参数，CSV 数据被逐行读取，每一行都使用`StringTokenizer`进行标记化。

例如，我们在 CSV 中放入了以下内容:

```
1|IND|India
2|MY|Malaysia
3|AU|Australia
```

因此，应该生成以下令牌:

```
1
IND
India
2
MY
Malaysia
3
AU
Australia
```

### 3.6。测试

现在，让我们创建一个快速测试用例:

```
public class TokenizerTest {

    private MyTokenizer myTokenizer = new MyTokenizer();
    private List<String> expectedTokensForString = Arrays.asList(
      "Welcome" , "to" , "baeldung.com" );
    private List<String> expectedTokensForFile = Arrays.asList(
      "1" , "IND" , "India" , 
      "2" , "MY" , "Malaysia" , 
      "3", "AU" , "Australia" );

    @Test
    public void givenString_thenGetListOfString() {
        String str = "Welcome,to,baeldung.com";
        List<String> actualTokens = myTokenizer.getTokens( str );

        assertEquals( expectedTokensForString, actualTokens );
    }

    @Test
    public void givenFile_thenGetListOfString() {
        List<String> actualTokens = myTokenizer.getTokensFromFile( 
          "data.csv", "|" );

        assertEquals( expectedTokensForFile , actualTokens );
    }
}
```

## 4。结论

在这个快速教程中，我们看了一些使用核心 Java `StringTokenizer`的实际例子。

像往常一样，完整的源代码可以在 GitHub 上获得[。](https://web.archive.org/web/20221129004745/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-string-apis)