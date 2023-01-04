# Vavr 简介

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/vavr>

## 1。概述

在这篇文章中，我们将探索到底什么是 Vavr，为什么我们需要它，以及如何在我们的项目中使用它。

Vavr 是 Java 8+的**函数库，提供不可变的数据类型和函数控制结构。**

### 1.1。Maven 依赖关系

为了使用 Vavr，您需要添加依赖项:

```java
<dependency>
    <groupId>io.vavr</groupId>
    <artifactId>vavr</artifactId>
    <version>0.9.0</version>
</dependency>
```

建议始终使用最新版本。关注[这个链接](https://web.archive.org/web/20221017183302/https://search.maven.org/classic/#artifactdetails%7Cio.vavr%7Cvavr%7C0.9.0%7Cjar)就可以获得。

## 2。选项

`The main goal of Option is to eliminate null checks in our code by leveraging the Java type system.`

`Option`是 Vavr 中的一个对象容器，其最终目标类似于 Java 8 中的[可选的](/web/20221017183302/https://www.baeldung.com/java-optional)。Vavr 的`Option`实现了 `Serializable, Iterable,` 并拥有更丰富的 API `.`

因为 Java 中的任何对象引用都可以有一个`null`值，所以我们通常必须在使用它之前用`if`语句检查是否为空。这些检查使代码健壮而稳定:

```java
@Test
public void givenValue_whenNullCheckNeeded_thenCorrect() {
    Object possibleNullObj = null;
    if (possibleNullObj == null) {
        possibleNullObj = "someDefaultValue";
    }
    assertNotNull(possibleNullObj);
}
```

如果没有检查，应用程序可能会因为一个简单的`NPE:`而崩溃

```java
@Test(expected = NullPointerException.class)
public void givenValue_whenNullCheckNeeded_thenCorrect2() {
    Object possibleNullObj = null;
    assertEquals("somevalue", possibleNullObj.toString());
}
```

然而，这些检查使得代码**冗长，可读性**差，尤其是当`if`语句被多次嵌套时。

`Option`通过完全删除`nulls`并用每个可能场景的有效对象引用替换它们来解决这个问题。

使用`Option`时，`null`值将计算为`None`的实例，而非空值将计算为`Some`的实例:

```java
@Test
public void givenValue_whenCreatesOption_thenCorrect() {
    Option<Object> noneOption = Option.of(null);
    Option<Object> someOption = Option.of("val");

    assertEquals("None", noneOption.toString());
    assertEquals("Some(val)", someOption.toString());
}
```

因此，不要直接使用对象值，建议将它们包装在一个`Option`实例中，如上所示。

请注意，我们在调用`toString` 之前不需要做检查，也不需要像以前一样处理`NullPointerException`。期权的`toString`在每次调用中返回给我们有意义的值。

在这一部分的第二个片段中，我们需要一个`null`检查，在尝试使用变量之前，我们将为变量分配一个默认值。`Option`可以在一行中处理这个，即使有一个 null:

```java
@Test
public void givenNull_whenCreatesOption_thenCorrect() {
    String name = null;
    Option<String> nameOption = Option.of(name);

    assertEquals("baeldung", nameOption.getOrElse("baeldung"));
}
```

或非空值:

```java
@Test
public void givenNonNull_whenCreatesOption_thenCorrect() {
    String name = "baeldung";
    Option<String> nameOption = Option.of(name);

    assertEquals("baeldung", nameOption.getOrElse("notbaeldung"));
}
```

请注意，在没有`null`检查的情况下，我们如何在一行中获得一个值或返回一个默认值。

## 3。元组

Java 中没有元组数据结构的直接等价物。元组是[函数式编程](/web/20221017183302/https://www.baeldung.com/cs/functional-programming)语言中的常见概念。**元组是不可变的，可以以类型安全的方式保存多个不同类型的对象。**

Vavr 给 Java 8 带来了元组。元组的类型是`Tuple1, Tuple2`到`Tuple8`，这取决于它们要接受的元素的数量。

目前有八个元素的上限。我们访问像`tuple` `._n`这样的元组元素，其中`n`类似于数组中的索引概念:

```java
public void whenCreatesTuple_thenCorrect1() {
    Tuple2<String, Integer> java8 = Tuple.of("Java", 8);
    String element1 = java8._1;
    int element2 = java8._2();

    assertEquals("Java", element1);
    assertEquals(8, element2);
}
```

注意，第一个元素是用`n==1`检索的。所以元组不像数组一样使用零基。将要存储在元组中的元素的类型必须在其类型声明中声明，如下图所示:

```java
@Test
public void whenCreatesTuple_thenCorrect2() {
    Tuple3<String, Integer, Double> java8 = Tuple.of("Java", 8, 1.8);
    String element1 = java8._1;
    int element2 = java8._2();
    double element3 = java8._3();

    assertEquals("Java", element1);
    assertEquals(8, element2);
    assertEquals(1.8, element3, 0.1);
}
```

元组的位置是存储一组固定的任何类型的对象，这些对象最好作为一个单元来处理，并且可以传递。一个更明显的**用例是从 Java 中的函数或方法**返回多个对象。

## 4。试试

在 Vavr 中， **`Try`是可能导致异常的计算** **的容器。**

因为`Option`包装了一个可空的对象，所以我们不必显式地用`if`检查来处理`nulls`，所以`Try`包装了一个计算，所以我们不必显式地用`try-catch`块来处理异常。

以下面的代码为例:

```java
@Test(expected = ArithmeticException.class)
public void givenBadCode_whenThrowsException_thenCorrect() {
    int i = 1 / 0;
}
```

如果没有`try-catch`块，应用程序将会崩溃。为了避免这种情况，您需要将语句包装在一个`try-catch`块中。使用 Vavr，我们可以将相同的代码包装在一个`Try`实例中，并得到一个结果:

```java
@Test
public void givenBadCode_whenTryHandles_thenCorrect() {
    Try<Integer> result = Try.of(() -> 1 / 0);

    assertTrue(result.isFailure());
}
```

计算是否成功可以在代码中的任意点进行检查。

在上面的代码片段中，我们选择了简单地检查成功或失败。我们也可以选择返回一个默认值:

```java
@Test
public void givenBadCode_whenTryHandles_thenCorrect2() {
    Try<Integer> computation = Try.of(() -> 1 / 0);
    int errorSentinel = result.getOrElse(-1);

    assertEquals(-1, errorSentinel);
}
```

或者甚至显式抛出我们选择的异常:

```java
@Test(expected = ArithmeticException.class)
public void givenBadCode_whenTryHandles_thenCorrect3() {
    Try<Integer> result = Try.of(() -> 1 / 0);
    result.getOrElseThrow(ArithmeticException::new);
}
```

在上述所有情况下，我们都可以控制计算后发生的事情，这要感谢 Vavr 的`Try`。

## 5。功能接口

随着 Java 8 的到来，[功能接口](/web/20221017183302/https://www.baeldung.com/java-8-functional-interfaces)被内置并且更容易使用，尤其是当与 lambdas 结合使用时。

然而，Java 8 只提供了两个基本功能。其中一个只接受一个参数并产生一个结果:

```java
@Test
public void givenJava8Function_whenWorks_thenCorrect() {
    Function<Integer, Integer> square = (num) -> num * num;
    int result = square.apply(2);

    assertEquals(4, result);
}
```

第二个函数只接受两个参数并产生一个结果:

```java
@Test
public void givenJava8BiFunction_whenWorks_thenCorrect() {
    BiFunction<Integer, Integer, Integer> sum = 
      (num1, num2) -> num1 + num2;
    int result = sum.apply(5, 7);

    assertEquals(12, result);
}
```

另一方面，Vavr 进一步扩展了 Java 中函数接口的概念，它支持最多 8 个参数，并为 API 添加了记忆、组合和 currying 方法。

就像元组一样，这些函数接口是根据它们接受的参数个数来命名的:`Function0`、`Function1`、`Function2`等。对于 Vavr，我们应该这样编写上面的两个函数:

```java
@Test
public void givenVavrFunction_whenWorks_thenCorrect() {
    Function1<Integer, Integer> square = (num) -> num * num;
    int result = square.apply(2);

    assertEquals(4, result);
}
```

还有这个:

```java
@Test
public void givenVavrBiFunction_whenWorks_thenCorrect() {
    Function2<Integer, Integer, Integer> sum = 
      (num1, num2) -> num1 + num2;
    int result = sum.apply(5, 7);

    assertEquals(12, result);
}
```

当没有参数但我们仍然需要一个输出时，在 Java 8 中我们需要使用一个*供应商*类型，在 Vavr 中`Function0`有帮助:

```java
@Test
public void whenCreatesFunction_thenCorrect0() {
    Function0<String> getClazzName = () -> this.getClass().getName();
    String clazzName = getClazzName.apply();

    assertEquals("com.baeldung.vavr.VavrTest", clazzName);
}
```

五参数函数怎么样，只是使用`Function5`的问题:

```java
@Test
public void whenCreatesFunction_thenCorrect5() {
    Function5<String, String, String, String, String, String> concat = 
      (a, b, c, d, e) -> a + b + c + d + e;
    String finalString = concat.apply(
      "Hello ", "world", "! ", "Learn ", "Vavr");

    assertEquals("Hello world! Learn Vavr", finalString);
}
```

我们还可以结合任何函数的静态工厂方法`FunctionN.of`,从方法引用创建一个 Vavr 函数。比如我们有下面的`sum`方法:

```java
public int sum(int a, int b) {
    return a + b;
}
```

我们可以这样创建一个函数:

```java
@Test
public void whenCreatesFunctionFromMethodRef_thenCorrect() {
    Function2<Integer, Integer, Integer> sum = Function2.of(this::sum);
    int summed = sum.apply(5, 6);

    assertEquals(11, summed);
}
```

## 6。收藏

Vavr 团队在设计新的集合 API 方面投入了大量精力，以满足函数式编程的要求，即持久性、不变性。

**Java 集合是可变的，这使得它们成为程序失败的一个重要来源**，尤其是在并发的情况下。`Collection`接口提供了这样的方法:

```java
interface Collection<E> {
    void clear();
}
```

该方法删除集合中的所有元素(产生副作用)并且不返回任何内容。像`ConcurrentHashMap`这样的类是为了处理已经产生的问题而创建的。

这样的班级不仅没有增加边际收益，还降低了班级的表现，而班级的漏洞是它试图填补的。

有了不变性，我们可以免费获得线程安全:不需要编写新的类来处理一个从一开始就不应该存在的问题。

在 Java 中为集合添加不变性的其他现有策略仍然会产生更多的问题，即异常:

```java
@Test(expected = UnsupportedOperationException.class)
public void whenImmutableCollectionThrows_thenCorrect() {
    java.util.List<String> wordList = Arrays.asList("abracadabra");
    java.util.List<String> list = Collections.unmodifiableList(wordList);
    list.add("boom");
}
```

以上问题在 Vavr 集合中都是不存在的。

要在 Vavr 中创建列表:

```java
@Test
public void whenCreatesVavrList_thenCorrect() {
    List<Integer> intList = List.of(1, 2, 3);

    assertEquals(3, intList.length());
    assertEquals(new Integer(1), intList.get(0));
    assertEquals(new Integer(2), intList.get(1));
    assertEquals(new Integer(3), intList.get(2));
}
```

API 也可用于就地对列表执行计算:

```java
@Test
public void whenSumsVavrList_thenCorrect() {
    int sum = List.of(1, 2, 3).sum().intValue();

    assertEquals(6, sum);
}
```

Vavr 集合提供了 Java 集合框架中大多数常见的类，实际上，所有的特性都实现了。

好处是**不变性**、**移除无效返回类型**和**产生副作用的 API**，一组更丰富的**函数来操作底层元素**，与 Java 的集合操作相比，代码非常短、健壮和紧凑。

全面介绍 Vavr 集合超出了本文的范围。

## 7 .**。验证**

Vavr 将`Applicative Functor`的概念从函数式编程世界带到了 Java。用最简单的话来说， **an `Applicative Functor`使我们能够执行一系列动作，同时累积结果**。

类别`vavr.control.Validation`促进了误差的累积。请记住，通常情况下，一遇到错误，程序就会终止。

然而，`Validation`继续处理和累积程序的错误，以批量处理它们。

假设我们通过`name`和`age`注册用户，我们希望首先获取所有输入，并决定是创建一个`Person`实例还是返回一个错误列表。下面是我们的`Person`班:

```java
public class Person {
    private String name;
    private int age;

    // standard constructors, setters and getters, toString
}
```

接下来，我们创建一个名为`PersonValidator`的类。每个字段将通过一种方法进行验证，另一种方法可用于将所有结果合并到一个`Validation`实例中:

```java
class PersonValidator {
    String NAME_ERR = "Invalid characters in name: ";
    String AGE_ERR = "Age must be at least 0";

    public Validation<Seq<String>, Person> validatePerson(
      String name, int age) {
        return Validation.combine(
          validateName(name), validateAge(age)).ap(Person::new);
    }

    private Validation<String, String> validateName(String name) {
        String invalidChars = name.replaceAll("[a-zA-Z ]", "");
        return invalidChars.isEmpty() ? 
          Validation.valid(name) 
            : Validation.invalid(NAME_ERR + invalidChars);
    }

    private Validation<String, Integer> validateAge(int age) {
        return age < 0 ? Validation.invalid(AGE_ERR)
          : Validation.valid(age);
    }
}
```

`age`的规则是它应该是大于 0 的整数，而`name`的规则是它不应该包含特殊字符:

```java
@Test
public void whenValidationWorks_thenCorrect() {
    PersonValidator personValidator = new PersonValidator();

    Validation<List<String>, Person> valid = 
      personValidator.validatePerson("John Doe", 30);

    Validation<List<String>, Person> invalid = 
      personValidator.validatePerson("John? Doe!4", -1);

    assertEquals(
      "Valid(Person [name=John Doe, age=30])", 
        valid.toString());

    assertEquals(
      "Invalid(List(Invalid characters in name: ?!4, 
        Age must be at least 0))", 
          invalid.toString());
}
```

**有效值包含在`Validation.Valid`实例中，验证错误列表包含在`Validation.Invalid`实例**中。所以任何验证方法都必须返回两者之一。

在`Validation.Valid`里面是一个`Person`的实例，而在`Validation.Invalid`里面是一个错误列表。

## 8。懒惰

`Lazy`是一个容器，它表示一个延迟计算的值，即计算被推迟到需要结果时。此外，评估值被缓存或存储，并在每次需要时反复返回，而无需重复计算:

```java
@Test
public void givenFunction_whenEvaluatesWithLazy_thenCorrect() {
    Lazy<Double> lazy = Lazy.of(Math::random);
    assertFalse(lazy.isEvaluated());

    double val1 = lazy.get();
    assertTrue(lazy.isEvaluated());

    double val2 = lazy.get();
    assertEquals(val1, val2, 0.1);
}
```

在上面的例子中，我们正在评估的函数是`Math.random`。请注意，在第二行中，我们检查了值，并意识到该函数尚未执行。这是因为我们仍然对返回值不感兴趣。

在第三行代码中，我们通过调用`Lazy.get`来显示对计算值的兴趣。此时，函数执行并且`Lazy.evaluated`返回 true。

我们还继续通过再次尝试`get`值来确认`Lazy`的记忆位。如果我们提供的函数再次执行，我们肯定会收到一个不同的随机数。

然而，`Lazy`再次延迟返回最初计算的值，因为最终断言确认了这一点。

## 9。模式匹配

模式匹配是几乎所有函数式编程语言的固有概念。Java 里暂时没有这种东西。

相反，每当我们想要基于接收到的输入执行计算或返回值时，我们使用多个`if`语句来解析要执行的正确代码:

```java
@Test
public void whenIfWorksAsMatcher_thenCorrect() {
    int input = 3;
    String output;
    if (input == 0) {
        output = "zero";
    }
    if (input == 1) {
        output = "one";
    }
    if (input == 2) {
        output = "two";
    }
    if (input == 3) {
        output = "three";
    }
    else {
        output = "unknown";
    }

    assertEquals("three", output);
}
```

我们可以突然看到代码跨越多行，而只检查三种情况。每个检查占用三行代码。如果我们必须检查多达 100 个案例，那将是大约 300 行，这可不好！

另一种方法是使用`switch`语句:

```java
@Test
public void whenSwitchWorksAsMatcher_thenCorrect() {
    int input = 2;
    String output;
    switch (input) {
    case 0:
        output = "zero";
        break;
    case 1:
        output = "one";
        break;
    case 2:
        output = "two";
        break;
    case 3:
        output = "three";
        break;
    default:
        output = "unknown";
        break;
    }

    assertEquals("two", output);
}
```

没有任何好转。我们仍然平均每次检查 3 行。很多混乱和潜在的错误。忘记一个`break`子句在编译时不是问题，但会导致以后难以发现的错误。

在 Vavr 中，我们用一个`Match`方法替换整个`switch`块。每个`case`或`if`语句都被一个`Case`方法调用所替代。

最后，像`$()`这样的原子模式取代了条件，然后条件计算一个表达式或值。我们还将其作为第二个参数提供给`Case`:

```java
@Test
public void whenMatchworks_thenCorrect() {
    int input = 2;
    String output = Match(input).of(
      Case($(1), "one"), 
      Case($(2), "two"), 
      Case($(3), "three"),
      Case($(), "?"));

    assertEquals("two", output);
}
```

注意这段代码有多紧凑，每次检查平均只有一行。模式匹配 API 比这更强大，可以做更复杂的事情。

例如，我们可以用谓词代替原子表达式。假设我们正在解析控制台命令的`help`和`version`标志:

```java
Match(arg).of(
    Case($(isIn("-h", "--help")), o -> run(this::displayHelp)),
    Case($(isIn("-v", "--version")), o -> run(this::displayVersion)),
    Case($(), o -> run(() -> {
        throw new IllegalArgumentException(arg);
    }))
);
```

一些用户可能更熟悉简写版本(-v)，而另一些用户则更熟悉完整版本(–version)。一个好的设计师必须考虑所有这些情况。

不需要几个`if`语句，我们已经处理了多个条件。我们将在另一篇文章中学习更多关于模式匹配中的谓词、多条件和副作用。

## 10。结论

在本文中，我们介绍了 Vavr，这是一个流行的 Java 8 函数式编程库。我们已经解决了可以快速调整以改进代码的主要特性。

本文的完整源代码可以在 [Github 项目](https://web.archive.org/web/20221017183302/https://github.com/eugenp/tutorials/tree/master/vavr-modules/vavr)中找到。