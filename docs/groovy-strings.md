# Groovy 中的字符串类型

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/groovy-strings>

## 1.概观

在本教程中，我们将仔细看看 [Groovy](/web/20221112235428/https://www.baeldung.com/groovy-language) 中的几种类型的字符串，包括单引号、双引号、三引号和斜杠字符串。

我们还将探索 Groovy 对特殊字符、多行、正则表达式、转义和变量插值的字符串支持。

## 2.增强功能`java.lang.String`

最好先说明，因为 Groovy 是基于 Java 的，所以它拥有 Java 的所有`String `功能，比如连接、字符串 API，以及由此带来的字符串常量池的固有好处。

让我们首先看看 Groovy 是如何扩展这些基础知识的。

### 2.1.串并置

字符串串联只是两个字符串的组合:

```java
def first = 'first'
def second = "second"        
def concatenation = first + second
assertEquals('firstsecond', concatenation)
```

Groovy 在此基础上构建了其他几个字符串类型，我们一会儿就会看到。请注意，我们可以互换连接每种类型。

### 2.2.字符串插值

现在，Java 通过`printf`提供了一些非常基本的模板，但是 Groovy 更深入，提供了 **`string interpolation, `用变量**模板化字符串的过程:

```java
def name = "Kacper"
def result = "Hello ${name}!"
assertEquals("Hello Kacper!", result.toString())
```

虽然 Groovy 支持所有字符串类型的串联，但它只为某些类型提供插值。

### 2.3.`GString`

但是在这个例子中隐藏了一个小问题——**为什么我们要调用`toString()`？**

实际上，`result `不属于`String`类型，即使它看起来像。

因为`String `类是`final`，Groovy 的支持插值的 string 类`GString`并没有子类化它。换句话说，为了让 Groovy 提供这种增强，**它有自己的字符串类，`GString`，不能从`String.`** 扩展

简而言之，如果我们做到了:

```java
assertEquals("Hello Kacper!", result)
```

这调用了`assertEquals(Object, Object), `,我们得到:

```java
java.lang.AssertionError: expected: java.lang.String<Hello Kacper!>
  but was: org.codehaus.groovy.runtime.GStringImpl<Hello Kacper!>
Expected :java.lang.String<Hello Kacper!> 
Actual   :org.codehaus.groovy.runtime.GStringImpl<Hello Kacper!>
```

## 3.单引号字符串

**Groovy 中最简单的字符串可能是单引号:**

```java
def example = 'Hello world'
```

在引擎盖下，这些只是普通的老式 Java `Strings`，当我们需要在字符串中包含引号时，它们会派上用场**。**

而不是:

```java
def hardToRead = "Kacper loves \"Lord of the Rings\""
```

我们可以很容易地将一个字符串与另一个字符串连接起来:

```java
def easyToRead = 'Kacper loves "Lord of the Rings"'
```

因为我们可以像这样交换引号类型，所以减少了转义引号的需要。

## 4.三重单引号字符串

在定义多行内容的上下文中，三个单引号字符串很有帮助。

例如，假设我们有一些`JSON`要表示为一个字符串:

```java
{
    "name": "John",
    "age": 20,
    "birthDate": null
}
```

我们不需要借助连接和显式换行符来表示这一点。

相反，让我们使用三个单引号字符串:

```java
def jsonContent = '''
{
    "name": "John",
    "age": 20,
    "birthDate": null
}
'''
```

**Groovy 将其存储为一个简单的 Java `String`** ，并为我们添加所需的连接和换行符。

然而，还有一个挑战需要克服。

**通常为了代码的可读性，我们缩进代码:**

```java
def triple = '''
    firstline
    secondline
'''
```

但是**三个单引号字符串保留空白**。这意味着上面的字符串实际上是:

```java
(newline)
    firstline(newline)
    secondline(newline)
```

不是:

| 1T2 2 | `firstline(newline)``secondline(newline)` |

就像我们预期的那样。

请继续关注，看看我们如何摆脱他们。

### 4.1.换行符

让我们确认一下**我们之前的字符串是以换行符**开始的:

```java
assertTrue(triple.startsWith("\n"))
```

有可能剥掉那个角色。**为了防止这种情况，我们需要放一个反斜杠`\`** 作为第一个和最后一个字符:

```java
def triple = '''\
    firstline
    secondline
'''
```

现在，我们至少有:

| 1T2 2 | `firstline(newline)``secondline(newline)` |

解决了一个问题，还有一个。

### 4.2.去掉代码缩进

接下来，让我们来处理缩进。我们希望保留我们的格式，但删除不必要的空白字符。

Groovy String API 来帮忙了！

**要删除字符串中每一行的前导空格，我们可以使用 Groovy 默认方法之一，`String#stripIndent()` :**

```java
def triple = '''\
    firstline
    secondline'''.stripIndent()
assertEquals("firstline\nsecondline", triple)
```

请注意，通过将刻度上移一行，**我们还删除了一个尾随换行符。**

### 4.3.相对缩进

我们应该记住`stripIndent `不叫`stripWhitespace.`

**`stripIndent` 确定字符串中缩短的非空白行的缩进量。**

因此，让我们为我们的`triple`变量改变一下缩进:

```java
class TripleSingleQuotedString {

    @Test
    void 'triple single quoted with multiline string with last line with only whitespaces'() {
        def triple = '''\
            firstline
                secondline\
        '''.stripIndent()

        // ... use triple
    }
}
```

印刷术`triple`将向我们展示:

```java
firstline
    secondline
```

**因为`firstline `是最小缩进的非空白行，所以它变成零缩进，而`secondline`仍然相对于它缩进。**

还要注意，这一次，我们用斜线删除了尾随空格，就像我们前面看到的那样。

### 4.4.用`stripMargin()`剥离

**为了更好的控制，我们可以使用|和`stripMargin` :** 来告诉 Groovy 从哪里开始这一行

```java
def triple = '''\
    |firstline
    |secondline'''.stripMargin()
```

它将显示:

```java
firstline
secondline
```

管道指明了字符串中该行真正开始的位置。

此外，我们可以通过自定义分隔符将一个`Character`或`CharSequence`作为参数传递给`stripMargin `。

太好了，我们去掉了所有不必要的空白，我们的字符串只包含我们想要的！

### 4.5.转义特殊字符

有了三重单引号字符串的所有优点，一个自然的结果是需要转义单引号和反斜杠，它们是我们字符串的一部分。

为了表示特殊字符，我们还需要用反斜杠对它们进行转义。最常见的特殊字符是换行符(`\n`)和制表符(`\t`)。

例如:

```java
def specialCharacters = '''hello \'John\'. This is backslash - \\ \nSecond line starts here'''
```

将导致:

```java
hello 'John'. This is backslash - \
Second line starts here
```

有几个我们需要记住，即:

*   `\t `–制表
*   `\n`–换行
*   `\b`–退格键
*   `\r`–回车
*   `\\`–反斜杠
*   `\f`–换页
*   `\'`–单引号

## 5.双引号字符串

虽然双引号字符串也只是 Java `Strings`，但它们的特殊功能是插值。当双引号字符串包含插值字符时，Groovy 会将 Java `String`换成`GString`。

### 5.1。 `GString`和懒评

**我们可以通过用`${}`包围表达式或者用`$`包围带点的表达式来插入双引号的字符串。**

**它的求值是懒惰的**，尽管——它不会被转换成`String`，直到它被传递给一个需要`String`的方法:

```java
def string = "example"
def stringWithExpression = "example${2}"
assertTrue(string instanceof String)
assertTrue(stringWithExpression instanceof GString)
assertTrue(stringWithExpression.toString() instanceof String)
```

### 5.2.引用变量的占位符

我们可能想对插值做的第一件事是给它发送一个变量引用:

```java
def name = "John"
def helloName = "Hello $name!"
assertEquals("Hello John!", helloName.toString())
```

### 5.2.带有表达式的占位符

但是，我们也可以给它表达式:

```java
def result = "result is ${2 * 2}"    
assertEquals("result is 4", result.toString())
```

我们可以将偶数语句放入占位符中，但这被认为是不好的做法。

### 5.3.带点运算符的占位符

我们甚至可以在字符串中遍历对象层次结构:

```java
def person = [name: 'John']
def myNameIs = "I'm $person.name, and you?"
assertEquals("I'm John, and you?", myNameIs.toString())
```

使用 getters，Groovy 通常可以推断出属性名。

**但是如果我们直接调用一个方法，我们就需要使用`${}`** 因为有括号:

```java
def name = 'John'
def result = "Uppercase name: ${name.toUpperCase()}".toString()
assertEquals("Uppercase name: JOHN", result)
```

### 5.4.`hashCode` 中的`GString`和`String`

与普通的`java.util.String, `相比，插值字符串当然是天赐之物，但是**它们在一个重要的方面有所不同。**

看，Java `Strings`是不可变的，所以在给定的字符串上调用`hashCode`总是返回相同的值。

但是， **`GString`哈希码可以改变**，因为`String `表示依赖于插值。

**实际上，即使对于相同的结果字符串，它们也不会有相同的散列码:**

```java
def string = "2+2 is 4"
def gstring = "2+2 is ${4}"
assertTrue(string.hashCode() != gstring.hashCode())
```

**因此，我们不应该使用`GString`作为`Map`中的键！**

## 6.三重双引号字符串

我们见过三重单引号字符串，也见过双引号字符串。

**让我们结合两者的力量，两全其美——多行字符串插值:**

```java
def name = "John"
def multiLine = """
    I'm $name.
    "This is quotation from 'War and Peace'"
"""
```

另外，请注意**我们不必转义单引号或双引号**！

## 7.斜线字符串

现在，让我们说我们正在用正则表达式做一些事情，因此我们在所有地方都避免了反斜杠:

```java
def pattern = "\\d{1,3}\\s\\w+\\s\\w+\\\\\\w+"
```

显然是一团糟。

为了帮助解决这个问题， **Groovy 通过斜杠字符串本机支持 regex】**

```java
def pattern = /\d{3}\s\w+\s\w+\\\w+/
assertTrue("3 Blind Mice\Men".matches(pattern))
```

斜线字符串**可以是插值字符串，也可以是多行字符串**:

```java
def name = 'John'
def example = /
    Dear ([A-Z]+),
    Love, $name
/
```

当然，我们必须避开正斜杠:

```java
def pattern = /.*foobar.*\/hello.*/ 
```

**我们不能用`Slashy String`** 来表示空字符串，因为编译器把`//`理解为注释:

```java
// if ('' == //) {
//     println("I can't compile")
// }
```

## 8.美元斜线字符串

斜线字符串是很棒的，尽管不得不避开正斜线是令人失望的。为了避免额外转义一个正斜杠，我们可以使用一个美元斜杠字符串。

假设我们有一个正则表达式模式:`[0-3]+/[0-3]+`。这是美元斜线字符串的一个很好的候选，因为在斜线字符串中，我们必须写:`[0-3]+//[0-3]+`。

美元斜线**字符串是多行`GString`字符串，以$/开头，以/$结尾。**为了转义美元或正斜杠，我们可以在它前面加上美元符号($)，但这不是必需的。

我们不需要在`GString`占位符中对$进行转义。

例如:

```java
def name = "John"

def dollarSlashy = $/
    Hello $name!,

    I can show you a $ sign or an escaped dollar sign: $ 
    Both slashes work: \ or /, but we can still escape it: $/

    We have to escape opening and closing delimiters:
    - $$/  
    - $/$
 /$ 
```

将输出:

```java
Hello John!,

I can show you a $ sign or an escaped dollar sign: $ 
Both slashes work: \ or /, but we can still escape it: /

We have to escape opening and closing delimiter:
- $/  
- /$
```

## 9.性格；角色；字母

熟悉 Java 的人已经想知道 Groovy 对字符做了什么，因为它对字符串使用单引号。

实际上， **Groovy** **并没有明确的字符字面意义。**

有三种方法**可以让一个`Groovy`字符串成为一个真实的字符:**

 ***   声明变量时显式使用“char”关键字
*   使用“as”运算符
*   通过转换为“char”

让我们来看看它们:

```java
char a = 'A'
char b = 'B' as char
char c = (char) 'C'
assertTrue(a instanceof Character)
assertTrue(b instanceof Character)
assertTrue(c instanceof Character)
```

当我们想保持字符为变量时，第一种方法非常方便。当我们想把一个字符作为参数传递给一个函数时，另外两种方法更有趣。

## 10。总结

显然，这已经很多了，所以让我们快速总结一些要点:

*   用单引号(')创建的字符串不支持插值
*   斜线和三重双引号字符串可以是多行的
*   由于代码缩进，多行字符串包含空白字符
*   反斜杠(\)用于转义每种类型中的特殊字符，美元斜杠字符串除外，在这种情况下我们必须使用美元($)进行转义

## 11.结论

在本文中，我们讨论了在 Groovy 中创建字符串的许多方法及其对多行、插值和正则表达式的支持。

所有这些片段都可以在 Github 上找到。

关于 Groovy 语言本身特性的更多信息，可以从我们的[Groovy 语言介绍](/web/20221112235428/https://www.baeldung.com/groovy-language)开始。**