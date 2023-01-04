# 具有不同值类型的 Java HashMap

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-hashmap-different-value-types>

## 1.概观

一个`[HashMap](/web/20220627165559/https://www.baeldung.com/java-hashmap)`存储键值映射。在本教程中，我们将讨论如何在一个`HashMap`中存储不同类型的值。

## 2.问题简介

自从引入了 [Java 泛型](/web/20220627165559/https://www.baeldung.com/java-generics)，我们通常以一种通用的方式使用`HashMap`——例如:

```java
Map<String, Integer> numberByName = new HashMap<>();
```

在这种情况下，我们只能将`String`和`Integer`数据作为键值对放入映射`numberByName`。这很好，因为它确保了类型安全。例如，如果我们试图将一个`Float`对象放入`Map`，我们将得到“不兼容类型”编译错误。

但是，**有时候，我们会想把不同类型的数据放到一个`Map`** 中。例如，我们希望`numberByName`地图也将`Float`和`BigDecimal`对象存储为值。

在讨论如何实现之前，让我们创建一个示例问题，以便于演示和解释。假设我们有三个不同类型的对象:

```java
Integer intValue = 777;
int[] intArray = new int[]{2, 3, 5, 7, 11, 13};
Instant instant = Instant.now(); 
```

正如我们所看到的，这三种类型是完全不同的。所以首先，我们将尝试把这三个对象放在一个`HashMap`中。为了简单起见，我们将使用`String`值作为键。

当然，在某些时候，我们需要从`Map`中读出数据并使用这些数据。因此，我们将遍历`HashMap`中的条目，并为每个条目打印带有一些描述的值。

那么，让我们来看看如何实现这一目标。

## 3.使用`Map<String, Object>`

我们知道在 Java 中， **`Object`是所有类型**的超类型。因此，如果我们将一个`Map`声明为`Map<String, Object>`，它应该接受任何类型的值。

接下来，让我们看看这种方法是否符合我们的要求。

### 3.1.将数据放入`Map`

正如我们前面提到的，一个`Map<String, Object>`允许我们将任何类型的值放入其中:

```java
Map<String, Object> rawMap = new HashMap<>();
rawMap.put("E1 (Integer)", intValue);
rawMap.put("E2 (IntArray)", intArray);
rawMap.put("E3 (Instant)", instant); 
```

这很简单。接下来，让我们访问`Map`中的条目并打印值和描述。

### 3.2.使用数据

在我们将一个值放入`Map<String, Object>`之后，我们已经丢失了该值的具体类型。因此，**在使用数据**之前，我们需要检查并把值转换成合适的类型。例如，我们可以使用[`instanceof`操作符](/web/20220627165559/https://www.baeldung.com/java-instanceof)来验证值的类型:

```java
rawMap.forEach((k, v) -> {
    if (v instanceof Integer) {
        Integer theV = (Integer) v;
        System.out.println(k + " -> "
          + String.format("The value is a %s integer: %d", theV > 0 ? "positive" : "negative", theV));
    } else if (v instanceof int[]) {
        int[] theV = (int[]) v;
        System.out.println(k + " -> "
          + String.format("The value is an array of %d integers: %s", theV.length, Arrays.toString(theV)));
    } else if (v instanceof Instant) {
        Instant theV = (Instant) v;
        System.out.println(k + " -> "
          + String.format("The value is an instant: %s", FORMATTER.format(theV)));
    } else {
        throw new IllegalStateException("Unknown Type Found.");
    }
});
```

如果我们执行上面的代码，我们将看到输出:

```java
E1 (Integer) -> The value is a positive integer: 777
E2 (IntArray) -> The value is an array of 6 integers: [2, 3, 5, 7, 11, 13]
E3 (Instant) -> The value is an instant: 2021-11-23 21:48:02
```

这种方法如我们所料。

然而，它也有一些缺点。接下来，让我们仔细看看它们。

### 3.3.不足之处

首先，如果我们计划让 map 支持相对更多的不同类型，**多个`if-else`语句将成为一个大的代码块，使代码难以阅读**。

此外，**如果我们想要使用的类型包含继承关系，`instanceof`检查可能会失败**。

例如，如果我们把一个`java.lang.Integer intValue`和一个`[java.lang.Number](https://web.archive.org/web/20220627165559/https://docs.oracle.com/en/java/javase/12/docs/api/java.base/java/lang/Number.html) numberValue` 放在地图中，我们不能使用`instanceof`操作符来区分它们。这是因为`(intValue instanceof Integer)`和`(intValue instanceof Number)`都返回`true`。

因此，我们必须添加额外的检查来确定值的具体类型。但是，当然，这将使代码难以阅读。

最后，由于我们的映射接受任何类型的值，**我们已经失去了类型安全**。也就是说，当遇到意外类型时，我们必须处理异常。

可能会出现一个问题:有没有一种方法可以接受不同类型的数据并保持类型安全？

接下来，我们将提出另一种方法来解决这个问题。

## 4.为所有必需的类型创建超类型

在这一节中，我们将引入一个超类型来保持类型安全。

### 4.1.数据模型

首先，我们创建一个接口`DynamicTypeValue`:

```java
public interface DynamicTypeValue {
    String valueDescription();
}
```

这个接口将是我们期望地图支持的所有类型的超类型。它还可以包含一些常见的操作。例如，我们已经定义了一个方法`valueDescription`。

然后**，我们为每个具体类型创建一个类来包装值并实现我们创建的接口**。例如，我们可以为`Integer`类型创建一个`IntegerTypeValue`类:

```java
public class IntegerTypeValue implements DynamicTypeValue {
    private Integer value;

    public IntegerTypeValue(Integer value) {
        this.value = value;
    }

    @Override
    public String valueDescription() {
        if(value == null){
            return "The value is null.";
        }
        return String.format("The value is a %s integer: %d", value > 0 ? "positive" : "negative", value);
    }
} 
```

类似地，让我们为其他两种类型创建类:

```java
public class IntArrayTypeValue implements DynamicTypeValue {
    private int[] value;

    public IntArrayTypeValue(int[] value) { ... }

    @Override
    public String valueDescription() {
        // null handling omitted
        return String.format("The value is an array of %d integers: %s", value.length, Arrays.toString(value));
    }
}
```

```java
public class InstantTypeValue implements DynamicTypeValue {
    private static DateTimeFormatter FORMATTER = ...

    private Instant value;

    public InstantTypeValue(Instant value) { ... }

    @Override
    public String valueDescription() {
        // null handling omitted
        return String.format("The value is an instant: %s", FORMATTER.format(value));
    }
} 
```

如果我们需要支持更多类型，我们只需添加相应的类。

接下来，让我们看看如何使用上面的数据模型在地图中存储和使用不同类型的值。

### 4.2.将数据放入并使用`Map`

首先，让我们看看如何声明`Map`并将各种类型的数据放入其中:

```java
Map<String, DynamicTypeValue> theMap = new HashMap<>();
theMap.put("E1 (Integer)", new IntegerTypeValue(intValue));
theMap.put("E2 (IntArray)", new IntArrayTypeValue(intArray));
theMap.put("E3 (Instant)", new InstantTypeValue(instant)); 
```

正如我们所看到的，我们已经将地图声明为`Map<String, DynamicTypeValue> `，从而保证了**类型的安全性**:只有`DynamicTypeValue`类型的数据才允许放入地图。

**当我们向地图添加数据时，我们实例化我们已经创建的相应类**。

当我们使用数据时，不需要**型式检查和铸造**:

```java
theMap.forEach((k, v) -> System.out.println(k + " -> " + v.valueDescription())); 
```

如果我们运行代码，它会打印:

```java
E1 (Integer) -> The value is a positive integer: 777
E2 (IntArray) -> The value is an array of 5 integers: [2, 3, 5, 7, 11]
E3 (Instant) -> The value is an instant: 2021-11-23 22:32:43
```

正如我们所看到的，**这种方法的代码是干净的，更容易阅读**。

此外，由于我们为需要支持的每个类型创建了一个包装类，因此具有继承关系的类型不会导致任何问题。

由于类型安全，**我们不需要处理面对意外类型数据的错误情况**。

## 5.结论

在本文中，我们讨论了如何让 Java `HashMap`支持不同类型的值数据。

此外，我们已经通过示例提出了两种实现方法。

和往常一样，本文附带的源代码可以在 GitHub 上的[处获得。](https://web.archive.org/web/20220627165559/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-collections-maps-4)