# 如何在 Groovy 中删除字符串的前缀

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/groovy-remove-string-prefix>

## 1.介绍

在这个快速教程中，我们将学习如何在 [Groovy](/web/20221126233043/https://www.baeldung.com/groovy-language) 中移除字符串的前缀。

首先，我们来看看`String`类为此提供了什么。之后，我们将继续学习正则表达式，看看如何使用它们来删除前缀。

## 2.使用`String`方法

通常， [Groovy](/web/20221126233043/https://www.baeldung.com/groovy-language) 被认为是 Java 生态系统的动态语言。因此，我们仍然可以使用每个 Java `String`类方法和新的 Groovy 方法。但是，对于前缀的移除，仍然缺少像`removePrefix()`这样简单的方法。

从 Groovy 字符串中移除前缀包括两个步骤:首先确认，然后移除。这两个步骤都可以使用`StringGroovyMethods`类来执行，该类提供了许多字符串操作的实用方法。

### 2.1.`startsWith()`方法

方法测试一个字符串是否以一个特定的前缀开始。如果前缀存在，它返回`true`，否则返回`false`。

让我们从一个 [groovy 闭包](/web/20221126233043/https://www.baeldung.com/groovy-closures)开始:

```java
@Test 
public void whenCasePrefixIsRemoved_thenReturnTrue(){
    def trimPrefix = {
        it.startsWith('Groovy-') ? it.minus('Groovy-') : it 
    }
    def actual = trimPrefix("Groovy-Tutorials at Baeldung")
    def expected = "Tutorials at Baeldung"
    assertEquals(expected, actual)
} 
```

一旦确认存在，那么我们也可以使用`substring()`方法来删除它:

```java
trimPrefix.substring('Groovy-'.length()) 
```

### 2.2.`startsWithIgnoreCase()`方法

**`startsWith()`方法区分大小写**。因此，通过应用`toLowerCase()`或`toUpperCase()` 方法，需要人工努力来否定案例的影响。

顾名思义，`startsWithIgnoreCase()` 搜索前缀时不考虑大小写。如果前缀存在，则返回 true，否则返回 false。

让我们看看如何使用这种方法:

```java
@Test
public void whenPrefixIsRemovedWithIgnoreCase_thenReturnTrue() {

    String prefix = "groovy-"
    String trimPrefix = "Groovy-Tutorials at Baeldung"
    def actual
    if(trimPrefix.startsWithIgnoreCase(prefix)) {
        actual = trimPrefix.substring(prefix.length())
    }

    def expected = "Tutorials at Baeldung"

    assertEquals(expected, actual)
} 
```

### 2.3.`startsWithAny()`方法

当我们只需要检查一个前缀时，上述解决方案非常有用。在检查多个前缀时，Groovy 也提供了检查多个前缀的支持。

**`startsWithAny()`方法检查`CharSequence`是否以任何指定的前缀开始。**前缀确定后，我们可以根据需要应用逻辑:

```java
String trimPrefix = "Groovy-Tutorials at Baeldung"
if (trimPrefix.startsWithAny("Java", "Groovy", "Linux")) {
    // logic to remove prefix
} 
```

## 3.使用正则表达式

正则表达式是匹配或替换模式的有效方法。Groovy 有一个[模式操作符~](/web/20221126233043/https://www.baeldung.com/groovy-pattern-matching) ，它提供了一种创建`java.util.regex.Pattern` 实例的简单方法。

让我们定义一个简单的正则表达式来删除前缀:

```java
@Test
public void whenPrefixIsRemovedUsingRegex_thenReturnTrue() {

    def regex = ~"^groovy-"
    String trimPrefix = "groovy-Tutorials at Baeldung"
    String actual = trimPrefix - regex

    def expected = "Tutorials at Baeldung"
    assertEquals("Tutorials at Baeldung", actual)
} 
```

上述正则表达式的不区分大小写版本:

```java
def regex = ~"^([Gg])roovy-" 
```

脱字符运算符^将确保指定的子字符串存在于开始处。

### 3.1.`replaceFirst()`方法

使用正则表达式和本地字符串方法，我们可以执行非常强大的技巧。`replaceFirst()`方法就是这些方法中的一种。它替换与给定正则表达式匹配的第一个匹配项。

让我们使用`replaceFirst()`方法删除一个前缀:

```java
@Test
public void whenPrefixIsRemovedUsingReplaceFirst_thenReturnTrue() {

    def regex = ~"^groovy"
    String trimPrefix = "groovyTutorials at Baeldung's groovy page"
    String actual = trimPrefix.replaceFirst(regex, "")

    def expected = "Tutorials at Baeldung's groovy page"
    assertEquals(expected, actual)
} 
```

### 3.2.`replaceAll()`方法

与`replaceFirst()`一样， `replaceAll()` 也接受正则表达式和给定的替换。**它替换匹配给定标准**的每个子串。要删除前缀，我们也可以使用这个方法。

让我们使用`replaceAll()`仅替换字符串开头的子字符串:

```java
@Test
public void whenPrefixIsRemovedUsingReplaceAll_thenReturnTrue() {

    String trimPrefix = "groovyTutorials at Baeldung groovy"
    String actual = trimPrefix.replaceAll(/^groovy/, "")

    def expected = "Tutorials at Baeldung groovy"
    assertEquals(expected, actual)
} 
```

## 4.结论

在这个快速教程中，我们探索了几种从字符串中删除前缀的方法。为了确认前缀的存在，我们看到了如何对大写和小写字符串都这样做。

同时，我们已经看到了如何在许多提供的子字符串中检测前缀。我们还研究了可以用来删除子串的多种方法。最后，我们简要讨论了 regex 在这方面的作用。

和往常一样，所有代码示例都可以在 GitHub 上找到[。](https://web.archive.org/web/20221126233043/https://github.com/eugenp/tutorials/tree/master/core-groovy-modules/core-groovy-strings)