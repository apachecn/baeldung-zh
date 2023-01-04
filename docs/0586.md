# 在 Java 中扩展枚举

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-extending-enums>

## 1.概观

Java 5 中引入的 [enum](/web/20221208143917/https://www.baeldung.com/a-guide-to-java-enums) 类型是一种特殊的数据类型，表示一组常数。

使用枚举，我们可以以类型安全的方式定义和使用常量。它为常数带来了编译时检查。此外，它允许我们在`switch-case`语句中使用常量。

在本教程中，我们将讨论在 Java 中扩展枚举，包括添加新的常量值和新的功能。

## 2.枚举和继承

当我们想要扩展一个 Java 类时，我们通常会创建一个子类。在 Java 中，枚举也是类。

在这一节中，我们将看看是否可以继承一个 enum，就像我们对普通 Java 类所做的那样。

### 2.1.扩展枚举类型

首先，我们来看一个例子，这样我们可以很快理解问题:

```
public enum BasicStringOperation {
    TRIM("Removing leading and trailing spaces."),
    TO_UPPER("Changing all characters into upper case."),
    REVERSE("Reversing the given string.");

    private String description;

    // constructor and getter
}
```

如上面的代码所示，我们有一个包含三个基本字符串操作的 enum，`BasicStringOperation,`。

现在，假设我们想给 enum 添加一些扩展，比如`MD5_ENCODE`和`BASE64_ENCODE`。我们可能会想到这个简单的解决方案:

```
public enum ExtendedStringOperation extends BasicStringOperation {
    MD5_ENCODE("Encoding the given string using the MD5 algorithm."),
    BASE64_ENCODE("Encoding the given string using the BASE64 algorithm.");

    private String description;

    // constructor and getter
}
```

然而，当我们试图编译这个类时，我们会看到编译器错误:

```
Cannot inherit from enum BasicStringOperation
```

### 2.2.枚举不允许继承

让我们弄清楚为什么我们会收到一个编译器错误。

当我们编译一个 enum 时，Java 编译器会给它施一些魔法:

*   **它将 enum 变成抽象类`java.lang.Enum`** 的子类
*   **它将枚举编译成一个`final`类**

例如，如果我们使用 [`javap`](/web/20221208143917/https://www.baeldung.com/java-class-view-bytecode#javap) 反汇编我们编译的`BasicStringOperation`枚举，我们会看到它被表示为`java.lang.Enum<BasicStringOperation>`的子类:

```
$ javap BasicStringOperation  
public final class com.baeldung.enums.extendenum.BasicStringOperation 
    extends java.lang.Enum<com.baeldung.enums.extendenum.BasicStringOperation> {
  public static final com.baeldung.enums.extendenum.BasicStringOperation TRIM;
  public static final com.baeldung.enums.extendenum.BasicStringOperation TO_UPPER;
  public static final com.baeldung.enums.extendenum.BasicStringOperation REVERSE;
 ...
} 
```

我们知道，我们不能在 Java 中继承一个`final`类。此外，即使我们可以创建`ExtendedStringOperation`枚举来继承`BasicStringOperation`，我们的`ExtendedStringOperation`枚举也将扩展两个类:`BasicStringOperation`和`java.lang.Enum.` ，也就是说，这将成为多重继承的情况，这在 Java 中是不支持的。

## 3.用接口模拟可扩展枚举

我们已经知道，我们不能创建现有枚举的子类。然而，接口是可扩展的。因此，**我们可以通过实现一个接口**来模拟可扩展的枚举。

### 3.1.模拟扩展常数

为了快速理解这种技术，让我们来看看如何模拟扩展我们的`BasicStringOperation`枚举来拥有`MD5_ENCODE`和`BASE64_ENCODE`操作。

首先，我们将创建一个接口`,` `StringOperation`:

```
public interface StringOperation {
    String getDescription();
} 
```

接下来，我们将让两个枚举实现上面的接口:

```
public enum BasicStringOperation implements StringOperation {
    TRIM("Removing leading and trailing spaces."),
    TO_UPPER("Changing all characters into upper case."),
    REVERSE("Reversing the given string.");

    private String description;
    // constructor and getter override
}

public enum ExtendedStringOperation implements StringOperation {
    MD5_ENCODE("Encoding the given string using the MD5 algorithm."),
    BASE64_ENCODE("Encoding the given string using the BASE64 algorithm.");

    private String description;

    // constructor and getter override
} 
```

最后，我们将看到如何模拟一个可扩展的`BasicStringOperation`枚举。

假设我们的应用程序中有一个方法来获取对`BasicStringOperation`枚举的描述:

```
public class Application {
    public String getOperationDescription(BasicStringOperation stringOperation) {
        return stringOperation.getDescription();
    }
} 
```

现在，我们可以将参数类型`BasicStringOperation,` 更改为接口类型`StringOperation,`，以使该方法接受来自两个枚举的实例:

```
public String getOperationDescription(StringOperation stringOperation) {
    return stringOperation.getDescription();
}
```

### 3.2.扩展功能

我们已经演示了如何用接口来模拟枚举的扩展常量。我们还可以向接口添加方法来扩展枚举的功能。

例如，假设我们想要扩展我们的`StringOperation`枚举，这样每个常量实际上都可以对给定的字符串应用操作:

```
public class Application {
    public String applyOperation(StringOperation operation, String input) {
        return operation.apply(input);
    }
    //...
} 
```

为了实现这一点，我们将向接口添加`apply()`方法:

```
public interface StringOperation {
    String getDescription();
    String apply(String input);
} 
```

接下来，我们让每个`StringOperation`枚举实现这个方法:

```
public enum BasicStringOperation implements StringOperation {
    TRIM("Removing leading and trailing spaces.") {
        @Override
        public String apply(String input) { 
            return input.trim(); 
        }
    },
    TO_UPPER("Changing all characters into upper case.") {
        @Override
        public String apply(String input) {
            return input.toUpperCase();
        }
    },
    REVERSE("Reversing the given string.") {
        @Override
        public String apply(String input) {
            return new StringBuilder(input).reverse().toString();
        }
    };

    //...
}

public enum ExtendedStringOperation implements StringOperation {
    MD5_ENCODE("Encoding the given string using the MD5 algorithm.") {
        @Override
        public String apply(String input) {
            return DigestUtils.md5Hex(input);
        }
    },
    BASE64_ENCODE("Encoding the given string using the BASE64 algorithm.") {
        @Override
        public String apply(String input) {
            return new String(new Base64().encode(input.getBytes()));
        }
    };

    //...
} 
```

一种测试方法证明了这种方法的预期效果:

```
@Test
public void givenAStringAndOperation_whenApplyOperation_thenGetExpectedResult() {
    String input = " hello";
    String expectedToUpper = " HELLO";
    String expectedReverse = "olleh ";
    String expectedTrim = "hello";
    String expectedBase64 = "IGhlbGxv";
    String expectedMd5 = "292a5af68d31c10e31ad449bd8f51263";
    assertEquals(expectedTrim, app.applyOperation(BasicStringOperation.TRIM, input));
    assertEquals(expectedToUpper, app.applyOperation(BasicStringOperation.TO_UPPER, input));
    assertEquals(expectedReverse, app.applyOperation(BasicStringOperation.REVERSE, input));
    assertEquals(expectedBase64, app.applyOperation(ExtendedStringOperation.BASE64_ENCODE, input));
    assertEquals(expectedMd5, app.applyOperation(ExtendedStringOperation.MD5_ENCODE, input));
} 
```

## 4.在不更改代码的情况下扩展枚举

我们已经学习了如何通过实现接口来扩展一个枚举，但是有时候，我们想要扩展一个枚举的功能而不修改它。例如，我们可能希望从第三方库中扩展一个枚举。

### 4.1.关联枚举常量和接口实现

首先，我们来看一个枚举示例:

```
public enum ImmutableOperation {
    REMOVE_WHITESPACES, TO_LOWER, INVERT_CASE
} 
```

假设枚举来自外部库，因此，我们不能更改代码。

现在，在我们的`Application`类中，我们希望有一个方法将给定的操作应用于输入字符串:

```
public String applyImmutableOperation(ImmutableOperation operation, String input) {...}
```

因为我们不能改变枚举代码，**我们可以使用`[EnumMap](/web/20221208143917/https://www.baeldung.com/java-enum-map)`来关联枚举常量和所需的实现**。

首先，我们将创建一个接口:

```
public interface Operator {
    String apply(String input);
} 
```

然后我们将使用一个`EnumMap<ImmutableOperation, Operator>`创建枚举常量和`Operator`实现之间的映射:

```
public class Application {
    private static final Map<ImmutableOperation, Operator> OPERATION_MAP;

    static {
        OPERATION_MAP = new EnumMap<>(ImmutableOperation.class);
        OPERATION_MAP.put(ImmutableOperation.TO_LOWER, String::toLowerCase);
        OPERATION_MAP.put(ImmutableOperation.INVERT_CASE, StringUtils::swapCase);
        OPERATION_MAP.put(ImmutableOperation.REMOVE_WHITESPACES, input -> input.replaceAll("\\s", ""));
    }

    public String applyImmutableOperation(ImmutableOperation operation, String input) {
        return operationMap.get(operation).apply(input);
    }
```

这样，我们的`applyImmutableOperation()`方法可以对给定的输入字符串应用相应的操作:

```
@Test
public void givenAStringAndImmutableOperation_whenApplyOperation_thenGetExpectedResult() {
    String input = " He ll O ";
    String expectedToLower = " he ll o ";
    String expectedRmWhitespace = "HellO";
    String expectedInvertCase = " hE LL o ";
    assertEquals(expectedToLower, app.applyImmutableOperation(ImmutableOperation.TO_LOWER, input));
    assertEquals(expectedRmWhitespace, app.applyImmutableOperation(ImmutableOperation.REMOVE_WHITESPACES, input));
    assertEquals(expectedInvertCase, app.applyImmutableOperation(ImmutableOperation.INVERT_CASE, input));
} 
```

### 4.2.正在验证`EnumMap `对象

如果枚举来自外部库，我们就不知道它是否被更改过，比如向枚举添加新的常量。因此，如果我们不改变`EnumMap`的初始化来包含新的 enum 值，如果新添加的 enum 常量被传递给我们的应用程序，我们的`EnumMap`方法可能会遇到问题。

为了避免这种情况，我们可以在初始化之后验证`EnumMap`,检查它是否包含所有的枚举常量:

```
static {
    OPERATION_MAP = new EnumMap<>(ImmutableOperation.class);
    OPERATION_MAP.put(ImmutableOperation.TO_LOWER, String::toLowerCase);
    OPERATION_MAP.put(ImmutableOperation.INVERT_CASE, StringUtils::swapCase);
    // ImmutableOperation.REMOVE_WHITESPACES is not mapped

    if (Arrays.stream(ImmutableOperation.values()).anyMatch(it -> !OPERATION_MAP.containsKey(it))) {
        throw new IllegalStateException("Unmapped enum constant found!");
    }
} 
```

如上面的代码所示，如果来自`ImmutableOperation `的任何常量没有被映射，就会抛出一个`IllegalStateException`。由于我们的验证是在`static`块中，`IllegalStateException`将是`ExceptionInInitializerError`的原因:

```
@Test
public void givenUnmappedImmutableOperationValue_whenAppStarts_thenGetException() {
    Throwable throwable = assertThrows(ExceptionInInitializerError.class, () -> {
        ApplicationWithEx appEx = new ApplicationWithEx();
    });
    assertTrue(throwable.getCause() instanceof IllegalStateException);
} 
```

因此，一旦应用程序由于提到的错误和原因而无法启动，我们应该再次检查`ImmutableOperation`以确保所有的常量都被映射。

## 5.结论

枚举是 Java 中的一种特殊数据类型。在本文中，我们讨论了为什么枚举不支持继承。然后我们讨论了如何用接口模拟可扩展枚举。我们还学习了如何在不改变 enum 的情况下扩展它的功能。

和往常一样，GitHub 上的[提供了这篇文章的完整源代码。](https://web.archive.org/web/20221208143917/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-lang-oop-types)