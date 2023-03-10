# “找不到符号”编译错误

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-cannot-find-symbol-error>

## 1.概观

在本教程中，我们将回顾什么是编译错误。然后我们将具体解释“`cannot find symbol`”错误以及它是如何引起的。

## 2.编译时错误

在编译过程中，编译器会分析和验证代码中的许多东西，比如引用类型、类型转换和方法声明等等。编译过程的这一部分很重要，因为在这个阶段我们会得到一个编译错误。

基本上，有三种类型的编译时错误:

*   我们可能会有语法错误。任何程序员都会犯的最常见的错误之一就是忘记在语句的末尾加上分号。其他一些错误包括忘记导入、括号不匹配或省略 return 语句。
*   **接下来，还有** **的打字错误。这是在我们的代码中验证类型安全的过程。通过这项检查，我们可以确保我们拥有一致的表达式类型。例如，如果我们定义了一个类型为`int`的变量，我们不应该给它赋值`double`或`String`。**
*   最后，编译器可能会崩溃。这种情况非常罕见，但也有可能发生。在这种情况下，知道我们的代码可能不是问题，而是一个外部问题是很好的。

## 3.`“cannot find symbol”`错误

“`cannot find symbol`”错误主要出现在我们试图使用程序中没有定义或声明的变量时。

当我们的代码编译时，编译器需要验证我们拥有的所有标识符。**错误****`cannot find symbol`意味着我们** **引用了一些编译器不知道的**。

### 3.1。什么会导致`“cannot find symbol”`错误？

真正的原因只有一个。编译器找不到我们试图引用的变量的定义。

但是出现这种情况的原因有很多。为了帮助我们理解为什么，让我们提醒自己我们的 Java 源代码由什么组成:

*   关键词:`true, false, class, while`
*   文字:数字和文本
*   运算符和其他非字母数字标记:-、/、+、=、{
*   标识符:`main`、`Reader`、`i`、`toString`等。
*   注释和空白

## 4.拼错

最常见的问题都与拼写有关。如果我们回忆一下，所有的 Java 标识符都是区分大小写的，我们会发现下面这些都是错误引用`StringBuilder` 类的不同方式:

*   `StringBiulder`
*   `stringBuilder`
*   `String_Builder`

## 5.实例范围

当使用在类范围之外声明的内容时，也会导致此错误。

例如，假设我们有一个调用`generateId `方法的`Article`类:

```java
public class Article {
    private int length;
    private long id;

    public Article(int length) {
        this.length = length;
        this.id = generateId();
    }
}
```

但是我们在一个单独的类中声明了`generateId` 方法:

```java
public class IdGenerator {
    public long generateId() {
        Random random = new Random();
        return random.nextInt();
    }
}
```

通过这种设置，编译器将在 `Article` 代码片段的第 7 行为`generateId`给出一个“`cannot find symbol`错误。这是因为第 7 行的语法暗示了在`Article`中声明了`generateId`方法。

像在所有成熟的语言中一样，解决这个问题的方法不止一种，但一种方法是在`Article` 类中构造`IdGenerator`，然后调用该方法:

```java
public class Article {
    private int length;
    private long id;

    public Article(int length) {
        this.length = length;
        this.id = new IdGenerator().generateId();
    }
}
```

## 6.未定义的变量

有时我们会忘记声明变量。正如我们从下面的代码片段中看到的，我们试图操作我们还没有声明的变量，在本例中是`text`:

```java
public class Article {
    private int length;

    // ...

    public void setText(String newText) {
        this.text = newText; // text variable was never defined
    }
}
```

我们通过声明类型为`String`的变量`text`来解决这个问题:

```java
public class Article {
    private int length;
    private String text;
    // ...

    public void setText(String newText) {
        this.text = newText;
    }
}
```

## 7.变量作用域

当一个变量声明超出了我们试图使用它的范围时，它会在编译过程中导致一个错误。这通常发生在我们处理循环的时候。

循环内部的变量在循环外部是不可访问的:

```java
public boolean findLetterB(String text) {
    for (int i=0; i < text.length(); i++) {
        Character character = text.charAt(i);
        if (String.valueOf(character).equals("b")) {
            return true;
        }
        return false;
    }

    if (character == "a") {  // <-- error!
        ...
    }
}
```

如果我们需要更多地检查字符，那么`if`语句应该放在`for loop`中:

```java
public boolean findLetterB(String text) {
    for (int i = 0; i < text.length(); i++) {
        Character character = text.charAt(i);
        if (String.valueOf(character).equals("b")) {
            return true;
        } else if (String.valueOf(character).equals("a")) {
            ...
        }
        return false;
    }
}
```

## 8.方法或字段的使用无效

如果我们将字段用作方法，也会出现“`cannot find symbol`”错误，反之亦然:

```java
public class Article {
    private int length;
    private long id;
    private List<String> texts;

    public Article(int length) {
        this.length = length;
    }
    // getters and setters
}
```

如果我们试图引用文章的`texts`字段，就好像它是一个方法一样:

```java
Article article = new Article(300);
List<String> texts = article.texts();
```

然后我们会看到错误。

这是因为编译器正在寻找一个名为`texts`的方法，但是没有。

实际上，我们可以用一种`getter`方法来代替:

```java
Article article = new Article(300);
List<String> texts = article.getTexts();
```

错误地操作数组而不是数组元素也是一个问题:

```java
for (String text : texts) {
    String firstLetter = texts.charAt(0); // it should be text.charAt(0)
}
```

忘记关键字`new` 也是如此:

```java
String s = String(); // should be 'new String()'
```

## 9.包和类导入

**另一个问题是忘记导入类或包，**，就像使用一个`List`对象而不导入`java.util.List`:

```java
// missing import statement: 
// import java.util.List

public class Article {
    private int length;
    private long id;
    private List<String> texts;  <-- error!
    public Article(int length) {
        this.length = length;
    }
}
```

这段代码无法编译，因为程序不知道`List`是什么。

## 10.错误的进口

由于 IDE 完成或自动更正，导入错误的类型也是一个常见问题。

想象一个场景，我们想在 Java 中使用日期。很多时候，我们可能会导入一个错误的`Date`类，它不提供我们可能需要的与其他日期类相同的方法和功能:

```java
Date date = new Date();
int year, month, day;
```

为了获取`java.util.Date`的年、月或日，我们还需要导入`Calendar`类并从中提取信息。

简单地从 `java.util.Date`调用`getDate()`是行不通的:

```java
...
date.getDay();
date.getMonth();
date.getYear();
```

相反，我们使用`Calendar`对象:

```java
...
Calendar cal = Calendar.getInstance(TimeZone.getTimeZone("Europe/Paris"));
cal.setTime(date);
year = cal.get(Calendar.YEAR);
month = cal.get(Calendar.MONTH);
day = cal.get(Calendar.DAY_OF_MONTH);
```

然而，如果我们已经导入了`LocalDate`类，我们将不需要额外的代码来提供我们需要的信息:

```java
...
LocalDate localDate=date.toInstant().atZone(ZoneId.systemDefault()).toLocalDate();
year = localDate.getYear();
month = localDate.getMonthValue();
day = localDate.getDayOfMonth();
```

## 11.结论

编译器遵循一套特定于语言的固定规则。如果代码不遵守这些规则，编译器就不能执行转换过程，从而导致编译错误。当我们面对“`cannot find symbol`”编译错误时，关键是要找出原因。

从错误消息中，我们可以找到发生错误的代码行，以及哪个元素是错误的。了解导致此错误的最常见问题将使解决它变得快速而容易。