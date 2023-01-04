# 将值附加到 Java 枚举

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-enum-values>

## 1。概述

Java `enum`类型提供了一种语言支持的方式来创建和使用常量值。通过定义一组有限的值，`enum`比像`String`或`int`这样的常量文字变量更加类型安全。

**然而，`enum`值需要是有效的标识符**，我们被鼓励按照惯例使用 SCREAMING _ SNAKE _ CASE。

考虑到这些限制，**和`enum`值不适合人类可读的字符串或非字符串值。**

在本教程中，我们将使用`enum`特性作为 Java 类来附加我们想要的值。

## 延伸阅读:

## Java 枚举指南

A quick and practical guide to the use of the Java Enum implementation, what it is, what problems it solves and how it can be used to implement commonly used design patterns.[Read more](/web/20221012024725/https://www.baeldung.com/a-guide-to-java-enums) →

## [在 Java 中迭代枚举值](/web/20221012024725/https://www.baeldung.com/java-enum-iteration)

Learn three simple ways to iterate over a Java enum.[Read more](/web/20221012024725/https://www.baeldung.com/java-enum-iteration) →

## Java 构造函数指南

Learn the basics about constructors in Java as well as some advanced tips[Read more](/web/20221012024725/https://www.baeldung.com/java-constructors) →

## 2。使用 Java `Enum`作为类

我们经常创建一个简单的值列表`enum`。例如，元素周期表的前两行是一个简单的`enum`:

```
public enum Element {
    H, HE, LI, BE, B, C, N, O, F, NE
}
```

使用上面的语法，我们已经创建了名为`Element`的`enum`的十个静态最终实例。虽然这非常有效，但我们只捕获了元素符号。虽然大写形式适合 Java 常量，但这不是我们通常写符号的方式。

此外，我们还遗漏了元素周期表的其他属性，比如名称和原子量。

**虽然`enum`类型在 Java 中有特殊的行为，但是我们可以像处理其他类一样添加构造函数、字段和方法。因此，我们可以增强我们的`enum` 来包含我们需要的值。**

## 3。添加构造函数和最终字段

让我们从添加元素名称开始。

**我们将使用构造函数**将这些名字设置到一个`final`变量中:

```
public enum Element {
    H("Hydrogen"),
    HE("Helium"),
    // ...
    NE("Neon");

    public final String label;

    private Element(String label) {
        this.label = label;
    }
}
```

首先，我们注意到声明列表中的特殊语法。这就是为`enum`类型调用构造函数的方式。**虽然对`enum`使用`new` 操作符是非法的，但是我们可以在声明列表中传递构造函数参数。**

然后我们声明一个实例变量`label`。关于这一点，有几点需要注意。

**首先，我们选择了`label`标识符，而不是`name`。尽管成员字段`name`是可用的，我们还是选择`label`以避免与预定义的`Enum.name()`方法混淆。**

二、**我们的`label`场是`final`。虽然`enum`的字段不一定是`final`，但在大多数情况下，我们不希望我们的标签改变。本着`enum` 价值观不变的精神，这是有道理的。**

最后，`label`字段是公共的，所以我们可以直接访问标签:

```
System.out.println(BE.label);
```

另一方面，该字段可以是用`getLabel()`方法访问的`private`。为了简洁起见，本文将继续使用公共字段样式。

## 4。定位 Java `Enum`值

Java 为所有的`enum`类型提供了一个`valueOf(String)`方法。

因此，我们总是可以基于声明的名称获得一个`enum`值:

```
assertSame(Element.LI, Element.valueOf("LI"));
```

然而，我们可能也想通过我们的标签字段来查找一个`enum`值。

为此，我们可以添加一个`static`方法:

```
public static Element valueOfLabel(String label) {
    for (Element e : values()) {
        if (e.label.equals(label)) {
            return e;
        }
    }
    return null;
}
```

静态`valueOfLabel()`方法迭代`Element`值，直到找到匹配。如果没有找到匹配，它返回`null`。相反，可以抛出一个异常，而不是返回`null`。

让我们看一个使用我们的`valueOfLabel()`方法的简单例子:

```
assertSame(Element.LI, Element.valueOfLabel("Lithium"));
```

## 5。缓存查找值

**我们可以通过使用`Map`缓存标签来避免迭代`enum`值。**

为此，我们定义了一个`static final Map`，并在类加载时填充它:

```
public enum Element {

    // ... enum values

    private static final Map<String, Element> BY_LABEL = new HashMap<>();

    static {
        for (Element e: values()) {
            BY_LABEL.put(e.label, e);
        }
    }

   // ... fields, constructor, methods

    public static Element valueOfLabel(String label) {
        return BY_LABEL.get(label);
    }
}
```

**由于被缓存，`enum`值只迭代一次**，简化了`valueOfLabel()`方法。

作为一种替代方法，我们可以在第一次使用`valueOfLabel()`方法访问缓存时延迟构造缓存。在这种情况下，映射访问必须同步，以防止并发问题。

## 6。附加多个值

**`Enum`构造函数可以接受多个值。**

为了说明，让我们添加原子序数作为一个`int`和原子量作为一个`float`:

```
public enum Element {
    H("Hydrogen", 1, 1.008f),
    HE("Helium", 2, 4.0026f),
    // ...
    NE("Neon", 10, 20.180f);

    private static final Map<String, Element> BY_LABEL = new HashMap<>();
    private static final Map<Integer, Element> BY_ATOMIC_NUMBER = new HashMap<>();
    private static final Map<Float, Element> BY_ATOMIC_WEIGHT = new HashMap<>();

    static {
        for (Element e : values()) {
            BY_LABEL.put(e.label, e);
            BY_ATOMIC_NUMBER.put(e.atomicNumber, e);
            BY_ATOMIC_WEIGHT.put(e.atomicWeight, e);
        }
    }

    public final String label;
    public final int atomicNumber;
    public final float atomicWeight;

    private Element(String label, int atomicNumber, float atomicWeight) {
        this.label = label;
        this.atomicNumber = atomicNumber;
        this.atomicWeight = atomicWeight;
    }

    public static Element valueOfLabel(String label) {
        return BY_LABEL.get(label);
    }

    public static Element valueOfAtomicNumber(int number) {
        return BY_ATOMIC_NUMBER.get(number);
    }

    public static Element valueOfAtomicWeight(float weight) {
        return BY_ATOMIC_WEIGHT.get(weight);
    }
}
```

类似地，我们可以在`enum`中添加任何我们想要的值，例如适当的大小写符号，“he”、“Li”和“Be”。

此外，我们可以通过添加方法来执行操作，从而将计算值添加到我们的`enum`中。

## 7 .**。控制界面**

由于向我们的`enum`添加了字段和方法，我们改变了它的公共接口。因此，我们使用核心`Enum` `name()`和`valueOf()`方法的代码将不会知道我们的新字段。

**Java 语言已经为我们定义了`static` `valueOf()`方法，所以我们不能提供自己的`valueOf()`实现。**

同样，**因为`Enum.name()`方法是`final`，我们也不能覆盖它。**

因此，使用标准的`Enum` API 无法利用我们的额外字段。相反，让我们看看一些不同的方法来暴露我们的领域。

### 7.1。`toString()`压倒一切

超越`toString()`可能是超越`name()`的替代方案:

```
@Override 
public String toString() { 
    return this.label; 
}
```

默认情况下，`Enum.toString()`返回与`Enum.name().`相同的值

### 7.2。实现一个接口

**Java 中的`enum`类型可以实现接口。**虽然这种方法不像`Enum` API 那样通用，但接口确实有助于我们进行通用化。

让我们考虑这个接口:

```
public interface Labeled {
    String label();
}
```

为了与`Enum.name()`方法保持一致，我们的`label()`方法没有`get`前缀。

因为`valueOfLabel()` 方法是`static`，所以我们不把它包含在我们的接口中。

最后，我们可以在我们的`enum`中实现接口:

```
public enum Element implements Labeled {

    // ...

    @Override
    public String label() {
        return label;
    }

    // ...
}
```

这种方法的一个好处是`Labeled`接口可以应用于任何类，而不仅仅是`enum`类型。我们现在有了一个更加上下文相关的 API，而不是依赖于通用的`Enum` API。

## 8。结论

在本文中，我们探索了 Java `Enum`实现的许多特性。通过添加构造函数、字段和方法，我们看到`enum`能做的远不止文字常量。

和往常一样，本文的完整源代码可以在 [GitHub](https://web.archive.org/web/20221012024725/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-lang-oop-types) 上找到。