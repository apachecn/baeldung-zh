# Groovy 中字符串的模式匹配

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/groovy-pattern-matching>

## 1.概观

在本文中，我们将研究 Groovy 语言在字符串模式匹配方面的特性。

我们将看到 Groovy 的电池内置方法如何为我们的基本模式匹配需求提供一个强大且符合人体工程学的语法。

## 2.模式运算符

Groovy 语言引入了所谓的模式操作符`~`。这个操作符可以被认为是 Java 的`java.util.regex.Pattern.compile(string)`方法的语法捷径。

作为`Spock`测试的一部分，让我们在实践中检验一下:

```java
def "pattern operator example"() {
    given: "a pattern"
    def p = ~'foo'

    expect:
    p instanceof Pattern

    and: "you can use slashy strings to avoid escaping of blackslash"
    def digitPattern = ~/\d*/
    digitPattern.matcher('4711').matches()
}
```

这也很方便，但是我们会看到这个操作符仅仅是其他一些更有用的操作符的基线。

## 3.匹配运算符

大多数时候，尤其是在编写测试时，我们并不真的对创建`Pattern`对象感兴趣，而是想检查`String`是否匹配某个正则表达式(或`Pattern`)。因此，Groovy 也包含匹配操作符`==~`。

它返回一个`boolean`并对指定的正则表达式执行严格匹配。基本上，这是调用`Pattern.matches(regex, string)`的语法捷径。

同样，作为`Spock`测试的一部分，我们将在实践中研究它:

```java
def "match operator example"() {
    expect:
    'foobar' ==~ /.*oba.*/

    and: "matching is strict"
    !('foobar' ==~ /foo/)
}
```

## 4.查找运算符

模式匹配上下文中的最后一个 Groovy 操作符是 find 操作符`~=`。在这种情况下，操作员将直接创建并返回一个`java.util.regex.Matcher`实例。

当然，我们可以通过访问已知的 Java API 方法来操作这个`Matcher`实例。但是除此之外，我们还可以使用多维数组访问匹配的组。

这还不是全部——`Matcher`实例将通过调用它的`find()`方法自动强制转换为`boolean`类型，如果它被用作谓词的话。引用官方 Groovy 文档，这意味着“=~操作符与简单使用 Perl 的=~操作符是一致的”。

在这里，我们可以看到操作人员的行动:

```java
def "find operator example"() {
    when: "using the find operator"
    def matcher = 'foo and bar, baz and buz' =~ /(\w+) and (\w+)/

    then: "will find groups"
    matcher.size() == 2

    and: "can access groups using array"
    matcher[0][0] == 'foo and bar'
    matcher[1][2] == 'buz'

    and: "you can use it as a predicate"
    'foobarbaz' =~ /bar/
}
```

## 5.结论

我们已经看到了 Groovy 语言如何以一种非常方便的方式让我们访问内置的关于正则表达式的 Java 特性。

官方 Groovy 文档也包含了一些关于这个主题的简明例子。如果您认为文档中的代码示例是作为文档构建的一部分来执行的，那就太棒了。

和往常一样，代码示例可以在 [GitHub](https://web.archive.org/web/20220630140629/https://github.com/eugenp/tutorials/tree/master/core-groovy-modules/core-groovy-2) 上找到。