# Java 缺少 Return 语句

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-missing-return-statement>

## 1.概观

在本教程中，我们来看看 Java 开发过程中的一个常见错误。通常，初学者会面临这个问题，Java 应用程序中的遗漏返回语句错误。

**遗漏返回语句错误是编译时错误**。它在编译阶段被抛出。现代的 ide 会即时处理这个错误。因此，这种类型的错误易于检测。

主要原因是:

*   return 语句只是被错误地省略了
*   该方法不返回任何值，但是方法签名中没有声明 void 类型

## 2.缺少返回语句

首先，我们来看几个例子。这些例子都与误省略的 return 语句有关。然后，我们将寻找方法签名中缺失的 void 类型的例子。每个例子都展示了我们如何解决 java 缺少 return 语句的错误。

### 2.1.省略 Return 语句

接下来，让我们定义一个简单的`pow` 方法:

```java
public int pow(int number) {
    int pow = number * number;
}
```

由于编译了前面的代码，我们得到了:

```java
java: missing return statement
```

为了解决这个问题，我们只需在`pow` 变量后添加一个 return 语句:

```java
public int pow(int number) {
    int pow = number * number;
    return pow;
}
```

因此，如果我们调用方法`pow,` ,我们会得到预期的结果。

类似地，但是对于条件结构，会出现以下错误:

```java
public static String checkNumber(int number) {
    if (number == 0) {
        return "It's equals to zero";
    }
    for (int i = 0; i < number; i++) {
        if (i > 100) {
            return "It's a big number";
        }
    }
}
```

上面的代码检查一个输入的数字。首先，将输入的数字与 0 进行比较。如果条件为真，则返回一个字符串值。然后，如果数字大于 0，我们找到一个带有内部条件的 [for 循环](/web/20221208143816/https://www.baeldung.com/java-for-loop)。如果“`i`”大于 100，则满足 for-loop 中的条件语句。但是，一个负的输入数呢？。是的，你是对的。我们遗漏了一个默认的 return 语句。因此，如果我们编译我们的代码，我们会再次得到`java: missing return statement`错误。

所以，为了修复它，我们只需要在方法的末尾放一个默认的 return 语句:

```java
public static String checkNumber(int number) {
    if (number == 0) {
        return "It's equals to zero";
    }
    for (int i = 0; i < number; i++) {
        if (i > 100) {
            return "It's a big number";
        }
    }
    return "It's a negative number";
}
```

### 2.2。Lambdas 中缺少返回

此外，当我们使用 lambdas 时，可能会出现此错误。对于函数来说，检测这个错误可能有点棘手。streams 中的`[map](/web/20221208143816/https://www.baeldung.com/java-8-streams-introduction#3-mapping)`方法是发生这种错误的常见地方。让我们检查一下我们的代码:

```java
public Map<String, Integer> createDictionary() {
    List<String> words = Arrays.asList("Hello", "World");
    Map<String, Integer> dictionary = new HashMap<>();
    words.stream().map(s -> {dictionary.put(s, 1);});
    return dictionary;
}
```

前面的代码看起来很好。有退货声明。我们的返回数据类型等于方法签名。但是，流中的`map`方法内部的代码呢？。`map`方法需要一个函数作为参数。在这种情况下，我们只将数据放入 map 方法内的字典中。因此，如果我们试图编译这段代码，我们会再次得到`java: missing return statement`错误。

接下来，为了解决这个错误，我们简单地将流中的`map`替换为`[forEach](/web/20221208143816/https://www.baeldung.com/foreach-java)`方法:

```java
words.forEach(s -> {dictionary.put(s, 1);});
```

或者，直接从流中返回地图:

```java
dictionary = words.stream().collect(Collectors.toMap(s -> s, s -> 1))
```

### 2.3。缺少方法签名

最后，最后一种情况是我们错过了向方法签名添加返回类型。因此，当我们试图编译我们的方法时，我们会得到一个错误。下面的代码示例向我们展示了这种行为:

```java
public pow(int number) {
    int pow = number * number;
    return pow;
}
```

我们忘记添加 int 作为返回类型。如果我们将它添加到我们的方法签名中，将会修复这个错误:

```java
public int pow(int number) {
    int pow = number * number;
    return pow;
}
```

## 3.结论

在本文中，我们讨论了一些缺失 return 语句的例子。它如何出现在我们的代码中，以及我们如何修复它。这有助于避免我们代码中将来的错误，并且可能需要几分钟的代码检查。

像往常一样，本文中使用的所有片段都可以在 GitHub 上获得[。](https://web.archive.org/web/20221208143816/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-exceptions-4)