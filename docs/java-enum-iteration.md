# 在 Java 中迭代枚举值

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-enum-iteration>

## 1。概述

在 Java 中，`Enum`是一种数据类型，它帮助我们将一组预定义的常数赋给一个变量。

在这个快速教程中，我们将学习在 Java 中迭代`Enum`的不同方法。

## 2。迭代`Enum`个值

让我们首先定义一个`Enum`，这样我们可以创建一些简单的代码示例:

```java
public enum DaysOfWeekEnum {
    SUNDAY,
    MONDAY,
    TUESDAY, 
    WEDNESDAY, 
    THURSDAY, 
    FRIDAY, 
    SATURDAY
} 
```

没有迭代的方法，像`forEach()`或`iterator()`。相反，我们可以使用由`values()`方法返回的`Enum`值的数组。

### 2.1。使用`for`循环进行迭代

首先，我们可以简单地使用老式的`for`循环:

```java
for (DaysOfWeekEnum day : DaysOfWeekEnum.values()) { 
    System.out.println(day); 
}
```

### 2.2。使用`Stream` 进行迭代

我们还可以使用`java.util.Stream`对`Enum`值执行操作。

要创建一个`Stream,`,我们有两种选择。首先是使用`Stream.of`:

```java
Stream.of(DaysOfWeekEnum.values());
```

第二种是使用`Arrays.stream`:

```java
Arrays.stream(DaysOfWeekEnum.values());
```

让我们扩展`DaysOfWeekEnum`类来创建一个使用`Stream`的例子:

```java
public enum DaysOfWeekEnum {

    SUNDAY("off"), 
    MONDAY("working"), 
    TUESDAY("working"), 
    WEDNESDAY("working"), 
    THURSDAY("working"), 
    FRIDAY("working"), 
    SATURDAY("off");

    private String typeOfDay;

    DaysOfWeekEnum(String typeOfDay) {
        this.typeOfDay = typeOfDay;
    }

    // standard getters and setters 

    public static Stream<DaysOfWeekEnum> stream() {
        return Stream.of(DaysOfWeekEnum.values()); 
    }
}
```

现在我们将编写一个示例来打印非工作日:

```java
public class EnumStreamExample {

    public static void main() {
        DaysOfWeekEnum.stream()
        .filter(d -> d.getTypeOfDay().equals("off"))
        .forEach(System.out::println);
    }
}
```

当我们运行时，我们得到的输出是:

```java
SUNDAY
SATURDAY
```

### 2.3。使用`forEach()` 进行迭代

Java 8 中的`Iterable`接口增加了`forEach()`方法。所以所有的 java 集合类都有一个`forEach()`方法的实现。为了将这些与一个`Enum`一起使用，我们首先需要将 `Enum`转换成一个合适的集合。我们可以使用`Arrays.asList()`生成一个`ArrayList,`,然后我们可以使用`forEach()`方法循环使用它:

```java
Arrays.asList(DaysOfWeekEnum.values())
  .forEach(day -> System.out.println(day)); 
```

### 2.4。使用`EnumSet` 进行迭代

`EnumSet`是一个专门的 set 实现，我们可以将其用于`Enum`类型:

```java
EnumSet.allOf(DaysOfWeekEnum.class)
  .forEach(day -> System.out.println(day));
```

### 2.5。使用`Enum`值的`ArrayList`

我们也可以把一个`Enum`的值加到一个`List`上。这让我们可以像操纵其他任何东西一样操纵`List`:

```java
List<DaysOfWeekEnum> days = new ArrayList<>();
days.add(DaysOfWeekEnum.FRIDAY);
days.add(DaysOfWeekEnum.SATURDAY);
days.add(DaysOfWeekEnum.SUNDAY);
for (DaysOfWeekEnum day : days) {
    System.out.println(day);
}
days.remove(DaysOfWeekEnum.SATURDAY);
if (!days.contains(DaysOfWeekEnum.SATURDAY)) {
    System.out.println("Saturday is no longer in the list");
}
for (DaysOfWeekEnum day : days) {
    System.out.println(day);
} 
```

我们也可以通过使用`Arrays.asList()`来创建一个`ArrayList`。

然而，由于`ArrayList`是由`Enum`值数组支持的，所以它是不可变的，所以我们不能从`List.`中添加或移除项目，下面代码中的移除会因`UnsupportedOperationException`而失败:

```java
List<DaysOfWeekEnum> days = Arrays.asList(DaysOfWeekEnum.values());
days.remove(0); 
```

## 3。结论

在本文中，我们讨论了使用 `forEach,` `Stream`和 Java 中的`for`循环迭代`Enum`的各种方法。

如果我们需要执行任何并行操作，`Stream`是一个很好的选择。否则，对于使用哪种方法没有限制。

和往常一样，这里解释的所有例子的代码都可以在 GitHub 上找到[。](https://web.archive.org/web/20220628095914/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-lang-oop-types)