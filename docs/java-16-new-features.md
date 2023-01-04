# Java 16 的新特性

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-16-new-features>

## 1.概观

于 2021 年 3 月 16 日发布的 Java 16 ，是基于 [Java 15](/web/20220824083848/https://www.baeldung.com/java-15-new) 的最新短期增量版本。这个版本附带了一些有趣的特性，比如记录和密封类。

在本文中，我们将探索其中的一些新特性。

## 2.从代理实例调用默认方法(JDK-8159746)

作为对接口中默认方法的增强，随着 Java 16 的发布，已经添加了对使用反射通过[动态代理](/web/20220824083848/https://www.baeldung.com/java-dynamic-proxies)调用接口的默认方法的支持。

为了说明这一点，让我们看一个简单的默认方法示例:

```java
interface HelloWorld {
    default String hello() {
        return "world";
    }
}
```

有了这个增强，我们可以使用反射调用该接口的代理上的默认方法:

```java
Object proxy = Proxy.newProxyInstance(getSystemClassLoader(), new Class<?>[] { HelloWorld.class },
    (prox, method, args) -> {
        if (method.isDefault()) {
            return InvocationHandler.invokeDefault(prox, method, args);
        }
        // ...
    }
);
Method method = proxy.getClass().getMethod("hello");
assertThat(method.invoke(proxy)).isEqualTo("world");
```

## 3.日间支持(JDK-8247781)

DateTimeFormatter 新增的[是一天中的时段符号“`B`”，它提供了 am/pm 格式的另一种选择:](https://web.archive.org/web/20220824083848/https://bugs.openjdk.java.net/browse/JDK-8247781)

```java
LocalTime date = LocalTime.parse("15:25:08.690791");
DateTimeFormatter formatter = DateTimeFormatter.ofPattern("h B");
assertThat(date.format(formatter)).isEqualTo("3 in the afternoon");
```

我们得到的不是类似于“`3pm`”的输出，而是“`3 in the afternoon`”。我们也可以用“`B`”、“`BBBB`”或“`BBBBB`”、[、`DateTimeFormatter`、](/web/20220824083848/https://www.baeldung.com/java-datetimeformatter)图案分别表示短、满、窄款式。

## 4.添加`Stream.toList`方法(JDK-8180352)

目的是用一些常用的`Stream`收集器减少样板文件，比如`Collectors.toList`和`Collectors.toSet`:

```java
List<String> integersAsString = Arrays.asList("1", "2", "3");
List<Integer> ints = integersAsString.stream().map(Integer::parseInt).collect(Collectors.toList());
List<Integer> intsEquivalent = integersAsString.stream().map(Integer::parseInt).toList();
```

我们的`ints`示例以旧的方式工作，但是`intsEquivalent`具有相同的结果，并且更加简洁。

## 5.载体原料药培养箱(JEP-338)

Vector API 正处于 Java 16 的初始培育阶段。这个 API 的想法是提供一种矢量计算的方法，最终能够比传统的标量计算方法执行得更好(在支持 CPU 架构上)。

让我们看看传统上我们如何将两个数组相乘:

```java
int[] a = {1, 2, 3, 4};
int[] b = {5, 6, 7, 8};

var c = new int[a.length];

for (int i = 0; i < a.length; i++) {
    c[i] = a[i] * b[i];
}
```

对于长度为 4 的数组，标量计算的这个例子将在 4 个周期内执行。现在，让我们看看等效的基于矢量的计算:

```java
int[] a = {1, 2, 3, 4};
int[] b = {5, 6, 7, 8};

var vectorA = IntVector.fromArray(IntVector.SPECIES_128, a, 0);
var vectorB = IntVector.fromArray(IntVector.SPECIES_128, b, 0);
var vectorC = vectorA.mul(vectorB);
vectorC.intoArray(c, 0);
```

我们在基于向量的代码中做的第一件事是使用这个类`fromArray.` 的静态工厂方法从我们的输入数组中创建两个`IntVectors `。第一个参数是向量的大小，接下来是数组和偏移量(这里设置为 0)。这里最重要的是向量的大小，我们得到了 128 位。在 Java 中，每个`int`需要 4 个字节来保存。

因为我们有一个 4 `ints,`的输入数组，所以需要 128 位来存储。我们的单个`Vector`可以存储整个数组。

在某些架构上，编译器能够优化字节码，将计算周期从 4 个减少到 1 个。这些优化有利于机器学习和密码学等领域。

我们应该注意，处于酝酿阶段意味着这个 Vector API 会随着新版本的发布而发生变化。

## 6.记录(JEP-395)

[记录](/web/20220824083848/https://www.baeldung.com/java-record-keyword)是 Java 14 中引入的。Java 16 带来了一些[增量变化](https://web.archive.org/web/20220824083848/https://bugs.openjdk.java.net/browse/JDK-8262133)。

记录类似于`enum` s，事实上它们是类的受限形式。定义一个`record`是定义一个不可变数据保存对象的简洁方法。

### 6.1.没有记录的示例

首先，让我们定义一个`Book`类:

```java
public final class Book {
    private final String title;
    private final String author;
    private final String isbn;

    public Book(String title, String author, String isbn) {
        this.title = title;
        this.author = author;
        this.isbn = isbn;
    }

    public String getTitle() {
        return title;
    }

    public String getAuthor() {
        return author;
    }

    public String getIsbn() {
        return isbn;
    }

    @Override
    public boolean equals(Object o) {
        // ...
    }

    @Override
    public int hashCode() {
        return Objects.hash(title, author, isbn);
    }
}
```

用 Java 创建简单的数据保存类需要大量样板代码。这可能很麻烦，并导致开发人员没有提供所有必要方法的错误，例如`equals`和`hashCode`。

类似地，有时开发人员会跳过创建合适的[不可变类](/web/20220824083848/https://www.baeldung.com/java-immutable-object)的必要步骤。有时我们最终会重用一个通用类，而不是为每个不同的用例定义一个专用类。

大多数现代的 ide 都提供了自动生成代码的能力(比如 setters、getters、constructors 等等)。)有助于缓解这些问题，并减少开发人员编写代码的开销。然而，记录提供了一个内置的机制来减少样板代码并创建相同的结果。

### 6.2.记录示例

这里的`Book`改写为`Record`:

```java
public record Book(String title, String author, String isbn) {
}
```

通过使用`record` 关键字，我们将`Book`类减少到两行。这使得它更容易，更不容易出错。

### 6.3.Java 16 中记录的新增内容

随着 Java 16 的发布，我们现在可以将记录定义为内部类的类成员。这是由于在 [JEP-384](https://web.archive.org/web/20220824083848/https://openjdk.java.net/jeps/384) 下 Java 15 的增量发布中遗漏了放宽限制:

```java
class OuterClass {
    class InnerClass {
        Book book = new Book("Title", "author", "isbn");
    }
}
```

## 7.`instanceof` (JEP-394)的模式匹配

从 Java 16 开始，`instanceof`关键字的[模式匹配已被添加。](https://web.archive.org/web/20220824083848/https://bugs.openjdk.java.net/browse/JDK-8250623)

以前，我们可能会编写这样的代码:

```java
Object obj = "TEST";

if (obj instanceof String) {
    String t = (String) obj;
    // do some logic...
}
```

这些代码必须首先检查`obj`的实例，然后将对象转换为`String` ，并将其赋给一个新的变量 `t.`，而不是仅仅关注应用程序所需的逻辑

随着模式匹配的引入，我们可以重写这段代码:

```java
Object obj = "TEST";

if (obj instanceof String t) {
    // do some logic
}
```

我们现在可以声明一个变量——在本例中是作为`instanceof`检查的一部分的`t –` 。

## 8.密封类(JEP-397)

Java 15 中首次引入的密封类[，提供了一种机制来确定哪些子类被允许扩展或实现父类或接口。](https://web.archive.org/web/20220824083848/https://openjdk.java.net/jeps/360)

### 8.1.例子

让我们通过定义一个接口和两个实现类来说明这一点:

```java
public sealed interface JungleAnimal permits Monkey, Snake  {
}

public final class Monkey implements JungleAnimal {
}

public non-sealed class Snake implements JungleAnimal {
}
```

`sealed`关键字与`permits`关键字一起使用，以确定哪些类被允许实现这个接口。在我们的例子中，这是`Monkey`和`Snake. `

密封类的所有继承类都必须标记有以下内容之一:

*   `sealed`–意味着他们必须使用`permits`关键字定义允许从它继承什么类。
*   `final`–防止任何进一步的子类
*   允许任何类都可以继承它。

密封类的一个显著优点是，它们允许进行详尽的模式匹配检查，而不需要捕捉所有未覆盖的情况。例如，使用我们定义的类，我们可以有逻辑来覆盖`JungleAnimal`的所有可能的子类:

```java
JungleAnimal j = // some JungleAnimal instance

if (j instanceof Monkey m) {
    // do logic
} else if (j instanceof Snake s) {
    // do logic
}
```

我们不需要一个`else`块，因为密封类只允许两种可能的子类型`Monkey`和`Snake`。

### 8.2.Java 16 中密封类的新增内容

Java 16 中的密封类增加了一些内容。这些是 Java 16 引入到密封类中的变化:

*   Java 语言将`sealed`、`non-sealed`和`permits`识别为上下文关键字(类似于`abstract`和`extends`)
*   限制创建作为密封类的子类的本地类的能力(类似于不能创建密封类的匿名类)。
*   在转换密封类和从密封类派生的类时进行更严格的检查

## 9.其他变化

继续 Java 15 版本中的 JEP-383， [外来链接器 API](https://web.archive.org/web/20220824083848/https://bugs.openjdk.java.net/browse/JDK-8249755) 提供了一种灵活的方式来访问主机上的本地代码。最初是为了 C 语言的互操作性，将来可能会适应 C++或 Fortran 等其他语言。这个特性的目标是最终取代 [Java 原生接口](https://web.archive.org/web/20220824083848/https://docs.oracle.com/en/java/javase/11/docs/specs/jni/index.html) 。

另一个重要的变化是 JDK 内部现在默认被强封装。这些从 Java 9 开始就可以访问了。然而，现在 JVM 需要参数`–illegal-access=permit`。这将影响所有的库和应用程序(特别是在测试的时候),这些库和应用程序目前正在直接使用 JDK 内部，并且简单地忽略警告信息。

## 10.结论

在本文中，我们介绍了作为增量 Java 16 版本的一部分引入的一些特性和变化。Java 16 的完整变更列表在 [JDK 发布说明](https://web.archive.org/web/20220824083848/https://jdk.java.net/16/release-notes)中。

一如既往，这篇文章中的所有代码都可以在 GitHub 上找到[。](https://web.archive.org/web/20220824083848/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-16)