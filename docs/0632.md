# Datafaker 简介

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-datafaker>

## 1.概观

在本教程中，我们将学习如何处理为不同目的生成模拟数据的问题。我们将学习如何使用[data maker](https://web.archive.org/web/20221024112317/http://www.datafaker.net/)并回顾几个例子。

## 2.历史

**Datafaker 是 [Javafaker](https://web.archive.org/web/20221024112317/https://github.com/DiUS/java-faker) 的现代分支。**它被转移到 Java 8 并经历了改进，增加了库的[性能](https://web.archive.org/web/20221024112317/http://www.datafaker.net/documentation/performance/)。然而，当前的 API 基本保持不变。因此，以前使用的 Javafaker 在迁移到 Datafaker 时不会有任何问题。**Java Faker 文章中提供的所有例子都适用于 Datafaker 的`1.6.0 `版本。**

当前的 Datafaker API 与 Javafaker 兼容。因此，本文将只关注不同之处和改进之处。

首先，让我们将`[datafaker](https://web.archive.org/web/20221024112317/https://search.maven.org/search?q=g:net.datafaker%20AND%20a:datafaker%20AND%20v:1.6.0)` Maven 依赖项添加到项目中:

```
<dependency>
    <groupId>net.datafaker</groupId>
    <artifactId>datafaker</artifactId>
    <version>1.6.0</version>
</dependency>
```

## 3.提供者

Datafaker 最重要的部分之一是[提供者](https://web.archive.org/web/20221024112317/http://www.datafaker.net/documentation/providers/)。这是一组特殊的类，使数据生成更加方便。需要注意的是，这些类是由带有正确数据的 [`yml `文件](https://web.archive.org/web/20221024112317/https://github.com/datafaker-net/datafaker/tree/main/src/main/resources)备份的。`Faker`方法和表达式直接或间接地使用这些文件来生成数据。在接下来的部分中，我们将更加熟悉这些方法和指令的工作。

## 4.数据生成的其他模式

Datafaker 和 Javafaker 都支持基于所提供的模式生成值。 **Datafaker 通过`templatify, exemplify, options, date, csv, and json` 指令`.`** 引入了附加功能

### 4.1.模板化

指令有几个参数。第一个是底座 S `tring`。第二个是给定字符串中将被替换的字符。其余的是随机选择的替换选项:

```
public class Templatify {
    private static Faker faker = new Faker();

    public static void main(String[] args) {
        System.out.println("Expression: " + getExpression());
        System.out.println("Expression with a placeholder: " + getExpressionWithPlaceholder());
    }

    static String getExpression() {
        return faker.expression("#{templatify 'test','t','j','r'}");
    }

    static String getExpressionWithPlaceholder() {
        return faker.expression("#{templatify '#ight', '#', 'f', 'l', 'm', 'n'}");
    }
}
```

虽然我们可以使用不带占位符的基本字符串，但它可能会产生不良的结果，因为它将替换给定字符串中的所有匹配项。我们可以引入一个占位符，一个只出现在基本字符串特定位置的字符。在上面的例子中，结果是:

```
Expression: resj
Expression with a placeholder: night
```

如果有多个地方可以放置随机字符，那么每次都会被随机化。使用字符串进行替换是可能的，但是文档中没有明确提到这一点。因此，最好谨慎使用。

### 4.2.举例说明

本指令根据提供的示例生成一个随机值。它会用相应的值替换小写或大写字符。数字也是如此。**特殊字符不变，这有助于创建格式化字符串:**

```
public class Examplify {
    private static Faker faker = new Faker();

    public static void main(String[] args) {
        System.out.println("Expression: " + getExpression());
        System.out.println("Number expression: " + getNumberExpression());
    }

    static String getExpression() {
        return faker.expression("#{examplify 'Cat in the Hat'}");
    }

    static String getNumberExpression() {
        return faker.expression("#{examplify '123-123-123'}");
    }
}
```

输出的示例:

```
Expression: Lvo lw ero Qkd
Number expression: 707-657-434
```

### 4.3.重新流放

这是创建格式化 S `tring`值的一种更灵活的方式。**我们可以使用`regexify`指令作为表达式，或者直接在`Faker `对象上调用`regexify`方法:**

```
public class Regexify {
    private static Faker faker = new Faker();

    public static void main(String[] args) {
        System.out.println("Expression: " + getExpression());
        System.out.println("Regexify with a method: " + getMethodExpression());
    }

    static String getExpression() {
        return faker.expression("#{regexify '(hello|bye|hey)'}");
    }

    static String getMethodExpression() {
        return faker.regexify("[A-D]{4,10}");
    }
} 
```

可能的输出:

```
Expression: bye
Regexify with a method: DCCC 
```

### 4.4.选择

**`options.option`指令允许从提供的列表中随机选择一个选项。**该功能可以通过`regexify`实现，但由于这是一种常见情况，所以单独的指令更有意义:

```
public class Option {
    private static Faker faker = new Faker();

    public static void main(String[] args) {
        System.out.println("First expression: " + getFirstExpression());
        System.out.println("Second expression: " + getSecondExpression());
        System.out.println("Third expression: " + getThirdExpression());
    }

    static String getFirstExpression() {
        return faker.expression("#{options.option 'Hi','Hello','Hey'}");
    }

    static String getSecondExpression() {
        return faker.expression("#{options.option '1','2','3','4','*'}");
    }

    static String getThirdExpression() {
        return faker.expression("#{regexify '(Hi|Hello|Hey)'}");
    }
}
```

上面代码的输出:

```
First expression: Hey
Second expression: 4
Third expression: Hello
```

如果选项的数量太多，为随机值创建一个自定义提供程序是有意义的。

### 4.5.战斗支援车

该指令根据其名称创建 CSV 格式的数据。然而，使用这个指令可能会引起混淆。因为，在幕后，两个签名完全不同的重载方法处理这个指令:

```
public class Csv {
    private static Faker faker = new Faker();

    public static void main(String[] args) {
        System.out.println("First expression:\n" + getFirstExpression());
        System.out.println("Second expression:\n" + getSecondExpression());
    }

    static String getFirstExpression() {
        String firstExpressionString
          = "#{csv '4','name_column','#{Name.first_name}','last_name_column','#{Name.last_name}'}";
        return faker.expression(firstExpressionString);
    }

    static String getSecondExpression() {
        String secondExpressionString
          = "#{csv ',','\"','true','4','name_column','#{Name.first_name}','last_name_column','#{Name.last_name}'}";
        return faker.expression(secondExpressionString);
    }
}
```

上面的指令使用了表达式`#{Name.first_name} `和`#{Name.last_name}`。下一节将解释这些表达式的用法。

表达式中`csv`指令之后的值被映射到上述方法的参数。这些方法的文档提供了更多信息。然而，有时解析这些指令可能会出现问题，在这种情况下，最好直接使用这些方法。上面的代码将产生以下输出:

```
First expression:
"name_column","last_name_column"
"Riley","Spinka"
"Lindsay","O'Conner"
"Sid","Rogahn"
"Prince","Wiegand"

Second expression:
"name_column","last_name_column"
"Jen","Schinner"
"Valeria","Walter"
"Mikki","Effertz"
"Deon","Bergnaum"
```

这是以编程方式生成模拟数据以便在应用程序之外使用的一种很好的方式。

### 4.6.JSON

另一种流行且常用的格式是 JSON。Datafaker 允许使用表达式生成 JSON 格式的数据:

```
public class Json {
    private static final Faker faker = new Faker();

    public static void main(String[] args) {
        System.out.println(getExpression());
    }

    static String getExpression() {
        return faker.expression(
          "#{json 'person'," + "'#{json ''first_name'',''#{Name.first_name}'',''last_name'',''#{Name.last_name}''}'," +
          "'address'," + "'#{json ''country'',''#{Address.country}'',''city'',''#{Address.city}''}'}");
    }
} 
```

上面的代码产生以下输出:

```
{"person": {"first_name": "Dorian", "last_name": "Simonis"}, "address": {"country": "Cameroon", "city": "South Ernestine"}}
```

### 4.7.方法调用

**事实上，所有的表达式都是方法调用，方法名和参数作为`String.`** 传递，因此，上面所有的指令都镜像了同名的方法。然而，有时使用纯文本创建模拟数据更方便:

```
public class MethodInvocation {
    private static Faker faker = new Faker();

    public static void main(String[] args) {
        System.out.println("Name from a method: " + getNameFromMethod());
        System.out.println("Name from an expression: " + getNameFromExpression());
    }

    static String getNameFromMethod() {
        return faker.name().firstName();
    }

    static String getNameFromExpression() {
        return faker.expression("#{Name.first_Name}");
    }
}
```

现在很明显，带有`csv `和`json `指令的表达式在内部使用了方法调用。**这样，我们可以在`Faker`对象**上调用任何数据生成方法。尽管方法名不区分大小写，并且允许格式上的变化，但是最好参考所用版本的文档来验证它。

**此外，可以通过表达式向方法传递参数。**我们在`regexify` 和`templatify` 指令的格式中看到了这一点。尽管在某些情况下可能有点麻烦和容易出错，但有时这是与`Faker:`互动的最方便的方式

```
public class MethodInvocationWithParams {
    public static int MIN = 1;
    public static int MAX = 10;
    public static String UNIT = "SECONDS";
    private static Faker faker = new Faker();

    public static void main(String[] args) {
        System.out.println("Duration from the method :" + getDurationFromMethod());
        System.out.println("Duration from the expression: " + getDurationFromExpression());
    }
    static Duration getDurationFromMethod() {
        return faker.date().duration(MIN, MAX, UNIT);
    }

    static String getDurationFromExpression() {
        return faker.expression("#{date.duration '1', '10', 'SECONDS'}");
    }
}
```

**表达式的缺点之一是它们返回一个`String`对象。**结果，这减少了我们可以对返回的对象进行的操作数量。上面的代码产生以下输出:

```
Duration from the method: PT6S
Duration from the expression: PT4S
```

## 5.收集

集合允许创建带有模拟数据的列表。在这种情况下，元素可以是不同的类型。集合由最特定的类型参数化:集合中所有类的父类。让我们来做一点工作，列出《星球大战》和《星际迷航》中的角色:

```
public class Collection {
    public static int MIN = 1;
    public static int MAX = 100;
    private static Faker faker = new Faker();

    public static void main(String[] args) {
        System.out.println(getFictionalCharacters());
    }

    static List<String> getFictionalCharacters() {
        return faker.collection(
          () -> faker.starWars().character(),
          () -> faker.starTrek().character())
            .len(MIN, MAX)
            .generate();
    }
} 
```

结果，我们得到了下面的列表:

`[Luke Skywalker, Wesley Crusher, Jean-Luc Picard, Greedo, Hikaru Sulu, William T. Riker]`

**因为我们集合中的两个供应商都返回字符串类型值，所以结果列表将由`String.`** 参数化。让我们检查混合不同类型数据的情况:

```
public class MixedCollection {
    public static int MIN = 1;
    public static int MAX = 20;
    private static Faker faker = new Faker();

    public static void main(String[] args) {
        System.out.println(getMixedCollection());
    }

    static List<? extends Serializable> getMixedCollection() {
        return faker.collection(
        () -> faker.date().birthday(),
        () -> faker.name().fullName())
          .len(MIN, MAX)
          .generate();
    }
}
```

**在这种情况下，`String`和`Timestamp `最具体的类是`Serializable.`** ，输出如下:

```
[1964-11-09 15:16:43.0, Devora Stamm DVM, 1980-01-11 15:18:00.0, 1989-04-28 05:13:54.0,
  2004-09-06 17:11:49.0, Irving Turcotte, Sherita Durgan I, 2004-03-08 00:45:57.0, 1979-08-25 22:48:50.0,
  Manda Hane, Latanya Hegmann, 1991-05-29 12:07:23.0, 1989-06-26 12:40:44.0, Kevin Quigley] 
```

## 6.结论

Datafaker 是 Javafaker 的一个新的改进版本。**本文介绍了 data maker `1.6.0,`中引入的新功能，它提供了生成数据的新方法。**然而，关于这个库还有更多要了解的，最好参考[官方文档](https://web.archive.org/web/20221024112317/http://www.datafaker.net/documentation/getting-started/)和 [GitHub 库](https://web.archive.org/web/20221024112317/https://github.com/datafaker-net/datafaker/)来获得更多关于 Datafaker 的功能和特性的信息。

和往常一样，文章中的代码可以在 GitHub 上找到[。](https://web.archive.org/web/20221024112317/https://github.com/eugenp/tutorials/tree/master/testing-modules/mocks-2)