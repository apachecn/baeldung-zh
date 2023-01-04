# 检查 Java 中是否存在枚举值

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-search-enum-values>

## 1.概观

我们几乎在每个应用程序中都可以看到枚举。这些包括订单状态代码，如`DRAFT` 和 `PROCESSING,`以及网络错误代码，如 400、404、500、501 等。每当我们在域中看到枚举数据时，我们会在应用程序中看到它的`Enum`。我们可以使用传入请求中的数据并找到该枚举。例如，我们可以将 web 错误`400`映射到`BAD_REQUEST`。

因此，我们需要逻辑来根据标准搜索枚举。这可能是它的名字或它的值。或者，它甚至可以是任意整数代码。

在本教程中，我们将学习如何根据标准搜索枚举。此外，我们还将探索返回找到的枚举的不同方法。

## 2.按名称搜索枚举

首先，我们知道枚举类型是一种特殊的数据类型。它使变量成为一组预定义的常数。让我们为方向定义一个枚举:

```java
public enum Direction {
    EAST, WEST, SOUTH, NORTH;
}
```

枚举值的名称是常量。例如，`Direction.EAST`的名字是`EAST`。现在我们可以通过它的名字来搜索方向。实现不区分大小写的搜索是个好主意。因此，`East`、`east`和`EAST`都将映射到`Direction.EAST`。让我们将下面的方法添加到`Direction`枚举中:

```java
public static Direction findByName(String name) {
    Direction result = null;
    for (Direction direction : values()) {
        if (direction.name().equalsIgnoreCase(name)) {
            result = direction;
            break;
        }
    }
    return result;
}
```

在这个实现中，如果我们没有找到给定名称的枚举，我们将返回`null`。如何对待找不到的场景取决于我们自己。一种选择是我们可以返回一个默认的枚举值。相反，我们可以抛出一个异常。我们很快会看到更多搜索枚举的例子。现在让我们测试我们的搜索逻辑。首先，积极的情况是:

```java
@Test
public void givenWeekdays_whenValidDirectionNameProvided_directionIsFound() {
    Direction result = Direction.findByName("EAST");
    assertThat(result).isEqualTo(Direction.EAST);
}
```

在本文的最后，我们将提供完整代码实现的链接，但是现在，我们将把重点放在代码片段上。在这里，我们搜索了名称“EAST”的方向，我们期望得到`Direction.EAST`。如前所述，我们知道搜索是不区分大小写的，所以我们应该对名称“east”或“East”得到相同的结果。让我们验证我们的期望:

```java
@Test
public void givenWeekdays_whenValidDirectionNameLowerCaseProvided_directionIsFound() {
    Direction result = Direction.findByName("east");
    assertThat(result).isEqualTo(Direction.EAST);
} 
```

我们还可以再添加一个测试来验证搜索方法是否为名称“East”返回相同的结果。下面的测试将说明我们对名称“East”得到了相同的结果。

```java
@Test public void givenWeekdays_whenValidDirectionNameLowerCaseProvided_directionIsFound() { 
    Direction result = Direction.findByName("East"); 
    assertThat(result).isEqualTo(Direction.EAST); 
}
```

## 3.按值搜索枚举

现在让我们为一周中的每一天定义一个枚举。这一次，让我们提供一个值和名称。事实上，我们可以在枚举中定义任何数据成员，然后将其用于我们的应用程序逻辑。下面是`Weekday`枚举的代码:

```java
public Weekday {
    MONDAY("Monday"),
    TUESDAY("Tuesday"),
    // ...
    SUNDAY("Sunday"),
    ;
    private final String value;

    Weekday(String value) {
        this.value = value;
    }
}
```

接下来，让我们通过值实现搜索。所以对于“星期一”，我们应该得到`Weekday.MONDAY`。让我们将下面的方法添加到枚举中:

```java
public static Weekday findByValue(String value) {
    Weekday result = null;
    for (Weekday day : values()) {
        if (day.getValue().equalsIgnoreCase(value)) {
            result = day;
            break;
        }
    }
    return result;
}
```

