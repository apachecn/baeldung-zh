# JUnit 5 参数化测试指南

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/parameterized-tests-junit-5>

## 1。概述

JUnit 5 ，JUnit 的下一代，用闪亮的新特性帮助开发人员编写测试。

其中一个这样的特征就是`p` `arameterized tests`。这个特性使我们能够**用不同的参数多次执行一个测试方法。**

在本教程中，我们将深入探索参数化测试，所以让我们开始吧。

## 延伸阅读:

## JUnit 5 指南

A quick and practical guide to JUnit 5[Read more](/web/20221102025235/https://www.baeldung.com/junit-5) →

## [使用带参数化的 spring JUnit 4 class runner](/web/20221102025235/https://www.baeldung.com/springjunit4classrunner-parameterized)

Learn how to use the Parameterized JUnit test runner with a Spring integration test[Read more](/web/20221102025235/https://www.baeldung.com/springjunit4classrunner-parameterized) →

## [JUnit params 简介](/web/20221102025235/https://www.baeldung.com/junit-params)

A quick and practical guide to a very useful library which will help you write parameterized unit tests - JUnitParams.[Read more](/web/20221102025235/https://www.baeldung.com/junit-params) →

## 2。依赖关系

为了使用 JUnit 5 参数化测试，我们需要从 JUnit 平台导入 [`junit-jupiter-params`](https://web.archive.org/web/20221102025235/https://search.maven.org/search?q=a:junit-jupiter-params%20AND%20g:org.junit.jupiter) 工件。这意味着，当使用 Maven 时，我们将向我们的`pom.xml`添加以下内容:

```
<dependency>
    <groupId>org.junit.jupiter</groupId>
    <artifactId>junit-jupiter-params</artifactId>
    <version>5.8.1</version>
    <scope>test</scope>
</dependency>
```

此外，在使用 Gradle 时，我们将稍微不同地指定它:

```
testCompile("org.junit.jupiter:junit-jupiter-params:5.8.1")
```

## 3。第一印象

假设我们有一个现有的效用函数，我们希望对它的行为有信心:

```
public class Numbers {
    public static boolean isOdd(int number) {
        return number % 2 != 0;
    }
}
```

参数化测试与其他测试相似，除了我们添加了`@ParameterizedTest`注释:

```
@ParameterizedTest
@ValueSource(ints = {1, 3, 5, -3, 15, Integer.MAX_VALUE}) // six numbers
void isOdd_ShouldReturnTrueForOddNumbers(int number) {
    assertTrue(Numbers.isOdd(number));
}
```

JUnit 5 test runner 执行上述测试——以及随后的`isOdd `方法——六次。每一次，它都从`@ValueSource`数组中给`number`方法参数分配一个不同的值。

因此，这个例子向我们展示了参数化测试需要的两件事情:

*   **参数的来源**，在本例中，是一个`int`数组
*   **访问它们的方法**，在本例中是`number`参数

这个例子还有另一个不明显的方面，所以我们将继续观察。

## 4。论据来源

正如我们现在应该知道的，参数化测试使用不同的参数多次执行相同的测试。

我们希望能做的不仅仅是数字，所以让我们探索吧。

### 4.1.简单值

**使用`@ValueSource `注释，我们可以将一组文字值传递给测试方法。**

假设我们要测试我们简单的`isBlank`方法:

```
public class Strings {
    public static boolean isBlank(String input) {
        return input == null || input.trim().isEmpty();
    }
}
```

我们期望从这个方法中为空字符串的`null`返回`true`。因此，我们可以编写一个参数化的测试来断言这种行为:

```
@ParameterizedTest
@ValueSource(strings = {"", "  "})
void isBlank_ShouldReturnTrueForNullOrBlankStrings(String input) {
    assertTrue(Strings.isBlank(input));
} 
```

正如我们所看到的，JUnit 将运行这个测试两次，每次都将数组中的一个参数分配给方法参数。

价值源的局限性之一是它们仅支持以下类型:

*   `short`(带有`shorts`属性)
*   `byte` ( `bytes`属性)
*   `int` ( `ints`属性)
*   `long` ( `longs`属性)
*   `float` ( `floats`属性)
*   `double` ( `doubles`属性)
*   `char` ( `chars`属性)
*   `java.lang.String` ( `strings`属性)
*   `java.lang.Class` ( `classes`属性)

同样，**我们每次只能传递一个参数给测试方法。**

在继续之前，请注意我们没有将`null`作为参数传递。这是另一个限制— **我们不能通过`@ValueSource`传递`null`，即使是对于`String` 和`Class`。**

### 4.2.Null 和空值

**从 JUnit 5.4 开始，我们可以使用`@NullSource`** 将单个`null `值传递给参数化的测试方法:

```
@ParameterizedTest
@NullSource
void isBlank_ShouldReturnTrueForNullInputs(String input) {
    assertTrue(Strings.isBlank(input));
}
```

因为原始数据类型不能接受`null `值，所以我们不能将`@NullSource `用于原始参数。

同样，我们可以使用`@EmptySource `注释传递空值:

```
@ParameterizedTest
@EmptySource
void isBlank_ShouldReturnTrueForEmptyStrings(String input) {
    assertTrue(Strings.isBlank(input));
}
```

**`@EmptySource `向带注释的方法传递一个空参数。**

对于`String`参数来说，传递的值就像空的`String`一样简单。**此外，该参数源可以为`Collection`类型和数组提供空值。**

为了传递`null `和空值，我们可以使用组合的`@NullAndEmptySource `注释:

```
@ParameterizedTest
@NullAndEmptySource
void isBlank_ShouldReturnTrueForNullAndEmptyStrings(String input) {
    assertTrue(Strings.isBlank(input));
}
```

与`@EmptySource`一样，合成的注释适用于`String` s、`Collection` s 和数组`.`

为了将更多的空字符串变体传递给参数化测试，**我们可以将`@ValueSource`、 `@NullSource`、 `and @EmptySource`组合在一起**:

```
@ParameterizedTest
@NullAndEmptySource
@ValueSource(strings = {"  ", "\t", "\n"})
void isBlank_ShouldReturnTrueForAllTypesOfBlankStrings(String input) {
    assertTrue(Strings.isBlank(input));
}
```

### 4.3.列举型别

**为了用不同的枚举值运行测试，我们可以使用`@EnumSource`注释。**

例如，我们可以断言所有月份的数字都在 1 到 12 之间:

```
@ParameterizedTest
@EnumSource(Month.class) // passing all 12 months
void getValueForAMonth_IsAlwaysBetweenOneAndTwelve(Month month) {
    int monthNumber = month.getValue();
    assertTrue(monthNumber >= 1 && monthNumber <= 12);
}
```

或者，我们可以使用`names `属性过滤出几个月。

我们还可以断言，四月、九月、六月和十一月都是 30 天:

```
@ParameterizedTest
@EnumSource(value = Month.class, names = {"APRIL", "JUNE", "SEPTEMBER", "NOVEMBER"})
void someMonths_Are30DaysLong(Month month) {
    final boolean isALeapYear = false;
    assertEquals(30, month.length(isALeapYear));
}
```

默认情况下，`names`将只保留匹配的枚举值。

我们可以通过将`mode`属性设置为`EXCLUDE`来改变这种情况:

```
@ParameterizedTest
@EnumSource(
  value = Month.class,
  names = {"APRIL", "JUNE", "SEPTEMBER", "NOVEMBER", "FEBRUARY"},
  mode = EnumSource.Mode.EXCLUDE)
void exceptFourMonths_OthersAre31DaysLong(Month month) {
    final boolean isALeapYear = false;
    assertEquals(31, month.length(isALeapYear));
}
```

除了文字字符串，我们还可以将一个正则表达式传递给`names` 属性:

```
@ParameterizedTest
@EnumSource(value = Month.class, names = ".+BER", mode = EnumSource.Mode.MATCH_ANY)
void fourMonths_AreEndingWithBer(Month month) {
    EnumSet<Month> months =
      EnumSet.of(Month.SEPTEMBER, Month.OCTOBER, Month.NOVEMBER, Month.DECEMBER);
    assertTrue(months.contains(month));
}
```

与`@ValueSource`非常相似，`@EnumSource`只适用于每次测试执行只传递一个参数的情况。

### 4.4.CSV 文字

假设我们要确保来自`String`的 `toUpperCase()`方法生成预期的大写值。`@ValueSource `不够用。

要为这样的场景编写参数化测试，我们必须

*   将一个**输入值** `and`和一个**期望值**传递给测试方法
*   用这些输入值计算**实际结果**
*   **断言** **实际值与期望值**

因此，我们需要能够传递多个参数的参数源。

`@CsvSource`是这些来源之一:

```
@ParameterizedTest
@CsvSource({"test,TEST", "tEst,TEST", "Java,JAVA"})
void toUpperCase_ShouldGenerateTheExpectedUppercaseValue(String input, String expected) {
    String actualValue = input.toUpperCase();
    assertEquals(expected, actualValue);
}
```

`@CsvSource`接受逗号分隔值的数组，每个数组条目对应于 CSV 文件中的一行。

这个源每次获取一个数组条目，用逗号分割它，并将每个数组作为单独的参数传递给带注释的测试方法。

默认情况下，逗号是列分隔符，但是我们可以使用`delimiter` 属性对其进行定制:

```
@ParameterizedTest
@CsvSource(value = {"test:test", "tEst:test", "Java:java"}, delimiter = ':')
void toLowerCase_ShouldGenerateTheExpectedLowercaseValue(String input, String expected) {
    String actualValue = input.toLowerCase();
    assertEquals(expected, actualValue);
}
```

现在它是一个冒号分隔的值，所以仍然是一个 CSV。

### 4.5.CSV 文件

我们可以引用一个实际的 CSV 文件，而不是在代码中传递 CSV 值。

例如，我们可以使用这样的 CSV 文件:

```
input,expected
test,TEST
tEst,TEST
Java,JAVA
```

我们可以加载 CSV 文件并且**忽略标题列**和`@CsvFileSource`:

```
@ParameterizedTest
@CsvFileSource(resources = "/data.csv", numLinesToSkip = 1)
void toUpperCase_ShouldGenerateTheExpectedUppercaseValueCSVFile(
  String input, String expected) {
    String actualValue = input.toUpperCase();
    assertEquals(expected, actualValue);
}
```

`resources `属性表示要读取的类路径上的 CSV 文件资源。而且，我们可以传递多个文件给它。

`numLinesToSkip `属性表示读取 CSV 文件时要跳过的行数。**默认情况下，`@CsvFileSource `不跳过任何行，但是这个特性通常对于跳过标题行**很有用，就像我们在这里做的一样。

就像简单的`@CsvSource`一样，分隔符可以用`delimiter `属性定制。

除了列分隔符，我们还有以下功能:

*   可以使用`lineSeparator`属性定制行分隔符——缺省值是换行符。
*   使用`encoding`属性可以定制文件编码——UTF-8 是默认值。

### 4.6.方法

到目前为止，我们讨论的参数来源有些简单，并且都有一个限制。用它们很难或者不可能通过复杂的物体。

提供更复杂参数的一种方法是使用一个方法作为参数源。

让我们用一个`@MethodSource`来测试`isBlank `方法:

```
@ParameterizedTest
@MethodSource("provideStringsForIsBlank")
void isBlank_ShouldReturnTrueForNullOrBlankStrings(String input, boolean expected) {
    assertEquals(expected, Strings.isBlank(input));
}
```

我们提供给`@MethodSource`的名称需要匹配一个现有的方法。

所以，让我们接下来写一个`provideStringsForIsBlank`，**的`static `方法，返回一个`Argument`的 `Stream`**:

```
private static Stream<Arguments> provideStringsForIsBlank() {
    return Stream.of(
      Arguments.of(null, true),
      Arguments.of("", true),
      Arguments.of("  ", true),
      Arguments.of("not blank", false)
    );
}
```

这里我们实际上是返回一串参数，但这不是严格的要求。例如，**我们可以返回任何其他类似集合的接口，如** `**List.** `

如果我们要为每个测试调用提供一个参数，那么就没有必要使用`Arguments `抽象:

```
@ParameterizedTest
@MethodSource // hmm, no method name ...
void isBlank_ShouldReturnTrueForNullOrBlankStringsOneArgument(String input) {
    assertTrue(Strings.isBlank(input));
}

private static Stream<String> isBlank_ShouldReturnTrueForNullOrBlankStringsOneArgument() {
    return Stream.of(null, "", "  ");
}
```

**当我们没有为`@MethodSource`提供名称时，JUnit 将搜索与测试方法同名的源方法。**

有时，在不同的测试类之间共享参数是有用的。在这些情况下，我们可以通过完全限定名引用当前类之外的源方法:

```
class StringsUnitTest {

    @ParameterizedTest
    @MethodSource("com.baeldung.parameterized.StringParams#blankStrings")
    void isBlank_ShouldReturnTrueForNullOrBlankStringsExternalSource(String input) {
        assertTrue(Strings.isBlank(input));
    }
}

public class StringParams {

    static Stream<String> blankStrings() {
        return Stream.of(null, "", "  ");
    }
}
```

使用`FQN#methodName`格式，我们可以引用一个外部静态方法。

### 4.7.自定义参数提供程序

另一种传递测试参数的高级方法是使用一个名为`ArgumentsProvider`的接口的定制实现:

```
class BlankStringsArgumentsProvider implements ArgumentsProvider {

    @Override
    public Stream<? extends Arguments> provideArguments(ExtensionContext context) {
        return Stream.of(
          Arguments.of((String) null), 
          Arguments.of(""), 
          Arguments.of("   ") 
        );
    }
}
```

然后我们可以用`@ArgumentsSource `注释来注释我们的测试，以使用这个定制的提供者:

```
@ParameterizedTest
@ArgumentsSource(BlankStringsArgumentsProvider.class)
void isBlank_ShouldReturnTrueForNullOrBlankStringsArgProvider(String input) {
    assertTrue(Strings.isBlank(input));
}
```

让我们让定制提供者成为一个更好的 API，用于定制注释。

### 4.8.自定义注释

假设我们想从一个静态变量加载测试参数:

```
static Stream<Arguments> arguments = Stream.of(
  Arguments.of(null, true), // null strings should be considered blank
  Arguments.of("", true),
  Arguments.of("  ", true),
  Arguments.of("not blank", false)
);

@ParameterizedTest
@VariableSource("arguments")
void isBlank_ShouldReturnTrueForNullOrBlankStringsVariableSource(
  String input, boolean expected) {
    assertEquals(expected, Strings.isBlank(input));
}
```

实际上， **JUnit 5 并没有提供这一点。**但是，我们可以推出自己的解决方案。

首先，我们可以创建一个注释:

```
@Documented
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
@ArgumentsSource(VariableArgumentsProvider.class)
public @interface VariableSource {

    /**
     * The name of the static variable
     */
    String value();
}
```

然后，我们需要以某种方式**消费注释**细节和**提供测试参数。** JUnit 5 提供了两种抽象来实现这些目标:

*   `AnnotationConsumer `消费注释详情
*   `ArgumentsProvider `提供测试参数

因此，我们接下来需要让`VariableArgumentsProvider `类从指定的静态变量中读取，并将其值作为测试参数返回:

```
class VariableArgumentsProvider 
  implements ArgumentsProvider, AnnotationConsumer<VariableSource> {

    private String variableName;

    @Override
    public Stream<? extends Arguments> provideArguments(ExtensionContext context) {
        return context.getTestClass()
                .map(this::getField)
                .map(this::getValue)
                .orElseThrow(() -> 
                  new IllegalArgumentException("Failed to load test arguments"));
    }

    @Override
    public void accept(VariableSource variableSource) {
        variableName = variableSource.value();
    }

    private Field getField(Class<?> clazz) {
        try {
            return clazz.getDeclaredField(variableName);
        } catch (Exception e) {
            return null;
        }
    }

    @SuppressWarnings("unchecked")
    private Stream<Arguments> getValue(Field field) {
        Object value = null;
        try {
            value = field.get(null);
        } catch (Exception ignored) {}

        return value == null ? null : (Stream<Arguments>) value;
    }
}
```

它非常有效。

## 5。变元转换

### 5.1.隐式转换

让我们用@ `CsvSource`重写其中一个`@EnumTest`:

```
@ParameterizedTest
@CsvSource({"APRIL", "JUNE", "SEPTEMBER", "NOVEMBER"}) // Pssing strings
void someMonths_Are30DaysLongCsv(Month month) {
    final boolean isALeapYear = false;
    assertEquals(30, month.length(isALeapYear));
}
```

这似乎不应该工作，但它不知何故做到了。

JUnit 5 将`String `参数转换为指定的枚举类型。为了支持这样的用例，JUnit Jupiter 提供了许多内置的隐式类型转换器。

转换过程取决于每个方法参数声明的类型。隐式转换可以将`String`实例转换为如下类型:

*   `UUID `
*   `Locale`
*   `LocalDate`、 `LocalTime`、 `LocalDateTime`、 `Year`、 `Month`等。
*   `File `和`Path`
*   `URL `和`URI`
*   `Enum `子类

### 5.2.显式转换

我们有时需要为参数提供自定义的显式转换器。

假设我们想将带有`yyyy/mm/dd` 格式的字符串转换成`LocalDate`实例。

首先，我们需要实现`ArgumentConverter`接口:

```
class SlashyDateConverter implements ArgumentConverter {

    @Override
    public Object convert(Object source, ParameterContext context)
      throws ArgumentConversionException {
        if (!(source instanceof String)) {
            throw new IllegalArgumentException(
              "The argument should be a string: " + source);
        }
        try {
            String[] parts = ((String) source).split("/");
            int year = Integer.parseInt(parts[0]);
            int month = Integer.parseInt(parts[1]);
            int day = Integer.parseInt(parts[2]);

            return LocalDate.of(year, month, day);
        } catch (Exception e) {
            throw new IllegalArgumentException("Failed to convert", e);
        }
    }
}
```

然后我们应该通过`@ConvertWith `注释来引用转换器:

```
@ParameterizedTest
@CsvSource({"2018/12/25,2018", "2019/02/11,2019"})
void getYear_ShouldWorkAsExpected(
  @ConvertWith(SlashyDateConverter.class) LocalDate date, int expected) {
    assertEquals(expected, date.getYear());
}
```

## 6。参数存取器

默认情况下，提供给参数化测试的每个参数对应一个方法参数。因此，当通过一个参数源传递一些参数时，测试方法签名会变得非常大和混乱。

解决这个问题的一种方法是将所有传递的参数封装到一个`ArgumentsAccessor `实例中，并通过索引和类型检索参数。

让我们考虑一下我们的`Person`类:

```
class Person {

    String firstName;
    String middleName;
    String lastName;

    // constructor

    public String fullName() {
        if (middleName == null || middleName.trim().isEmpty()) {
            return String.format("%s %s", firstName, lastName);
        }

        return String.format("%s %s %s", firstName, middleName, lastName);
    }
}
```

为了测试`fullName()`方法，我们将传递四个参数:`firstName`、 `middleName`、 `lastName`和`expected fullName`。我们可以使用`ArgumentsAccessor `来检索测试参数，而不是将它们声明为方法参数:

```
@ParameterizedTest
@CsvSource({"Isaac,,Newton,Isaac Newton", "Charles,Robert,Darwin,Charles Robert Darwin"})
void fullName_ShouldGenerateTheExpectedFullName(ArgumentsAccessor argumentsAccessor) {
    String firstName = argumentsAccessor.getString(0);
    String middleName = (String) argumentsAccessor.get(1);
    String lastName = argumentsAccessor.get(2, String.class);
    String expectedFullName = argumentsAccessor.getString(3);

    Person person = new Person(firstName, middleName, lastName);
    assertEquals(expectedFullName, person.fullName());
}
```

这里，我们将所有传递的参数封装到一个`ArgumentsAccessor `实例中，然后在测试方法体中，检索每个传递的参数及其索引。除了作为一个访问器，类型转换还通过`get*`方法得到支持:

*   `getString(index) `检索特定索引处的元素，并将其转换为`String`——对于原始类型也是如此。
*   `get(index) `简单地检索特定索引处的元素作为`Object`。
*   `get(index, type) `检索特定索引处的元素，并将其转换为给定的`type`。

## 7。自变量聚合器

直接使用`ArgumentsAccessor `抽象可能会降低测试代码的可读性或可重用性。为了解决这些问题，我们可以编写一个自定义的、可重用的聚合器。

为此，我们实现了`ArgumentsAggregator `接口:

```
class PersonAggregator implements ArgumentsAggregator {

    @Override
    public Object aggregateArguments(ArgumentsAccessor accessor, ParameterContext context)
      throws ArgumentsAggregationException {
        return new Person(
          accessor.getString(1), accessor.getString(2), accessor.getString(3));
    }
}
```

然后我们通过`@AggregateWith `注释引用它:

```
@ParameterizedTest
@CsvSource({"Isaac Newton,Isaac,,Newton", "Charles Robert Darwin,Charles,Robert,Darwin"})
void fullName_ShouldGenerateTheExpectedFullName(
  String expectedFullName,
  @AggregateWith(PersonAggregator.class) Person person) {

    assertEquals(expectedFullName, person.fullName());
}
```

`PersonAggregator `获取最后三个参数，并从中实例化一个`Person `类。

## 8。自定义显示名称

默认情况下，参数化测试的显示名称包含一个调用索引以及所有传递参数的一个`String `表示:

```
├─ someMonths_Are30DaysLongCsv(Month)
│     │  ├─ [1] APRIL
│     │  ├─ [2] JUNE
│     │  ├─ [3] SEPTEMBER
│     │  └─ [4] NOVEMBER
```

然而，我们可以通过`@ParameterizedTest`注释的`name`属性定制这个显示:

```
@ParameterizedTest(name = "{index} {0} is 30 days long")
@EnumSource(value = Month.class, names = {"APRIL", "JUNE", "SEPTEMBER", "NOVEMBER"})
void someMonths_Are30DaysLong(Month month) {
    final boolean isALeapYear = false;
    assertEquals(30, month.length(isALeapYear));
}
```

`April is 30 days long` 肯定是一个更易读的显示名称:

```
├─ someMonths_Are30DaysLong(Month)
│     │  ├─ 1 APRIL is 30 days long
│     │  ├─ 2 JUNE is 30 days long
│     │  ├─ 3 SEPTEMBER is 30 days long
│     │  └─ 4 NOVEMBER is 30 days long
```

自定义显示名称时，以下占位符可用:

*   **{index}** 将被替换为调用索引。简单地说，第一次执行的调用索引是 1，第二次是 2，依此类推。
*   **{arguments}** 是以逗号分隔的完整参数列表的占位符。
*   **{0}，{1}，..`.`** 是个别自变量的占位符。

## 9。结论

在本文中，我们探讨了 JUnit 5 中参数化测试的具体细节。

我们了解到参数化测试在两个方面不同于普通测试:它们用`@ParameterizedTest`进行注释，并且它们需要一个声明参数的来源。

此外，到目前为止，我们应该知道 JUnit 提供了一些工具来将参数转换成定制的目标类型或者定制测试名称。

像往常一样，我们的 [GitHub](https://web.archive.org/web/20221102025235/https://github.com/eugenp/tutorials/tree/master/testing-modules/junit5-annotations) 项目中提供了示例代码，所以请务必查看一下。