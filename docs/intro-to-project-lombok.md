# 龙目岛项目简介

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/intro-to-project-lombok>

## 1。避免重复代码

Java 是一种很棒的语言，但是对于我们必须在代码中完成的常见任务或遵循一些框架实践来说，它有时会变得太冗长。这通常不会给我们程序的商业方面带来任何真正的价值，这就是 Lombok 让我们更有效率的地方。

它的工作方式是插入到我们的构建过程中，并根据我们在代码中引入的大量项目注释自动生成 Java 字节码到我们的`.class`文件中。

## 延伸阅读:

## [Lombok 构建器，默认值为](/web/20221127130246/https://www.baeldung.com/lombok-builder-default-value)

Learn how to create a builder default property values using Lombok[Read more](/web/20221127130246/https://www.baeldung.com/lombok-builder-default-value) →

## [用 Eclipse 和 Intellij 设置 Lombok](/web/20221127130246/https://www.baeldung.com/lombok-ide)

Learn how to set up Lombok with popular IDEs[Read more](/web/20221127130246/https://www.baeldung.com/lombok-ide) →

在我们的构建中包括它，在我们使用的任何系统中，都是非常直接的。Project Lombok 的[项目页面](https://web.archive.org/web/20221127130246/https://projectlombok.org/features/index.html)有关于具体细节的详细说明。我的大多数项目都是基于 maven 的，所以我通常只在`provided`范围内放弃它们的依赖性，这样就可以了:

```java
<dependencies>
    ...
    <dependency>
        <groupId>org.projectlombok</groupId>
        <artifactId>lombok</artifactId>
        <version>1.18.20</version>
        <scope>provided</scope>
    </dependency>
    ...
</dependencies>
```

我们可以在这里查看最新的可用版本[。](https://web.archive.org/web/20221127130246/https://projectlombok.org/changelog.html)

注意，依赖 Lombok 不会让我们的用户也依赖它，因为它是一个纯粹的构建依赖，而不是运行时。

## 2。getter/setter，Constructors 如此重复

通过公共 getter 和 setter 方法封装对象属性在 Java 世界中是一种常见的做法，许多框架都广泛依赖于这种“Java Bean”模式(一个具有空构造函数和“属性”get/set 方法的类)。

这种情况非常普遍，以至于大多数 IDE 都支持为这些模式(以及更多)自动生成代码。然而，这些代码需要存在于我们的源代码中，并在添加新属性或重命名字段时进行维护。

让我们考虑一下我们想用作 JPA 实体的这个类:

```java
@Entity
public class User implements Serializable {

    private @Id Long id; // will be set when persisting

    private String firstName;
    private String lastName;
    private int age;

    public User() {
    }

    public User(String firstName, String lastName, int age) {
        this.firstName = firstName;
        this.lastName = lastName;
        this.age = age;
    }

    // getters and setters: ~30 extra lines of code
}
```

这是一个相当简单的类，但是想象一下，如果我们为 getters 和 setters 添加了额外的代码。我们最终会得到一个定义，其中会有比相关业务信息更多的样板零值代码:“用户有名字和姓氏，以及年龄。”

咱们现在`Lombok-ize`这节课:

```java
@Entity
@Getter @Setter @NoArgsConstructor // <--- THIS is it
public class User implements Serializable {

    private @Id Long id; // will be set when persisting

    private String firstName;
    private String lastName;
    private int age;

    public User(String firstName, String lastName, int age) {
        this.firstName = firstName;
        this.lastName = lastName;
        this.age = age;
    }
}
```

通过添加`@Getter`和`@Setter`注释，我们告诉 Lombok 为类的所有字段生成这些注释。`@NoArgsConstructor`会导致空的构造函数生成。

注意，这是**整个**类的代码，我们没有省略任何与上面带有`// getters and setters`注释的版本不同的内容。对于一个三个相关属性的类来说，这是一个显著的代码节省！

如果我们进一步向我们的`User`类添加属性(properties)，同样的情况也会发生；我们将注释应用于类型本身，因此它们默认会考虑所有字段。

如果我们想细化某些属性的可见性怎么办？例如，如果我们希望保持实体的`id`字段修饰符`package`或`protected`可见，因为它们应该被读取，但不是由应用程序代码显式设置的，我们可以对这个特定的字段使用更细粒度的`@Setter`:

```java
private @Id @Setter(AccessLevel.PROTECTED) Long id;
```

## 3.懒惰的吸气者

应用程序通常需要执行昂贵的操作，并保存结果以备后用。

例如，假设我们需要从文件或数据库中读取静态数据。通常情况下，检索一次数据，然后缓存它以允许在应用程序中进行内存读取是一种很好的做法。这使得应用程序不必重复昂贵的操作。

另一个常见的模式是**仅在第一次需要时检索这些数据**。换句话说，我们**只有在第一次调用对应的 getter 时才会得到数据。**我们称之为`lazy-loading`。

让我们假设这些数据被缓存为一个类中的一个字段。该类现在必须确保对该字段的任何访问都返回缓存的数据。实现这样一个类的一种可能的方法是让 getter 方法仅在字段为`null`时检索数据。**我们称之为`lazy`吸气剂**。

Lombok 通过我们上面看到的@ `Getter`注释中的 **`lazy` 参数实现了这一点。**

例如，考虑这个简单的类:

```java
public class GetterLazy {

    @Getter(lazy = true)
    private final Map<String, Long> transactions = getTransactions();

    private Map<String, Long> getTransactions() {

        final Map<String, Long> cache = new HashMap<>();
        List<String> txnRows = readTxnListFromFile();

        txnRows.forEach(s -> {
            String[] txnIdValueTuple = s.split(DELIMETER);
            cache.put(txnIdValueTuple[0], Long.parseLong(txnIdValueTuple[1]));
        });

        return cache;
    }
}
```

这将一些事务从一个文件读入一个`Map`。由于文件中的数据不会改变，我们将缓存它一次，并允许通过 getter 访问。

如果我们现在查看这个类的编译代码，我们会看到一个 **getter 方法，如果它是`null`的话，它会更新缓存，然后返回缓存的数据**:

```java
public class GetterLazy {

    private final AtomicReference<Object> transactions = new AtomicReference();

    public GetterLazy() {
    }

    //other methods

    public Map<String, Long> getTransactions() {
        Object value = this.transactions.get();
        if (value == null) {
            synchronized(this.transactions) {
                value = this.transactions.get();
                if (value == null) {
                    Map<String, Long> actualValue = this.readTxnsFromFile();
                    value = actualValue == null ? this.transactions : actualValue;
                    this.transactions.set(value);
                }
            }
        }

        return (Map)((Map)(value == this.transactions ? null : value));
    }
}
```

有趣的是指出 **Lombok 将数据字段包装在** `**[AtomicReference](/web/20221127130246/https://www.baeldung.com/java-atomic-variables).**` 中，这确保了对`transactions` 字段的原子更新。如果`transactions` 是`null.`，那么`getTransactions()`方法也确保读取文件

我们不鼓励在类中直接使用`AtomicReference transactions` 字段。**我们建议使用`getTransactions()`方法进入现场。**

出于这个原因，如果我们在同一个类`,`中使用另一个 Lombok 注释，比如`ToString`，它将使用`getTransactions()`，而不是直接访问该字段。

## 4。价值阶层/DTO 的

在许多情况下，我们希望定义一种数据类型，其唯一目的是将复杂的“值”表示为“数据传输对象”，大多数时候是以不可变的数据结构的形式，我们构建一次，就再也不想改变。

我们设计了一个类来表示成功的登录操作。我们希望所有字段都是非空的，对象都是不可变的，这样我们就可以安全地访问它的属性:

```java
public class LoginResult {

    private final Instant loginTs;

    private final String authToken;
    private final Duration tokenValidity;

    private final URL tokenRefreshUrl;

    // constructor taking every field and checking nulls

    // read-only accessor, not necessarily as get*() form
}
```

同样，我们必须为注释部分编写的代码量将比我们想要封装的信息量大得多。我们可以使用 Lombok 来改进这一点:

```java
@RequiredArgsConstructor
@Accessors(fluent = true) @Getter
public class LoginResult {

    private final @NonNull Instant loginTs;

    private final @NonNull String authToken;
    private final @NonNull Duration tokenValidity;

    private final @NonNull URL tokenRefreshUrl;

}
```

一旦我们添加了`@RequiredArgsConstructor`注释，我们将为类中所有的最终字段获得一个构造函数，就像我们声明它们一样。将`@NonNull`添加到属性中会使我们的构造函数检查可空性，并相应地抛出`NullPointerExceptions`。如果字段是非最终的，我们为它们添加了`@Setter`，也会出现这种情况。

我们希望我们的财产采用无聊的旧形式吗？因为我们在这个例子中添加了`@Accessors(fluent=true)`,“getters”将具有与属性相同的方法名；` getAuthToken()`干脆变成了`authToken()`。

这种“流畅”的形式适用于属性设置器的非最终字段，并允许链式调用:

```java
// Imagine fields were no longer final now
return new LoginResult()
  .loginTs(Instant.now())
  .authToken("asdasd")
  . // and so on
```

## 5。核心 Java 样板文件

另一种我们需要维护代码的情况是在生成`toString()`、`equals()`和`hashCode()`方法的时候。ide 试图用模板来帮助根据我们的类属性自动生成这些。

我们可以通过其他 Lombok 类级注释来实现自动化:

*   [`@ToString`](https://web.archive.org/web/20221127130246/https://projectlombok.org/features/ToString.html) :将生成一个包含所有类属性的`toString()`方法。当我们丰富我们的数据模型时，不需要我们自己写一个并维护它。
*   [`@EqualsAndHashCode`](https://web.archive.org/web/20221127130246/https://projectlombok.org/features/EqualsAndHashCode.html) :默认情况下会生成`equals()`和`hashCode()`两种方法，考虑到所有相关的字段，虽然语义也很好的按照[生成。](https://web.archive.org/web/20221127130246/http://www.artima.com/lejava/articles/equality.html)

这些生成器提供了非常方便的配置选项。例如，如果我们的带注释的类是层次结构的一部分，我们可以只使用`callSuper=true`参数，在生成方法的代码时将考虑父结果。

为了演示这一点，让我们假设我们的`User` JPA 实体示例包含了对与该用户相关联的事件的引用:

```java
@OneToMany(mappedBy = "user")
private List<UserEvent> events;
```

我们不希望在调用用户的`toString()`方法时，仅仅因为使用了`@ToString`注释，就丢弃整个事件列表。相反，我们可以这样参数化它，` @ToString(exclude = {“events”})`，这不会发生。这也有助于避免循环引用，例如，`UserEvent` s 引用了一个`User`。

对于`LoginResult`的例子，我们可能希望仅仅根据令牌本身而不是类中的其他最终属性来定义等式和哈希代码计算。那么我们可以简单地写一些类似于`@EqualsAndHashCode(of = {“authToken”})`的东西。

如果到目前为止我们已经讨论过的注释的特性是我们感兴趣的，我们可能还想检查一下 [`@Data`](https://web.archive.org/web/20221127130246/https://projectlombok.org/features/Data.html) 和 [`@Value`](https://web.archive.org/web/20221127130246/https://projectlombok.org/features/Value.html) 注释，因为它们的行为就好像它们中的一组已经被应用到我们的类中。毕竟，这些讨论过的用法在很多情况下是很常见的。

### 5.1.(不)使用 `@EqualsAndHashCode`与 JPA 实体

我们应该使用默认的`equals()`和`hashCode()`方法，还是为 JPA 实体创建自定义的方法，这是开发人员经常讨论的话题。有多种[方法](/web/20221127130246/https://www.baeldung.com/jpa-entity-equality)我们可以遵循，每种方法都有其优点和缺点。

**默认情况下，`@EqualsAndHashCode`包含实体类的所有非最终属性。**我们可以尝试通过使用`@EqualsAndHashCode`的`onlyExplicitlyIncluded` 属性让 Lombok 只使用实体的主键来“修复”这个问题。尽管如此，生成的`equals()`方法可能会导致一些问题。索本·让桑在他的一篇博客文章中详细解释了这个场景。

一般来说，**我们应该避免使用 Lombok 来为我们的 JPA 实体生成`equals()`和`hashCode()`方法。**

## 6。构建器模式

下面是一个 REST API 客户端的配置类示例:

```java
public class ApiClientConfiguration {

    private String host;
    private int port;
    private boolean useHttps;

    private long connectTimeout;
    private long readTimeout;

    private String username;
    private String password;

    // Whatever other options you may thing.

    // Empty constructor? All combinations?

    // getters... and setters?
}
```

我们可以有一个基于使用类默认空构造函数和为每个字段提供 setter 方法的初始方法；然而，理想情况下，我们希望配置一旦被构建(实例化)，就不要被重新`set`化，有效地使它们成为不可变的。因此，我们希望避免 setters，但是编写这样一个潜在的长 args 构造函数是一种反模式。

相反，我们可以告诉工具生成一个`builder`模式，这样我们就不必编写额外的`Builder`类和相关的流畅的类似 setter 的方法，只需将@Builder 注释添加到我们的`ApiClientConfiguration:`中

```java
@Builder
public class ApiClientConfiguration {

    // ... everything else remains the same

}
```

抛开上面的类定义(没有声明构造函数或 setter+`@Builder`)，我们最终可以将它用作:

```java
ApiClientConfiguration config = 
    ApiClientConfiguration.builder()
        .host("api.server.com")
        .port(443)
        .useHttps(true)
        .connectTimeout(15_000L)
        .readTimeout(5_000L)
        .username("myusername")
        .password("secret")
    .build();
```

## 7。检查例外负担

许多 Java APIs 被设计成可以抛出一些检查过的异常；客户端代码被强制为`catch`或声明为`throws`。有多少次我们把这些我们知道不会发生的例外变成了这样的事情？：

```java
public String resourceAsString() {
    try (InputStream is = this.getClass().getResourceAsStream("sure_in_my_jar.txt")) {
        BufferedReader br = new BufferedReader(new InputStreamReader(is, "UTF-8"));
        return br.lines().collect(Collectors.joining("\n"));
    } catch (IOException | UnsupportedCharsetException ex) {
        // If this ever happens, then its a bug.
        throw new RuntimeException(ex); <--- encapsulate into a Runtime ex.
    }
}
```

如果我们想避免这种代码模式，因为编译器会不高兴(而且我们**知道**被检查的错误不会发生)，使用恰当地命名为 [`@SneakyThrows`](https://web.archive.org/web/20221127130246/https://projectlombok.org/features/SneakyThrows.html) :

```java
@SneakyThrows
public String resourceAsString() {
    try (InputStream is = this.getClass().getResourceAsStream("sure_in_my_jar.txt")) {
        BufferedReader br = new BufferedReader(new InputStreamReader(is, "UTF-8"));
        return br.lines().collect(Collectors.joining("\n"));
    } 
}
```

## 8。确保我们的资源被释放

Java 7 引入了 try-with-resources 块，以确保我们的资源被任何实现`java.lang`的实例持有。`AutoCloseable`退出时被释放。

Lombok 通过 [@Cleanup](https://web.archive.org/web/20221127130246/https://projectlombok.org/features/Cleanup.html) 提供了另一种更灵活的方式来实现这一点。我们可以将它用于任何局部变量，我们希望确保它的资源被释放。他们不需要实现任何特定的接口，我们只需要调用`close()`方法:

```java
@Cleanup InputStream is = this.getClass().getResourceAsStream("res.txt");
```

我们的释放方法有一个不同的名字？没问题，我们只是自定义注释:

```java
@Cleanup("dispose") JFrame mainFrame = new JFrame("Main Window");
```

## 9。注释我们的类以获得一个日志记录器

我们中的许多人通过从我们选择的框架中创建一个`Logger`实例来谨慎地将日志记录语句添加到代码中。比如说 SLF4J:

```java
public class ApiClientConfiguration {

    private static Logger LOG = LoggerFactory.getLogger(ApiClientConfiguration.class);

    // LOG.debug(), LOG.info(), ...

}
```

这是一个如此常见的模式，以至于 Lombok 开发人员为我们简化了它:

```java
@Slf4j // or: @Log @CommonsLog @Log4j @Log4j2 @XSlf4j
public class ApiClientConfiguration {

    // log.debug(), log.info(), ...

}
```

支持很多[日志框架](https://web.archive.org/web/20221127130246/https://projectlombok.org/features/Log.html)，当然我们也可以自定义实例名、主题等。

## 10。编写线程安全的方法

在 Java 中，我们可以使用`synchronized`关键字来实现临界区；然而，这不是 100%安全的方法。其他客户端代码最终也可以在我们的实例上同步，这可能会导致意外的死锁。

这就是 [`@Synchronized`](https://web.archive.org/web/20221127130246/https://projectlombok.org/features/Synchronized.html) 的用武之地。我们可以用它来注释我们的方法(实例的和静态的),我们将得到一个自动生成的、私有的、未公开的字段，我们的实现将使用它来锁定:

```java
@Synchronized
public /* better than: synchronized */ void putValueInCache(String key, Object value) {
    // whatever here will be thread-safe code
}
```

## 11。自动合成对象

Java 没有语言级别的构造来平滑“偏好组合继承”的方法。其他语言有内置的概念，如`Traits`或`Mixins`来实现这一点。

当我们想要使用这种编程模式时，Lombok 的 [@Delegate](https://web.archive.org/web/20221127130246/https://projectlombok.org/features/experimental/Delegate.html) 非常方便。让我们考虑一个例子:

*   我们希望`User` s 和`Customer` s 在命名和电话号码上有一些共同的属性。
*   我们为这些字段定义了接口和适配器类。
*   我们将让我们的模型实现接口并`@Delegate`到它们的适配器，有效地用我们的联系信息`composing`它们。

首先，让我们定义一个接口:

```java
public interface HasContactInformation {

    String getFirstName();
    void setFirstName(String firstName);

    String getFullName();

    String getLastName();
    void setLastName(String lastName);

    String getPhoneNr();
    void setPhoneNr(String phoneNr);

}
```

现在一个适配器作为一个`support`类:

```java
@Data
public class ContactInformationSupport implements HasContactInformation {

    private String firstName;
    private String lastName;
    private String phoneNr;

    @Override
    public String getFullName() {
        return getFirstName() + " " + getLastName();
    }
}
```

现在是有趣的部分；看看将联系信息组合到两个模型类中是多么容易:

```java
public class User implements HasContactInformation {

    // Whichever other User-specific attributes

    @Delegate(types = {HasContactInformation.class})
    private final ContactInformationSupport contactInformation =
            new ContactInformationSupport();

    // User itself will implement all contact information by delegation

}
```

`Customer`的情况非常相似，为了简洁起见，我们可以省略这个例子。

## 12。滚回龙目岛？

简而言之:一点也不。

可能有人担心，如果我们在我们的一个项目中使用 Lombok，我们以后可能想要撤销那个决定。潜在的问题可能是有大量的类被注释了。在这种情况下，由于来自同一个项目的`delombok`工具，我们被覆盖。

通过`delombok-ing`我们的代码，我们从构建的字节码 Lombok 中获得了具有完全相同特性的自动生成的 Java 源代码。然后，我们可以简单地用这些新的`delomboked`文件替换我们原来的带注释的代码，并且不再依赖它。

这是我们可以[集成到我们的构建](https://web.archive.org/web/20221127130246/https://projectlombok.org/features/delombok.html)中的东西。

## 13。结论

还有一些其他特性我们没有在本文中介绍。我们可以更深入地了解[功能概述](https://web.archive.org/web/20221127130246/https://projectlombok.org/features/index.html)，了解更多细节和使用案例。

此外，我们展示的大多数函数都有许多定制选项，我们可能会觉得很方便。可用的内置[配置系统](https://web.archive.org/web/20221127130246/https://projectlombok.org/features/configuration.html)也可以在这方面帮助我们。

现在我们可以给 Lombok 一个进入我们的 Java 开发工具集的机会，我们可以提高我们的生产力。

示例代码可以在 [GitHub 项目](https://web.archive.org/web/20221127130246/https://github.com/eugenp/tutorials/tree/master/lombok-modules/lombok)中找到。