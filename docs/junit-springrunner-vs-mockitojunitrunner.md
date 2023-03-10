# spring runner vs MockitoJUnitRunner

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/junit-springrunner-vs-mockitojunitrunner>

## 1.概观

JUnit 是 Java 中最流行的单元测试框架之一。此外，Spring Boot 将它作为应用程序的默认测试依赖项。

在本教程中，我们将比较两个 JUnit 跑步者—`SpringRunner`和`MockitoJUnitRunner`。我们将了解它们的目的以及它们之间的主要区别。

## 2.`@RunWith`对`@ExtendWith`

在我们进一步讨论之前，让我们回顾一下如何扩展基本的 JUnit 功能或将其与其他库集成。

[**JUnit 4 允许我们通过应用额外的功能来实现定制的`Runner`类**](/web/20221115052649/https://www.baeldung.com/junit-4-custom-runners) 。为了调用一个定制的运行器，我们使用一个`@RunWith`注释来注释一个测试类:

```java
@RunWith(CustomRunner.class)
class JUnit4Test {
    // ...
}
```

正如我们所知，JUnit 4 现在处于遗留状态，由 JUnit 5 接替。新版本带给我们[一个全新的引擎，带有重写的 API](/web/20221115052649/https://www.baeldung.com/junit-5-migration) 。它还改变了扩展模型的概念。我们现在可以使用带有`@ExtendWith`注释 的`Extension` API，而不是实现定制的`Runner`或`Rule`[**类:**](/web/20221115052649/https://www.baeldung.com/junit-5-extensions)

```java
@ExtendWith(CustomExtensionOne.class)
@ExtendWith(CustomExtensionTwo.class)
class JUnit5Test {
    // ...
}
```

与之前的 runner 模型不同，我们可以为单个类提供多个扩展。大多数以前交付的 runners 也为它们的扩展副本进行了重写。

## 3.Spring 示例应用程序

为了更好地理解，让我们介绍一个简单的 Spring Boot 应用程序——一个将给定字符串映射为大写的转换器。

让我们从数据提供者的实现开始:

```java
@Component
public class DataProvider {

    private final List<String> memory = List.of("baeldung", "java", "dummy");

    public Stream<String> getValues() {
        return memory.stream();
    }
}
```

我们刚刚创建了一个带有硬编码字符串值的 Spring 组件。此外，它提供了一个单一的方法来传输这些字符串。

其次，让我们实现一个转变我们价值观的服务类:

```java
@Service
public class StringConverter {

    private final DataProvider dataProvider;

    @Autowired
    public StringConverter(DataProvider dataProvider) {
        this.dataProvider = dataProvider;
    }

    public List<String> convert() {
        return dataProvider.getValues().map(String::toUpperCase).toList();
    }
}
```

这是一个简单的 bean，它从先前创建的`DataProvider`中获取数据，并应用大写映射。

现在，我们可以使用我们的应用程序来创建 JUnit 测试。我们将看到`SpringRunner`和`MockitoJUnitRunner `类之间的区别。

## 4.`MockitoJUnitRunner`

正如我们所知， [Mockito 是一个模仿框架](/web/20221115052649/https://www.baeldung.com/mockito-annotations)，它与其他测试框架一起使用，以返回虚拟数据并避免外部依赖。这个库为我们提供了**[`MockitoJUnitRunner`](https://web.archive.org/web/20221115052649/https://www.javadoc.io/doc/org.mockito/mockito-core/latest/org/mockito/junit/MockitoJUnitRunner.html)——一个专用的 JUnit 4 runner 来集成 Mockito** 并利用该库的功能。

现在让我们为`StringConverter`创建第一个测试:

```java
public class StringConverterTest {
    @Mock
    private DataProvider dataProvider;

    @InjectMocks
    private StringConverter stringConverter;

    @Test
    public void givenStrings_whenConvert_thenReturnUpperCase() {
        Mockito.when(dataProvider.getValues()).thenReturn(Stream.of("first", "second"));

        val result = stringConverter.convert();

        Assertions.assertThat(result).contains("FIRST", "SECOND");
    }
}
```

我们刚刚模仿我们的`DataProvider`返回了两个字符串。但是如果我们运行它，测试就会失败:

```java
java.lang.NullPointerException: Cannot invoke "DataProvider.getValues()" because "this.dataProvider" is null
```

这是因为我们的模拟没有正确初始化。`@Mock`和`@InjectMocks`注释目前什么都不做。我们可以通过实现`init()`方法来解决这个问题:

```java
@Before
public void init() {
    MockitoAnnotations.openMocks(this);
}
```

如果我们不想使用注释，我们也可以以编程方式创建和注入模拟:

```java
@Before
public void init() {
    dataProvider = Mockito.mock(DataProvider.class);
    stringConverter = new StringConverter(dataProvider);
}
```

我们刚刚使用 Mockito API 以编程方式初始化了 mocks。现在，测试按预期工作，我们的断言成功了。

接下来，让我们回到我们的第一个版本，删除`init()`方法，并使用`MockitoJUnitRunner`注释该类:

```java
@RunWith(MockitoJUnitRunner.class)
public class StringConverterTest {
    // ...
}
```

测试再次成功。我们调用了一个自定义运行器，它负责管理我们的模拟。我们不必手动初始化它们。

综上所述， **`MockitoJUnitRunner`是 Mockito 框架**的专用运行器。**负责初始化`@Mock`、`@Spy`和`@InjectMock`注释**，这样就不需要显式使用[、](https://web.archive.org/web/20221115052649/https://www.javadoc.io/static/org.mockito/mockito-core/4.8.1/org/mockito/MockitoAnnotations.html#openMocks(java.lang.Object))。此外，**检测测试中未使用的存根，并在每个测试方法之后验证模拟用法**，就像`[Mockito.validateMockitoUsage()](https://web.archive.org/web/20221115052649/https://www.javadoc.io/doc/org.mockito/mockito-core/latest/org/mockito/Mockito.html#validateMockitoUsage())`一样。

我们应该记住，所有的运行程序最初都是为 JUnit 4 设计的。如果我们想在 JUnit 5 中支持 Mockito 注释，我们可以使用`MockitoExtension`:

```java
@ExtendWith(MockitoExtension.class)
public class StringConverterTest {
    // ...
}
```

这个扩展将功能从`MockitoJUnitRunner `移植到新的扩展模型中。

## 5.`SpringRunner`

如果我们更深入地分析我们的测试，我们会发现，尽管使用了 Spring，我们根本没有启动 Spring 容器。现在让我们试着修改我们的例子并初始化一个 Spring 上下文。

首先，让我们用 [`SpringRunner`](https://web.archive.org/web/20221115052649/https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/test/context/junit4/SpringRunner.html) 类代替`MockitoJUnitRunner`，并检查结果:

```java
@RunWith(SpringRunner.class)
public class StringConverterTest {
    // ...
}
```

和以前一样，测试成功，模拟被正确初始化。此外，还启动了一个 Spring 上下文。我们可以得出结论， **`SpringRunner`不仅启用了 Mockito 注释，就像`MockitoJUnitRunner`一样，还初始化了一个 Spring 上下文**。

当然，我们在测试中还没有看到 Spring 的全部潜力。我们可以将它们作为 Spring beans 注入，而不是构造新的对象。正如我们所知，Spring 测试模块默认集成了 Mockito，它还为我们提供了 [`@MockBean`和`@SpyBean`注释](/web/20221115052649/https://www.baeldung.com/java-spring-mockito-mock-mockbean#spring-boots-mockbean-annotation)——将 mocks 和 beans 特性集成在一起。

让我们重写我们的测试:

```java
@ContextConfiguration(classes = StringConverter.class)
@RunWith(SpringRunner.class)
public class StringConverterTest {
    @MockBean
    private DataProvider dataProvider;

    @Autowired
    private StringConverter stringConverter;

    // ...
}
```

我们只是用`DataProvider`对象旁边的`@MockBean`替换了`@Mock`注释。它仍然是一个模拟，但现在也可以作为一个豆使用。我们还通过`@ContextConfiguration`类注释配置了我们的 Spring 上下文，并注入了`StringConverter`。结果，测试仍然成功，但是它现在一起使用了 Spring beans 和 Mockito。

综上所述， **`SpringRunner`是为 JUnit 4 创建的自定义运行器，提供了 Spring TestContext 框架**的功能。由于 Mockito 是与 Spring 栈集成的默认 mock 框架，**runner 带来了由`MockitoJUnitRunner`** 提供的完全支持。有时，我们也可能会碰到 **`SpringJUnit4ClassRunner`，它是**的别名，我们可以交替使用两者。

如果我们要为`SpringRunner`寻找一个对应的扩展，我们应该使用`SpringExtension`:

```java
@ExtendWith(SpringExtension.class)
public class StringConverterTest {
    // ...
}
```

由于 JUnit 5 是 Spring Boot 堆栈中的默认测试框架，该扩展已经集成了许多测试切片注释，包括`@SpringBootTest`。

## 6.结论

在本文中，我们了解了`SpringRunner`和`MockitoJUnitRunner`的区别。

我们首先回顾了 JUnit 4 和 JUnit 5 中使用的扩展模型。JUnit 4 使用专用的运行器，而 JUnit 5 支持扩展。同时，我们可以提供单个流道或多个扩展。

然后，**我们看了看`MockitoJUnitRunner`，这** **使得 Mockito 框架在我们的 JUnit 4 测试中得到支持**。一般来说，我们可以通过专用的`@Mock`、`@Spy`和`@InjectMocks`注释来配置模拟，而不需要任何初始化方法。

最后，**我们讨论了`SpringRunner`，它释放了 Mockito 和 Spring 框架之间合作的所有优势。**它不仅支持基本的 Mockito 注释，还支持 Spring 注释:@ `MockBean`和`@SpyBean`。以这种方式构建的模拟可以使用 Spring 上下文注入。

和往常一样，这些例子的完整实现可以在 GitHub 上找到[。](https://web.archive.org/web/20221115052649/https://github.com/eugenp/tutorials/tree/master/testing-modules/spring-mockito)