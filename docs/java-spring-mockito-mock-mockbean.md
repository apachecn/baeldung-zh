# Mock ITO . Mock()vs @ Mock vs @ Mock bean

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-spring-mockito-mock-mockbean>

## 1.概观

在这个快速教程中，我们将看看用 Mockito 和 Spring mocking 支持创建 mock 对象的三种不同方式。我们还将讨论它们之间的区别。

## 延伸阅读:

## [模拟参数匹配](/web/20220710173358/https://www.baeldung.com/mockito-argument-matchers)

Learn how to use the ArgumentMatcher and how it differs from the ArgumentCaptor.[Read more](/web/20220710173358/https://www.baeldung.com/mockito-argument-matchers) →

## [使用 Mockito 模拟异常抛出](/web/20220710173358/https://www.baeldung.com/mockito-exceptions)

Learn to configure a method call to throw an exception in Mockito.[Read more](/web/20220710173358/https://www.baeldung.com/mockito-exceptions) →

## 2.`Mockito.mock()`

`Mockito.mock()`方法允许我们创建一个类或接口的模拟对象。

然后，我们可以使用 mock 来为其方法存根返回值，并验证它们是否被调用。

让我们看一个例子:

```java
@Test
public void givenCountMethodMocked_WhenCountInvoked_ThenMockedValueReturned() {
    UserRepository localMockRepository = Mockito.mock(UserRepository.class);
    Mockito.when(localMockRepository.count()).thenReturn(111L);

    long userCount = localMockRepository.count();

    Assert.assertEquals(111L, userCount);
    Mockito.verify(localMockRepository).count();
}
```

在使用这个方法之前，我们不需要对它做任何其他的事情。我们可以用它来创建模拟类字段，以及方法中的本地模拟。

## 3.莫奇托的`@Mock`注解

这个注释是`Mockito.mock()`方法的简写。需要注意的是，我们应该只在测试类中使用它。与`mock()`方法不同，我们需要启用 Mockito 注释来使用这个注释。

我们可以通过使用`MockitoJUnitRunner`来运行测试，或者通过显式调用`MockitoAnnotations.initMocks()`方法来做到这一点。

让我们看一个使用`MockitoJUnitRunner`的例子:

```java
@RunWith(MockitoJUnitRunner.class)
public class MockAnnotationUnitTest {

    @Mock
    UserRepository mockRepository;

    @Test
    public void givenCountMethodMocked_WhenCountInvoked_ThenMockValueReturned() {
        Mockito.when(mockRepository.count()).thenReturn(123L);

        long userCount = mockRepository.count();

        Assert.assertEquals(123L, userCount);
        Mockito.verify(mockRepository).count();
    }
}
```

除了使代码可读性更好之外， **`@Mock`还使得在失败的情况下更容易找到问题模拟，因为字段的名称出现在失败消息中:**

```java
Wanted but not invoked:
mockRepository.count();
-> at org.baeldung.MockAnnotationTest.testMockAnnotation(MockAnnotationTest.java:22)
Actually, there were zero interactions with this mock.

  at org.baeldung.MockAnnotationTest.testMockAnnotation(MockAnnotationTest.java:22) 
```

此外，当与`@InjectMocks`结合使用时，它可以显著减少设置代码的数量。

## 4.Spring Boot 的`@MockBean`注解

我们可以使用`@MockBean`将模拟对象添加到 Spring 应用程序上下文中。mock 将替换应用程序上下文中相同类型的任何现有 bean。

如果没有定义相同类型的 bean，将添加一个新的。这个注释在集成测试中很有用，在集成测试中，需要模拟一个特定的 bean，比如一个外部服务。

为了使用这个注释，我们必须使用`SpringRunner`来运行测试:

```java
@RunWith(SpringRunner.class)
public class MockBeanAnnotationIntegrationTest {

    @MockBean
    UserRepository mockRepository;

    @Autowired
    ApplicationContext context;

    @Test
    public void givenCountMethodMocked_WhenCountInvoked_ThenMockValueReturned() {
        Mockito.when(mockRepository.count()).thenReturn(123L);

        UserRepository userRepoFromContext = context.getBean(UserRepository.class);
        long userCount = userRepoFromContext.count();

        Assert.assertEquals(123L, userCount);
        Mockito.verify(mockRepository).count();
    }
}
```

当我们在字段上使用注释时，mock 将被注入到字段中，并在应用程序上下文中注册。

这在上面的代码中显而易见。这里，我们使用注入的`UserRepository `模仿来存根`count `方法`.`，然后我们使用来自应用程序上下文的 bean 来验证它确实是被模仿的 bean。

## 5.结论

在本文中，我们研究了创建模拟对象的三种方法的区别，以及如何使用每一种方法。

本文附带的源代码可以在 GitHub 上的[处获得。](https://web.archive.org/web/20220710173358/https://github.com/eugenp/tutorials/tree/master/testing-modules/spring-testing)