这里我们迭代枚举的常量，然后将输入的值与枚举的值成员进行比较。如前所述，我们忽略了值的大小写。现在我们可以测试它:

```java
@Test
public void givenWeekdays_whenValidWeekdayValueProvided_weekdayIsFound() {
    Weekday result = Weekday.findByValue("Monday");
    assertThat(result).isEqualTo(Weekday.MONDAY);
} 
```

如果我们不提供一个有效值，我们将得到`null`作为回报。让我们验证一下:

```java
@Test
public void givenWeekdays_whenInvalidWeekdayValueProvided_nullIsReturned() {
    Weekday result = Weekday.findByValue("mon");
    assertThat(result).isNull();
} 
```

搜索并不总是需要字符串值。这很不方便，因为我们必须首先将输入转换成字符串，然后将它传递给 search 方法。现在让我们看看如何通过非字符串值进行搜索，比如一个整数值。

## 4.按整数值搜索枚举

让我们定义一个名为`Month`的新枚举。下面是`Month`枚举的代码:

```java
public enum Month {
    JANUARY("January", 1),
    FEBRUARY("February", 2),
    // ...
    DECEMBER("December", 12),
    ;

    private final String value;
    private final int code;

    Month(String value, int code) {
        this.value = value;
        this.code = code;
    }
}
```

我们可以看到 month 枚举有两个成员，值和代码，代码是一个整数值。让我们实现按代码搜索月份的逻辑:

```java
public static Optional<Month> findByCode(int code) {
    return Arrays.stream(values()).filter(month -> month.getCode() == code).findFirst();
} 
```

这个搜索看起来与前面的搜索有些不同，因为我们使用了 Java 8 特性来演示实现搜索的另一种方式。这里，我们将返回枚举的一个`Optional`值，而不是返回枚举本身。类似地，我们将返回一个空的`Optional`，而不是`null`。所以如果我们搜索一个月的代码 1，我们应该得到`Month.JANUARY`。让我们通过测试来验证这一点:

```java
@Test
public void givenMonths_whenValidMonthCodeProvided_optionalMonthIsReturned() {
    Optional<Month> result = Month.findByCode(1);
    assertThat(result).isEqualTo(Optional.of(Month.JANUARY));
} 
```

对于无效的代码值，我们应该得到一个空的`Optional`。让我们也通过测试来验证这一点:

```java
@Test
public void givenMonths_whenInvalidMonthCodeProvided_optionalEmptyIsReturned() {
    Optional<Month> result = Month.findByCode(0);
    assertThat(result).isEmpty();
} 
```

在某些情况下，我们可能需要执行更严格的搜索。因此，我们不能容忍无效的输入，我们会抛出异常来证明这一点。

## 5.搜索方法引发的异常

我们可能想抛出一个异常，而不是返回`null`或空的`Optional`值。抛出哪个异常完全取决于系统的需求。如果找不到枚举，我们将选择抛出一个`IllegalArgumentException`。下面是搜索方法的代码:

```java
public static Month findByValue(String value) {
    return Arrays.stream(values()).filter(month -> month.getValue().equalsIgnoreCase(value)).findFirst().orElseThrow(IllegalArgumentException::new);
}
```

我们可以再次看到，在抛出异常时，我们使用的是 Java 8 风格。让我们用一个测试来验证它:

```java
@Test
public void givenMonths_whenInvalidMonthValueProvided_illegalArgExIsThrown() {
    assertThatIllegalArgumentException().isThrownBy(() -> Month.findByValue("Jan"));
} 
```

本文中演示的搜索方法并不是唯一的方法，但它们代表了最常见的选项。我们还可以调整这些实现来满足我们系统的需求。

## 6.结论

在本文中，我们学习了搜索枚举的各种方法。我们还讨论了返回结果的不同方式。最后，我们用可靠的单元测试来支持这些实现。

和往常一样，与本文相关的代码可以从 GitHub 上的[处获得。](https://web.archive.org/web/20220628061921/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-lang-oop-types-2)