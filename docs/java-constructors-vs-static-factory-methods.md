# Java 构造函数与静态工厂方法

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-constructors-vs-static-factory-methods>

## 1。概述

Java 构造函数是获取完全初始化的类实例的默认机制。毕竟，它们提供了注入依赖项所需的所有基础设施，无论是手动还是自动的。

即便如此，在一些特定的用例中，最好求助于静态工厂方法来获得相同的结果。

在本教程中，我们将强调使用静态工厂方法和普通的 Java 构造函数的优缺点。

## 2。静态工厂方法相对于构造函数的优势

在像 Java 这样的面向对象语言中，构造函数会有什么问题呢？总的来说，没什么。即便如此，著名的 [Joshua Block 的有效 Java Item 1](https://web.archive.org/web/20221208143921/https://www.pearson.com/us/higher-education/program/Bloch-Effective-Java-3rd-Edition/PGM1763855.html) 明确指出:

> `“Consider static factory methods instead of constructors”`

虽然这不是灵丹妙药，但以下是支持这种方法的最有说服力的理由:

1.  构造函数没有有意义的名字，所以它们总是被限制在语言强加的标准命名约定中。**静态工厂方法可以有有意义的名字**，因此显式地传达它们做什么
2.  静态工厂方法可以返回实现方法的相同类型、子类型以及原语，因此它们提供了更灵活的返回类型范围
3.  静态工厂方法可以封装预先构造完全初始化的实例所需的所有逻辑，因此它们可以用于将这些额外的逻辑移出构造函数。这防止构造函数[执行进一步的任务，而不仅仅是初始化字段](https://web.archive.org/web/20221208143921/https://stackoverflow.com/questions/23623516/flaw-constructor-does-real-work)
4.  静态工厂方法可以是受控实例化方法，单例模式是这一特性最显著的例子

## 3。JDK 的静态工厂方法

JDK 中有很多静态工厂方法的例子，展示了上面概述的许多优点。让我们探索其中的一些。

### 3.1。`String`班

由于众所周知的[`String`](/web/20221208143921/https://www.baeldung.com/java-string-pool)，我们不太可能使用 [`String`](https://web.archive.org/web/20221208143921/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/String.html) 类构造函数来创建一个新的`String`对象。即便如此，这也是完全合法的:

```java
String value = new String("Baeldung");
```

在这种情况下，构造函数将创建一个新的`String`对象，这是预期的行为。

或者，如果我们想要**使用静态工厂方法**创建一个新的`String`对象，我们可以使用下面的一些`valueOf()`方法的实现:

```java
String value1 = String.valueOf(1);
String value2 = String.valueOf(1.0L);
String value3 = String.valueOf(true);
String value4 = String.valueOf('a'); 
```

`valueOf()`有几个重载的实现。根据传递给方法的参数类型(例如，`int`、`long`、`boolean`、`char,`等等)，每个函数都将返回一个新的`String`对象。

这个名字非常清楚地表达了这个方法的作用。它还坚持 Java 生态系统中为静态工厂方法命名的公认标准。

### 3.2。`Optional`班

JDK 中静态工厂方法的另一个典型例子是 [`Optional`](https://web.archive.org/web/20221208143921/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/Optional.html) 类。这个类**实现了一些具有相当有意义的名字**的工厂方法，包括`[empty()](https://web.archive.org/web/20221208143921/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/Optional.html#empty())`、`[of()](https://web.archive.org/web/20221208143921/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/Optional.html#of(T))`和`[ofNullable()](https://web.archive.org/web/20221208143921/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/Optional.html#ofNullable(T))`:

```java
Optional<String> value1 = Optional.empty();
Optional<String> value2 = Optional.of("Baeldung");
Optional<String> value3 = Optional.ofNullable(null);
```

### 3.3。`Collections`班

很可能**JDK 中静态工厂方法最有代表性的例子是 [`Collections`](https://web.archive.org/web/20221208143921/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/Collections.html) 类。**这是一个不可实例化的类，只实现静态方法。

其中许多是工厂方法，在对提供的集合应用某种类型的算法后，它们也返回集合。

以下是该类的工厂方法的一些典型示例:

```java
Collection syncedCollection = Collections.synchronizedCollection(originalCollection);
Set syncedSet = Collections.synchronizedSet(new HashSet());
List<Integer> unmodifiableList = Collections.unmodifiableList(originalList);
Map<String, Integer> unmodifiableMap = Collections.unmodifiableMap(originalMap); 
```

JDK 中静态工厂方法的数量非常多，因此为了简洁起见，我们将保持示例列表简短。

尽管如此，上面的例子应该给我们一个清晰的概念，静态工厂方法在 Java 中是如何无处不在的。

## 4。自定义静态工厂方法

当然，我们可以实现自己的静态工厂方法。但是什么时候真的值得这样做，而不是通过简单的构造函数创建类实例呢？

让我们看一个简单的例子。

让我们考虑一下这个天真的`User`类:

```java
public class User {

    private final String name;
    private final String email;
    private final String country;

    public User(String name, String email, String country) {
        this.name = name;
        this.email = email;
        this.country = country;
    }

    // standard getters / toString
}
```

在这种情况下，没有明显的警告表明静态工厂方法可能比标准构造函数更好。

如果我们希望所有的`User`实例为`country`字段获得一个默认值，该怎么办？

如果我们用默认值初始化字段，我们也必须重构构造函数，因此使设计更加严格。

我们可以使用静态工厂方法来代替:

```java
public static User createWithDefaultCountry(String name, String email) {
    return new User(name, email, "Argentina");
}
```

下面是我们如何获得一个将默认值分配给`country`字段的`User`实例:

```java
User user = User.createWithDefaultCountry("John", "[[email protected]](/web/20221208143921/https://www.baeldung.com/cdn-cgi/l/email-protection)");
```

## 5。将逻辑移出构造函数

如果我们决定实现需要向构造函数添加更多逻辑的特性，我们的`User`类可能会很快腐烂成一个有缺陷的设计(此时应该敲响警钟了)。

假设我们想给这个类提供记录每个`User`对象创建时间的能力。

**如果我们只是将这种逻辑放入构造函数中，我们就会违反[单一责任原则](https://web.archive.org/web/20221208143921/https://en.wikipedia.org/wiki/Single_responsibility_principle)** 。我们最终会得到一个不仅仅是初始化字段的整体构造函数。

我们可以用静态工厂方法来保持我们的设计整洁:

```java
public class User {

    private static final Logger LOGGER = Logger.getLogger(User.class.getName());
    private final String name;
    private final String email;
    private final String country;

    // standard constructors / getters

    public static User createWithLoggedInstantiationTime(
      String name, String email, String country) {
        LOGGER.log(Level.INFO, "Creating User instance at : {0}", LocalTime.now());
        return new User(name, email, country);
    }
} 
```

下面是我们如何创建改进的`User`实例:

```java
User user 
  = User.createWithLoggedInstantiationTime("John", "[[email protected]](/web/20221208143921/https://www.baeldung.com/cdn-cgi/l/email-protection)", "Argentina");
```

## 6。实例控制的实例化

如上所示，在返回完全初始化的`User`对象之前，我们可以将逻辑块封装到静态工厂方法中。我们可以做到这一点，而不会让构造函数承担执行多个不相关任务的责任。

例如，**假设我们想让我们的`User`类成为单例类。我们可以通过实现一个实例控制的静态工厂方法来实现这一点:**

```java
public class User {

    private static volatile User instance = null;

    // other fields / standard constructors / getters

    public static User getSingletonInstance(String name, String email, String country) {
        if (instance == null) {
            synchronized (User.class) {
                if (instance == null) {
                    instance = new User(name, email, country);
                }
            }
        }
        return instance;
    }
} 
```

由于同步块，方法`getSingletonInstance()`的实现是**线程安全的，性能损失很小。**

在这种情况下，我们使用惰性初始化来演示实例控制的静态工厂方法的实现。

然而，值得一提的是，**实现 Singleton 的最佳方式是使用 Java `enum`类型，因为它既是序列化安全的，又是线程安全的**。关于如何使用不同的方法实现单例的完整细节，请查看[这篇文章](/web/20221208143921/https://www.baeldung.com/java-singleton)。

正如所料，用这个方法获得一个`User`对象看起来与前面的例子非常相似:

```java
User user = User.getSingletonInstance("John", "[[email protected]](/web/20221208143921/https://www.baeldung.com/cdn-cgi/l/email-protection)", "Argentina");
```

## 7。结论

在本文中，我们探索了一些用例，在这些用例中，静态工厂方法是使用普通 Java 构造函数的更好的替代方法。

此外，这种重构模式如此紧密地植根于典型的工作流，以至于大多数 ide 会为我们做这件事。

当然， [Apache NetBeans](https://web.archive.org/web/20221208143921/https://netbeans.apache.org/) 、 [IntelliJ IDEA](https://web.archive.org/web/20221208143921/https://www.jetbrains.com/idea/) 和 [Eclipse](https://web.archive.org/web/20221208143921/https://www.eclipse.org/downloads/) 执行重构的方式略有不同，所以请确保首先检查您的 IDE 文档。

与许多其他重构模式一样，我们应该谨慎使用静态工厂方法，并且只有在产生更灵活和干净的设计与实现额外方法的成本之间值得权衡时才这样做。

像往常一样，本文中显示的所有代码示例都可以在 GitHub 上的[处获得。](https://web.archive.org/web/20221208143921/https://github.com/eugenp/tutorials/tree/master/patterns-modules/design-patterns-creational)