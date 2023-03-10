# Java 8 指南可选

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-optional>

## 1。概述

在本教程中，我们将展示 Java 8 中引入的`Optional`类。

该类的目的是提供一个类型级的解决方案来表示可选值而不是`null`引用。

为了更深入地理解为什么我们应该关心`Optional`类，看一看[官方甲骨文文章](https://web.archive.org/web/20220626102211/http://www.oracle.com/technetwork/articles/java/java8-optional-2175753.html)。

## 延伸阅读:

## [Java 可选为返回类型](/web/20220626102211/https://www.baeldung.com/java-optional-return)

Learn the best practices and when to return the Optional type in Java.[Read more](/web/20220626102211/https://www.baeldung.com/java-optional-return) →

## [Java 可选 orelse()与 orelsegue()](/web/20220626102211/https://www.baeldung.com/java-optional-or-else-vs-or-else-get)的比较

Explore the differences between Optional orElse() and OrElseGet() methods.[Read more](/web/20220626102211/https://www.baeldung.com/java-optional-or-else-vs-or-else-get) →

## [过滤 Java 中的可选流](/web/20220626102211/https://www.baeldung.com/java-filter-stream-of-optional)

A quick and practical guide to filtering Streams of Optionals in Java 8 and Java 9[Read more](/web/20220626102211/https://www.baeldung.com/java-filter-stream-of-optional) →

## 2。创建`Optional`对象

创建`Optional`对象有几种方法。

要创建一个空的`Optional`对象，我们只需要使用它的`empty()`静态方法:

```java
@Test
public void whenCreatesEmptyOptional_thenCorrect() {
    Optional<String> empty = Optional.empty();
    assertFalse(empty.isPresent());
}
```

注意，我们使用了`isPresent()`方法来检查`Optional`对象中是否有值。只有当我们用非`null`值创建了`Optional`时，该值才存在。我们将在下一节研究`isPresent()`方法。

我们也可以用静态方法`of()`创建一个`Optional`对象:

```java
@Test
public void givenNonNull_whenCreatesNonNullable_thenCorrect() {
    String name = "baeldung";
    Optional<String> opt = Optional.of(name);
    assertTrue(opt.isPresent());
}
```

然而，传递给`of()`方法的参数不能是`null.`，否则，我们将得到一个`NullPointerException`:

```java
@Test(expected = NullPointerException.class)
public void givenNull_whenThrowsErrorOnCreate_thenCorrect() {
    String name = null;
    Optional.of(name);
}
```

但是如果我们期望一些`null`值，我们可以使用`ofNullable()`方法:

```java
@Test
public void givenNonNull_whenCreatesNullable_thenCorrect() {
    String name = "baeldung";
    Optional<String> opt = Optional.ofNullable(name);
    assertTrue(opt.isPresent());
}
```

通过这样做，如果我们传入一个`null`引用，它不会抛出异常，而是返回一个空的`Optional`对象:

```java
@Test
public void givenNull_whenCreatesNullable_thenCorrect() {
    String name = null;
    Optional<String> opt = Optional.ofNullable(name);
    assertFalse(opt.isPresent());
}
```

## 3。检查值存在:`isPresent()` 和`isEmpty()`

当我们有一个从方法返回的或者我们自己创建的`Optional`对象时，我们可以用`isPresent()`方法检查它是否有值:

```java
@Test
public void givenOptional_whenIsPresentWorks_thenCorrect() {
    Optional<String> opt = Optional.of("Baeldung");
    assertTrue(opt.isPresent());

    opt = Optional.ofNullable(null);
    assertFalse(opt.isPresent());
}
```

如果包装的值不是`null.`，该方法返回`true`

同样，从 Java 11 开始，我们可以用`isEmpty `方法做相反的事情:

```java
@Test
public void givenAnEmptyOptional_thenIsEmptyBehavesAsExpected() {
    Optional<String> opt = Optional.of("Baeldung");
    assertFalse(opt.isEmpty());

    opt = Optional.ofNullable(null);
    assertTrue(opt.isEmpty());
}
```

## 4。`ifPresent()`有条件动作用

`ifPresent()`方法使我们能够对包装的值运行一些代码，如果发现它不是`null`的话。在`Optional`之前，我们会做:

```java
if(name != null) {
    System.out.println(name.length());
}
```

这段代码检查名称变量是否为`null`，然后对其执行一些代码。这种方法很冗长，而且这不是唯一的问题——它还容易出错。

事实上，什么能保证在打印完变量后，我们不会再次使用它，然后**忘记执行空值检查？**

**如果一个空值出现在代码中，那么在运行时会导致一个`NullPointerException`。**当一个程序由于输入问题而失败时，这通常是糟糕的编程实践的结果。

使我们明确地将可空值作为一种实施良好编程实践的方式来处理。

现在让我们看看如何在 Java 8 中重构上面的代码。

在典型的函数式编程风格中，我们可以对实际存在的对象执行一个动作:

```java
@Test
public void givenOptional_whenIfPresentWorks_thenCorrect() {
    Optional<String> opt = Optional.of("baeldung");
    opt.ifPresent(name -> System.out.println(name.length()));
}
```

在上面的例子中，我们只用了两行代码来代替第一个例子中的五行代码:一行将对象包装成一个`Optional`对象，下一行执行隐式验证和代码。

## 5。默认值用`orElse()`

`orElse()`方法用于检索包装在`Optional`实例中的值。它接受一个参数，作为默认值。`orElse()` 方法返回包装的值，如果它存在，否则返回它的参数:

```java
@Test
public void whenOrElseWorks_thenCorrect() {
    String nullName = null;
    String name = Optional.ofNullable(nullName).orElse("john");
    assertEquals("john", name);
}
```

## 6。默认值用`orElseGet()`

`orElseGet()`方法类似于`orElse()`。然而，如果`Optional`值不存在，它并不返回一个值，而是采用一个供应商功能接口，该接口被调用并返回调用的值:

```java
@Test
public void whenOrElseGetWorks_thenCorrect() {
    String nullName = null;
    String name = Optional.ofNullable(nullName).orElseGet(() -> "john");
    assertEquals("john", name);
}
```

## 7。`orElse`与`orElseGet()`和的区别

对于许多不熟悉`Optional`或 Java 8 的程序员来说，`orElse()`和`orElseGet()`之间的区别并不明显。事实上，这两种方法给人的印象是它们在功能上相互重叠。

然而，这两者之间有一个微妙但非常重要的区别，如果不能很好地理解，它会极大地影响我们代码的性能。

让我们在 test 类中创建一个名为`getMyDefault()`的方法，它没有参数并返回一个默认值:

```java
public String getMyDefault() {
    System.out.println("Getting Default Value");
    return "Default Value";
}
```

让我们来看两个测试，观察它们的副作用，以确定`orElse()`和`orElseGet()` 的重叠之处和不同之处:

```java
@Test
public void whenOrElseGetAndOrElseOverlap_thenCorrect() {
    String text = null;

    String defaultText = Optional.ofNullable(text).orElseGet(this::getMyDefault);
    assertEquals("Default Value", defaultText);

    defaultText = Optional.ofNullable(text).orElse(getMyDefault());
    assertEquals("Default Value", defaultText);
}
```

在上面的例子中，我们将一个空文本包装在一个`Optional`对象中，并尝试使用两种方法中的每一种来获取包装后的值。

副作用是:

```java
Getting default value...
Getting default value...
```

在每种情况下都会调用`getMyDefault()`方法。碰巧的是，当包装的值不存在时，`orElse()`和`orElseGet()`以完全相同的方式工作。

现在让我们运行另一个测试，这个值是存在的，理想情况下，甚至不应该创建默认值:

```java
@Test
public void whenOrElseGetAndOrElseDiffer_thenCorrect() {
    String text = "Text present";

    System.out.println("Using orElseGet:");
    String defaultText 
      = Optional.ofNullable(text).orElseGet(this::getMyDefault);
    assertEquals("Text present", defaultText);

    System.out.println("Using orElse:");
    defaultText = Optional.ofNullable(text).orElse(getMyDefault());
    assertEquals("Text present", defaultText);
}
```

在上面的例子中，我们不再包装一个`null`值，代码的其余部分保持不变。

现在让我们来看看运行这段代码的副作用:

```java
Using orElseGet:
Using orElse:
Getting default value...
```

请注意，当使用`orElseGet()`检索包装的值时，甚至不会调用`getMyDefault()`方法，因为包含的值已经存在。

但是，当使用`orElse()`时，无论包装值是否存在，都会创建默认对象。因此，在这种情况下，我们只是创建了一个从不使用的冗余对象。

在这个简单的例子中，创建一个默认对象并没有很大的开销，因为 JVM 知道如何处理这种情况。**但是，当`getMyDefault()`这样的方法要进行 web 服务调用甚至查询数据库时，代价就变得非常明显了。**

## 8。`orElseThrow()`例外有

`orElseThrow()`方法继承了`orElse()`和`orElseGet()`，并增加了一种处理缺失值的新方法。

当包装值不存在时，它不返回默认值，而是引发一个异常:

```java
@Test(expected = IllegalArgumentException.class)
public void whenOrElseThrowWorks_thenCorrect() {
    String nullName = null;
    String name = Optional.ofNullable(nullName).orElseThrow(
      IllegalArgumentException::new);
}
```

Java 8 中的方法引用在这里派上了用场，可以传入异常构造函数。

**Java 10 引入了一个简化的无参数版本的`orElseThrow()`方法**。如果`Optional`为空，它会抛出一个`NoSuchElementException`:

```java
@Test(expected = NoSuchElementException.class)
public void whenNoArgOrElseThrowWorks_thenCorrect() {
    String nullName = null;
    String name = Optional.ofNullable(nullName).orElseThrow();
}
```

## 9。用`get()` 返回值

检索包装值的最后一种方法是`get()`方法:

```java
@Test
public void givenOptional_whenGetsValue_thenCorrect() {
    Optional<String> opt = Optional.of("baeldung");
    String name = opt.get();
    assertEquals("baeldung", name);
}
```

但是，与前三种方法不同，`get()`只能在包装的对象不是`null`时返回值；否则，它会抛出一个 no 这样的元素异常:

```java
@Test(expected = NoSuchElementException.class)
public void givenOptionalWithNull_whenGetThrowsException_thenCorrect() {
    Optional<String> opt = Optional.ofNullable(null);
    String name = opt.get();
}
```

这是`get()`方法的主要缺陷。理想情况下，`Optional`应该帮助我们避免这种不可预见的异常。因此，这种方法违背了`Optional`的目标，可能会在未来的版本中被否决。

因此，建议使用其他变体，使我们能够准备并显式处理`null`情况。

## 10。有条件返回用`filter()`

我们可以用`filter`方法对包装后的值进行内联测试。它接受一个谓词作为参数，并返回一个`Optional`对象。如果包装的值通过了谓词的测试，那么`Optional`将按原样返回。

但是，如果谓词返回`false`，那么它将返回一个空的`Optional`:

```java
@Test
public void whenOptionalFilterWorks_thenCorrect() {
    Integer year = 2016;
    Optional<Integer> yearOptional = Optional.of(year);
    boolean is2016 = yearOptional.filter(y -> y == 2016).isPresent();
    assertTrue(is2016);
    boolean is2017 = yearOptional.filter(y -> y == 2017).isPresent();
    assertFalse(is2017);
}
```

`filter`方法通常用于根据预定义的规则拒绝包装的值。我们可以用它来拒绝错误的电子邮件格式或不够强的密码。

让我们看另一个有意义的例子。假设我们想买一个调制解调器，我们只关心它的价格。

我们从某个网站接收关于调制解调器价格的推送通知，并将这些通知存储在对象中:

```java
public class Modem {
    private Double price;

    public Modem(Double price) {
        this.price = price;
    }
    // standard getters and setters
}
```

然后，我们将这些对象提供给一些代码，这些代码的唯一目的是检查调制解调器的价格是否在我们的预算范围内。

现在让我们看看没有`Optional`的代码:

```java
public boolean priceIsInRange1(Modem modem) {
    boolean isInRange = false;

    if (modem != null && modem.getPrice() != null 
      && (modem.getPrice() >= 10 
        && modem.getPrice() <= 15)) {

        isInRange = true;
    }
    return isInRange;
}
```

注意我们要写多少代码才能实现这一点，尤其是在`if`条件下。对于应用程序来说，`if`条件中唯一关键的部分是最后一次价格范围检查；其余的检查是防御性的:

```java
@Test
public void whenFiltersWithoutOptional_thenCorrect() {
    assertTrue(priceIsInRange1(new Modem(10.0)));
    assertFalse(priceIsInRange1(new Modem(9.9)));
    assertFalse(priceIsInRange1(new Modem(null)));
    assertFalse(priceIsInRange1(new Modem(15.5)));
    assertFalse(priceIsInRange1(null));
}
```

除此之外，有可能在一整天的时间里忘记空检查，而不会出现任何编译时错误。

现在我们来看一个带有`Optional#filter`的变体:

```java
public boolean priceIsInRange2(Modem modem2) {
     return Optional.ofNullable(modem2)
       .map(Modem::getPrice)
       .filter(p -> p >= 10)
       .filter(p -> p <= 15)
       .isPresent();
 }
```

**`map`调用只是用来将一个值转换成其他值。**记住，这个操作不会修改原始值。

在我们的例子中，我们从`Model`类获得一个价格对象。我们将在下一节详细讨论`map()`方法。

首先，如果一个`null`对象被传递给这个方法，我们预计不会有任何问题。

其次，我们在它的主体中编写的唯一逻辑正是方法名所描述的——价格范围检查。其他的由你负责:

```java
@Test
public void whenFiltersWithOptional_thenCorrect() {
    assertTrue(priceIsInRange2(new Modem(10.0)));
    assertFalse(priceIsInRange2(new Modem(9.9)));
    assertFalse(priceIsInRange2(new Modem(null)));
    assertFalse(priceIsInRange2(new Modem(15.5)));
    assertFalse(priceIsInRange2(null));
}
```

以前的方法承诺检查价格范围，但必须做更多的事情来抵御其固有的脆弱性。因此，我们可以使用`filter`方法来替换不必要的`if`语句，拒绝不需要的值。

## 11。用`map()`转换值

在上一节中，我们看了如何基于过滤器拒绝或接受一个值。

我们可以使用类似的语法通过`map()` 方法来转换`Optional`值:

```java
@Test
public void givenOptional_whenMapWorks_thenCorrect() {
    List<String> companyNames = Arrays.asList(
      "paypal", "oracle", "", "microsoft", "", "apple");
    Optional<List<String>> listOptional = Optional.of(companyNames);

    int size = listOptional
      .map(List::size)
      .orElse(0);
    assertEquals(6, size);
}
```

在这个例子中，我们将一个字符串列表包装在一个`Optional`对象中，并使用它的`map`方法对包含的列表执行一个操作。我们执行的操作是检索列表的大小。

`map`方法返回包装在`Optional`中的计算结果。然后，我们必须对返回的`Optional`调用适当的方法来检索它的值。

请注意，`filter`方法只是对值执行检查，并返回一个描述该值的`Optional`,前提是它匹配给定的谓词。否则返回一个空的`Optional.`。然而`map`方法获取现有值，使用该值执行计算，并返回包装在`Optional`对象中的计算结果:

```java
@Test
public void givenOptional_whenMapWorks_thenCorrect2() {
    String name = "baeldung";
    Optional<String> nameOptional = Optional.of(name);

    int len = nameOptional
     .map(String::length)
     .orElse(0);
    assertEquals(8, len);
}
```

我们可以将`map`和`filter`链接在一起，做一些更强大的事情。

假设我们想要检查用户输入的密码的正确性。我们可以使用`map`转换清理密码，并使用`filter`检查其正确性:

```java
@Test
public void givenOptional_whenMapWorksWithFilter_thenCorrect() {
    String password = " password ";
    Optional<String> passOpt = Optional.of(password);
    boolean correctPassword = passOpt.filter(
      pass -> pass.equals("password")).isPresent();
    assertFalse(correctPassword);

    correctPassword = passOpt
      .map(String::trim)
      .filter(pass -> pass.equals("password"))
      .isPresent();
    assertTrue(correctPassword);
}
```

正如我们所看到的，如果不首先清理输入，它将被过滤掉——然而用户可能想当然地认为前导和尾随空格都构成了输入。因此，在过滤掉不正确的密码之前，我们将一个不干净的密码转换成一个带有`map`的干净密码。

## 12。用`flatMap()`转换值

就像`map()`方法一样，我们也有`flatMap()`方法作为转换值的替代方法。不同之处在于，`map`仅在值被解包时才转换值，而`flatMap`在转换值之前获取一个被包装的值并将其解包。

之前，我们创建了简单的`String`和`Integer`对象，用于包装在`Optional`实例中。然而，我们经常会从复杂对象的访问器接收这些对象。

为了更清楚地了解这种差异，让我们来看一个`Person`对象，它记录一个人的详细信息，如姓名、年龄和密码:

```java
public class Person {
    private String name;
    private int age;
    private String password;

    public Optional<String> getName() {
        return Optional.ofNullable(name);
    }

    public Optional<Integer> getAge() {
        return Optional.ofNullable(age);
    }

    public Optional<String> getPassword() {
        return Optional.ofNullable(password);
    }

    // normal constructors and setters
}
```

我们通常会创建这样一个对象，并将其包装在一个`Optional`对象中，就像我们处理 String 一样。

或者，它可以通过另一个方法调用返回给我们:

```java
Person person = new Person("john", 26);
Optional<Person> personOptional = Optional.of(person);
```

请注意，当我们包装一个`Person`对象时，它将包含嵌套的`Optional`实例:

```java
@Test
public void givenOptional_whenFlatMapWorks_thenCorrect2() {
    Person person = new Person("john", 26);
    Optional<Person> personOptional = Optional.of(person);

    Optional<Optional<String>> nameOptionalWrapper  
      = personOptional.map(Person::getName);
    Optional<String> nameOptional  
      = nameOptionalWrapper.orElseThrow(IllegalArgumentException::new);
    String name1 = nameOptional.orElse("");
    assertEquals("john", name1);

    String name = personOptional
      .flatMap(Person::getName)
      .orElse("");
    assertEquals("john", name);
}
```

这里，我们试图检索`Person`对象的 name 属性来执行断言。

注意我们如何在第三个语句中用`map()`方法实现这一点，然后注意我们如何在之后用`flatMap()`方法做同样的事情。

`Person::getName`方法引用类似于我们在上一节中清除密码的`String::trim`调用。

唯一的区别是`getName()`返回一个`Optional`，而不是像`trim()`操作那样返回一个字符串。这一点，加上一个`map`转换将结果包装在一个`Optional`对象中的事实，导致了一个嵌套的`Optional`。

因此，在使用`map()`方法时，我们需要添加一个额外的调用来在使用转换后的值之前检索值。这样，`Optional`包装将被移除。使用`flatMap`时，该操作隐式执行。

## 13.Java 8 中的链接

有时，我们可能需要从多个`Optional`中获取第一个非空的`Optional`对象。在这种情况下，使用像`orElseOptional()`这样的方法会非常方便。不幸的是，Java 8 不直接支持这样的操作。

让我们首先介绍几个我们将在本节中使用的方法:

```java
private Optional<String> getEmpty() {
    return Optional.empty();
}

private Optional<String> getHello() {
    return Optional.of("hello");
}

private Optional<String> getBye() {
    return Optional.of("bye");
}

private Optional<String> createOptional(String input) {
    if (input == null || "".equals(input) || "empty".equals(input)) {
        return Optional.empty();
    }
    return Optional.of(input);
}
```

为了链接几个`Optional`对象并在 Java 8 中获得第一个非空对象，我们可以使用`Stream` API:

```java
@Test
public void givenThreeOptionals_whenChaining_thenFirstNonEmptyIsReturned() {
    Optional<String> found = Stream.of(getEmpty(), getHello(), getBye())
      .filter(Optional::isPresent)
      .map(Optional::get)
      .findFirst();

    assertEquals(getHello(), found);
}
```

这种方法的缺点是我们所有的`get` 方法总是被执行，不管非空的`Optional`出现在`Stream`的什么地方。

如果我们想延迟评估传递给`Stream.of()`的方法，我们需要使用方法引用和`Supplier`接口:

```java
@Test
public void givenThreeOptionals_whenChaining_thenFirstNonEmptyIsReturnedAndRestNotEvaluated() {
    Optional<String> found =
      Stream.<Supplier<Optional<String>>>of(this::getEmpty, this::getHello, this::getBye)
        .map(Supplier::get)
        .filter(Optional::isPresent)
        .map(Optional::get)
        .findFirst();

    assertEquals(getHello(), found);
}
```

如果我们需要使用带参数的方法，我们必须求助于 lambda 表达式:

```java
@Test
public void givenTwoOptionalsReturnedByOneArgMethod_whenChaining_thenFirstNonEmptyIsReturned() {
    Optional<String> found = Stream.<Supplier<Optional<String>>>of(
      () -> createOptional("empty"),
      () -> createOptional("hello")
    )
      .map(Supplier::get)
      .filter(Optional::isPresent)
      .map(Optional::get)
      .findFirst();

    assertEquals(createOptional("hello"), found);
}
```

通常，我们会希望返回一个默认值，以防所有链接的`Optional`都是空的。我们可以通过添加对`orElse()`或`orElseGet()`的呼叫来实现:

```java
@Test
public void givenTwoEmptyOptionals_whenChaining_thenDefaultIsReturned() {
    String found = Stream.<Supplier<Optional<String>>>of(
      () -> createOptional("empty"),
      () -> createOptional("empty")
    )
      .map(Supplier::get)
      .filter(Optional::isPresent)
      .map(Optional::get)
      .findFirst()
      .orElseGet(() -> "default");

    assertEquals("default", found);
}
```

## 14\. JDK 9 `Optional` API

Java 9 的发布给`Optional` API 增加了更多的新方法:

*   `or()`提供创建替代方案的供应商的方法`Optional`
*   `ifPresentOrElse()`方法，允许在`Optional`出现时执行一个动作，否则执行另一个动作
*   `stream()`将`Optional`转换为`Stream`的方法

这里是[进一步阅读](/web/20220626102211/https://www.baeldung.com/java-9-optional)的完整文章。

## 15.误用`Optional` s

最后，让我们来看看一个诱人但危险的使用`Optional` s 的方法:将一个`Optional`参数传递给一个方法。

假设我们有一个`Person`列表，我们想要一个方法在列表中搜索具有给定名字的人。此外，我们希望该方法能够匹配至少具有特定年龄的条目，如果指定了年龄的话。

这个参数是可选的，我们用这个方法:

```java
public static List<Person> search(List<Person> people, String name, Optional<Integer> age) {
    // Null checks for people and name
    return people.stream()
            .filter(p -> p.getName().equals(name))
            .filter(p -> p.getAge().get() >= age.orElse(0))
            .collect(Collectors.toList());
}
```

然后我们发布我们的方法，另一个开发人员试图使用它:

```java
someObject.search(people, "Peter", null);
```

现在开发人员执行它的代码并得到一个`NullPointerException.` **，我们不得不空检查我们的可选参数，这违背了我们想要避免这种情况的最初目的。**

以下是我们本可以做得更好的一些可能性:

```java
public static List<Person> search(List<Person> people, String name, Integer age) {
    // Null checks for people and name
    final Integer ageFilter = age != null ? age : 0;

    return people.stream()
            .filter(p -> p.getName().equals(name))
            .filter(p -> p.getAge().get() >= ageFilter)
            .collect(Collectors.toList());
}
```

这里，参数仍然是可选的，但是我们只在一次检查中处理它。

另一种可能性是**创建两个重载方法**:

```java
public static List<Person> search(List<Person> people, String name) {
    return doSearch(people, name, 0);
}

public static List<Person> search(List<Person> people, String name, int age) {
    return doSearch(people, name, age);
}

private static List<Person> doSearch(List<Person> people, String name, int age) {
    // Null checks for people and name
    return people.stream()
            .filter(p -> p.getName().equals(name))
            .filter(p -> p.getAge().get().intValue() >= age)
            .collect(Collectors.toList());
}
```

这样我们就提供了一个清晰的 API，两个方法做不同的事情(尽管它们共享实现)。

因此，有一些解决方案可以避免使用`Optional` s 作为方法参数。**Java 在发布`Optional`时的意图是将其用作返回类型**，从而表明一个方法可以返回一个空值。事实上，使用`Optional`作为方法参数的做法甚至被一些代码检查员劝阻[。](https://web.archive.org/web/20220626102211/https://rules.sonarsource.com/java/RSPEC-3553)

## 16.`Optional`连载

如上所述，`Optional`意在用作返回类型。不建议尝试将其用作字段类型。

另外，**在一个可序列化的类中使用`Optional`将导致一个`NotSerializableException`。**我们的文章 [Java `Optional`作为返回类型](/web/20220626102211/https://www.baeldung.com/java-optional-return)进一步解决了序列化的问题。

并且，在将`Optional`与 Jackson 一起使用的[中，我们解释了当`Optional`字段被序列化时会发生什么，以及一些实现预期结果的变通方法。](/web/20220626102211/https://www.baeldung.com/jackson-optional)

## 17。结论

在本文中，我们介绍了 Java 8 `Optional`类的大部分重要特性。

我们简要探讨了为什么我们会选择使用`Optional`而不是显式的空值检查和输入验证的一些原因。

我们还学习了如何使用`get()`、`orElse()`和 `orElseGet()` 方法获取`Optional`的值，或者如果为空，则获取默认值(并且看到了[与最后两个](/web/20220626102211/https://www.baeldung.com/java-filter-stream-of-optional)的重要区别)。

然后我们看到了如何用`map(), flatMap()` 和`filter()`来转换或过滤我们的`Optional`。我们讨论了流畅的`API` `Optional` 提供了什么，因为它允许我们轻松地链接不同的方法。

最后，我们看到了为什么使用`Optional` s 作为方法参数是一个坏主意，以及如何避免它。

文章中所有例子的源代码都可以在 GitHub 上找到。