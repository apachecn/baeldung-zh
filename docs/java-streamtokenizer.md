# StreamTokenizer 指南

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-streamtokenizer>

## 1.介绍

在本教程中，我们将展示如何使用 Java `StreamTokenizer`类将字符流解析成标记。

## 2.`StreamTokenizer`

`StreamTokenizer`类逐字符读取流。它们中的每一个都可以有零个或多个以下属性:空白、字母、数字、字符串引用或注释字符。

现在，我们需要了解默认配置。我们有以下类型的角色:

*   `Word characters`:类似于‘A’到‘Z’和‘A’到‘Z’的范围
*   `Numeric characters` : 0，1，…，9
*   `Whitespace characters`:从 0 到 32 的 ASCII 值
*   `Comment character` : /
*   `String quote characters`:‘和’

**注意，行尾被视为空白，而不是单独的标记，**并且默认情况下不识别 C/C++样式的注释。

这个类拥有一组重要的字段:

*   `TT_EOF` –表示流结束的常数
*   `TT_EOL` –表示行尾的常数
*   `TT_NUMBER` –表示数字令牌的常数
*   `TT_WORD` –表示单词标记的常数

## 3.默认配置

这里，我们将创建一个例子来理解`StreamTokenizer`机制。我们首先创建这个类的一个实例，然后调用`nextToken()`方法，直到它返回`TT_EOF`值:

```java
private static final int QUOTE_CHARACTER = '\'';
private static final int DOUBLE_QUOTE_CHARACTER = '"';

public static List<Object> streamTokenizerWithDefaultConfiguration(Reader reader) throws IOException {
    StreamTokenizer streamTokenizer = new StreamTokenizer(reader);
    List<Object> tokens = new ArrayList<Object>();

    int currentToken = streamTokenizer.nextToken();
    while (currentToken != StreamTokenizer.TT_EOF) {

        if (streamTokenizer.ttype == StreamTokenizer.TT_NUMBER) {
            tokens.add(streamTokenizer.nval);
        } else if (streamTokenizer.ttype == StreamTokenizer.TT_WORD
            || streamTokenizer.ttype == QUOTE_CHARACTER
            || streamTokenizer.ttype == DOUBLE_QUOTE_CHARACTER) {
            tokens.add(streamTokenizer.sval);
        } else {
            tokens.add((char) currentToken);
        }

        currentToken = streamTokenizer.nextToken();
    }

    return tokens;
}
```

测试文件只包含:

```java
3 quick brown foxes jump over the "lazy" dog!
#test1
//test2
```

现在，如果我们打印出数组的内容，我们会看到:

```java
Number: 3.0
Word: quick
Word: brown
Word: foxes
Word: jump
Word: over
Word: the
Word: lazy
Word: dog
Ordinary char: !
Ordinary char: #
Word: test1
```

为了更好地理解这个例子，我们需要解释一下`StreamTokenizer.ttype`、`StreamTokenizer.nval` 和`StreamTokenizer.sval` 字段。

**`ttype`字段包含刚刚读取的令牌的类型。**可能是`TT_EOF`、`TT_EOL`、`TT_NUMBER`、`TT_WORD`。但是，**对于带引号的字符串令牌，其值是引号字符的 ASCII 值**。此外，如果令牌是一个普通字符，如`‘!'`，没有属性，那么`ttype`将用该字符的 ASCII 值填充。

接下来，**我们使用`sval`字段来获取令牌，前提是它是一个`TT_WORD`** ，也就是一个单词令牌。但是，如果我们处理的是带引号的字符串标记——比如说`“lazy” –` ,那么这个字段包含字符串的主体。

最后，**我们使用了`nval`字段来获取令牌，只有当它是一个数字令牌时，才使用`TT_NUMBER`。**

## 4.自定义配置

这里，我们将更改默认配置并创建另一个示例。

首先，**我们将使用`wordChars(int low, int hi)`** 方法设置一些额外的单词字符。然后，**我们将使评论角色('/')成为普通角色**，并将`‘#'`提升为新的评论角色。

最后，**在`eolIsSignificant(boolean flag)`方法的帮助下，我们将把行尾看作一个记号字符**。

我们只需要在`streamTokenizer` 对象上调用这些方法:

```java
public static List<Object> streamTokenizerWithCustomConfiguration(Reader reader) throws IOException {
    StreamTokenizer streamTokenizer = new StreamTokenizer(reader);
    List<Object> tokens = new ArrayList<Object>();

    streamTokenizer.wordChars('!', '-');
    streamTokenizer.ordinaryChar('/');
    streamTokenizer.commentChar('#');
    streamTokenizer.eolIsSignificant(true);

    // same as before

    return tokens;
}
```

这里我们有一个新的输出:

```java
// same output as earlier
Word: "lazy"
Word: dog!
Ordinary char: 

Ordinary char: 

Ordinary char: /
Ordinary char: /
Word: test2
```

注意，双引号变成了标记的一部分，换行符不再是空白字符，而是普通字符，因此是单字符标记。

此外，现在会跳过“#”字符后面的字符，并且“/”是一个普通字符。

我们也可以**用`quoteChar(int ch)`方法**改变引号字符，或者甚至通过调用`whitespaceChars(int low, int hi)`方法改变空白字符。因此，可以通过调用不同组合**的`StreamTokenizer`的方法进行进一步定制。**

## 5.结论

在本教程中，**我们已经看到了如何使用`StreamTokenizer`类**将一串字符解析成记号。我们已经了解了默认机制，并用默认配置创建了一个示例。

最后，我们改变了默认参数，我们注意到了`StreamTokenizer`类的灵活性。

像往常一样，代码可以在 GitHub 上找到[。](https://web.archive.org/web/20220524005427/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-string-apis)