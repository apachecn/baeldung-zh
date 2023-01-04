# Mockito @Mock、@Spy、@Captor 和@InjectMocks 入门

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/mockito-annotations>

## 1。概述

在本教程中，我们将介绍以下莫克托库的**注释:** `@Mock`、`@Spy`、`@Captor`和`@InjectMocks`。

想知道更多的好消息，请看这里的系列文章。

## 延伸阅读:

## [mock ITO——利用间谍](/web/20220706105622/https://www.baeldung.com/mockito-spy)

Making good use of Spies in Mockito, and how spies are different from mocks.[Read more](/web/20220706105622/https://www.baeldung.com/mockito-spy) →

## [mock ITO vs EasyMock vs JMockit](/web/20220706105622/https://www.baeldung.com/mockito-vs-easymock-vs-jmockit)

A quick and practical guide to understanding and comparing Java mocking libraries.[Read more](/web/20220706105622/https://www.baeldung.com/mockito-vs-easymock-vs-jmockit) →

## [将 Mockito Mocks 注入春豆](/web/20220706105622/https://www.baeldung.com/injecting-mocks-in-spring)

This article will show how to use dependency injection to insert Mockito mocks into Spring Beans for unit testing.[Read more](/web/20220706105622/https://www.baeldung.com/injecting-mocks-in-spring) →

## 2。启用 Mockito 注释

在我们继续之前，让我们探索一下在 Mockito 测试中使用注释的不同方法。

### 2.1.`MockitoJUnitRunner`

我们的第一个选择是用`MockitoJUnitRunner` 来注释 JUnit 测试:

```java
@RunWith(MockitoJUnitRunner.class)
public class MockitoAnnotationTest {
    ...
}
```

### 2.2.`MockitoAnnotations.initMocks()`

或者，我们可以通过调用`MockitoAnnotations.initMocks()`以编程方式来**启用 Mockito 注释:**

```java
@Before
public void init() {
    MockitoAnnotations.initMocks(this);
}
```

### 2.3.`MockitoJUnit.rule()`

最后，**我们可以用一个`MockitoJUnit.rule()`** :

```java
public class MockitoInitWithMockitoJUnitRuleUnitTest {

    @Rule
    public MockitoRule initRule = MockitoJUnit.rule();

    ...
}
```

在这种情况下，我们必须记住制定我们的规则`public`。

## 3。`@Mock`注解

Mockito 中使用最广泛的注释是`@Mock`。我们可以使用`@Mock`来创建和注入模拟实例，而不必手动调用`Mockito.mock`。

在下面的例子中，我们将手动创建一个模拟的`ArrayList`，而不使用`@Mock`注释:

```java
@Test
public void whenNotUseMockAnnotation_thenCorrect() {
    List mockList = Mockito.mock(ArrayList.class);

    mockList.add("one");
    Mockito.verify(mockList).add("one");
    assertEquals(0, mockList.size());

    Mockito.when(mockList.size()).thenReturn(100);
    assertEquals(100, mockList.size());
}
```

现在我们将做同样的事情，但是我们将使用`@Mock`注释注入模拟:

```java
@Mock
List<String> mockedList;

@Test
public void whenUseMockAnnotation_thenMockIsInjected() {
    mockedList.add("one");
    Mockito.verify(mockedList).add("one");
    assertEquals(0, mockedList.size());

    Mockito.when(mockedList.size()).thenReturn(100);
    assertEquals(100, mockedList.size());
}
```

请注意，在这两个示例中，我们是如何与 mock 交互并验证其中一些交互的，只是为了确保 mock 行为正确。

## 4。`@Spy`注解

现在让我们看看如何使用`@Spy`注释来监视一个现有的实例。

在下面的例子中，我们创建了一个`List`的间谍，而没有使用`@Spy`注释:

```java
@Test
public void whenNotUseSpyAnnotation_thenCorrect() {
    List<String> spyList = Mockito.spy(new ArrayList<String>());

    spyList.add("one");
    spyList.add("two");

    Mockito.verify(spyList).add("one");
    Mockito.verify(spyList).add("two");

    assertEquals(2, spyList.size());

    Mockito.doReturn(100).when(spyList).size();
    assertEquals(100, spyList.size());
}
```

现在我们将做同样的事情，监视列表，但是我们将使用`@Spy`注释:

```java
@Spy
List<String> spiedList = new ArrayList<String>();

@Test
public void whenUseSpyAnnotation_thenSpyIsInjectedCorrectly() {
    spiedList.add("one");
    spiedList.add("two");

    Mockito.verify(spiedList).add("one");
    Mockito.verify(spiedList).add("two");

    assertEquals(2, spiedList.size());

    Mockito.doReturn(100).when(spiedList).size();
    assertEquals(100, spiedList.size());
}
```

注意，和以前一样，我们在这里是如何与间谍交互的，以确保它的行为正确。在本例中，我们:

*   使用了**真正的**方法`spiedList.add()`向`spiedList`添加元素。
*   **拔**的方法`spiedList.size()`返回`100`而不是使用`Mockito.doReturn()`返回`2`。

## 5。`@Captor`注解

接下来让我们看看如何使用`@Captor`注释来创建一个`ArgumentCaptor`实例。

在下面的例子中，我们将创建一个`ArgumentCaptor`而不使用`@Captor`注释:

```java
@Test
public void whenNotUseCaptorAnnotation_thenCorrect() {
    List mockList = Mockito.mock(List.class);
    ArgumentCaptor<String> arg = ArgumentCaptor.forClass(String.class);

    mockList.add("one");
    Mockito.verify(mockList).add(arg.capture());

    assertEquals("one", arg.getValue());
}
```

现在让**出于同样的目的利用`@Captor`** 来创建一个`ArgumentCaptor` 实例:

```java
@Mock
List mockedList;

@Captor 
ArgumentCaptor argCaptor;

@Test
public void whenUseCaptorAnnotation_thenTheSam() {
    mockedList.add("one");
    Mockito.verify(mockedList).add(argCaptor.capture());

    assertEquals("one", argCaptor.getValue());
}
```

请注意，当我们去掉配置逻辑时，测试变得更加简单，可读性更强。

## 6。`@InjectMocks`注解

现在让我们讨论如何使用`@InjectMocks`注释将模拟字段自动注入到测试对象中。

在下面的例子中，我们将使用`@InjectMocks`将模拟`wordMap`注入到`MyDictionary`T3 中:

```java
@Mock
Map<String, String> wordMap;

@InjectMocks
MyDictionary dic = new MyDictionary();

@Test
public void whenUseInjectMocksAnnotation_thenCorrect() {
    Mockito.when(wordMap.get("aWord")).thenReturn("aMeaning");

    assertEquals("aMeaning", dic.getMeaning("aWord"));
}
```

下面是这个类:

```java
public class MyDictionary {
    Map<String, String> wordMap;

    public MyDictionary() {
        wordMap = new HashMap<String, String>();
    }
    public void add(final String word, final String meaning) {
        wordMap.put(word, meaning);
    }
    public String getMeaning(final String word) {
        return wordMap.get(word);
    }
} 
```

## 7。给间谍注射模拟药物

与上面的测试类似，我们可能想给一个间谍注入一个模拟:

```java
@Mock
Map<String, String> wordMap;

@Spy
MyDictionary spyDic = new MyDictionary();
```

**然而，Mockito 不支持将 mocks 注入间谍，**并且以下测试结果出现异常:

```java
@Test 
public void whenUseInjectMocksAnnotation_thenCorrect() { 
    Mockito.when(wordMap.get("aWord")).thenReturn("aMeaning"); 

    assertEquals("aMeaning", spyDic.getMeaning("aWord")); 
}
```

如果我们想将 mock 与 spy 一起使用，我们可以通过构造函数手动注入 mock:

```java
MyDictionary(Map<String, String> wordMap) {
    this.wordMap = wordMap;
}
```

我们现在可以手动创建间谍，而不是使用注释:

```java
@Mock
Map<String, String> wordMap; 

MyDictionary spyDic;

@Before
public void init() {
    MockitoAnnotations.initMocks(this);
    spyDic = Mockito.spy(new MyDictionary(wordMap));
} 
```

测试现在将通过。

## 8。一边跑进 NPE 一边用标注

当我们试图实际使用标注有`@Mock` 或`@Spy`的实例时，我们经常会**碰到`NullPointerException`** :

```java
public class MockitoAnnotationsUninitializedUnitTest {

    @Mock
    List<String> mockedList;

    @Test(expected = NullPointerException.class)
    public void whenMockitoAnnotationsUninitialized_thenNPEThrown() {
        Mockito.when(mockedList.size()).thenReturn(1);
    }
}
```

大多数时候，这只是因为我们忘记正确启用 Mockito 注释。

因此，我们必须记住，每次我们想要使用 Mockito 注释时，我们都必须采取额外的步骤，并像我们之前已经解释过的那样初始化它们。

## 9。注释

最后，这里是**一些关于 Mockito 注释的注释**:

*   Mockito 的注释最小化了重复的模拟创建代码。
*   它们使测试更具可读性。
*   `@InjectMocks`是注入`@Spy`和 `@Mock`实例所必需的。

## 10。结论

在这篇简短的文章中，我们解释了 Mockito 库中**注释的基础知识。**

所有这些例子的实现都可以在 GitHub 上找到[。这是一个 Maven 项目，因此应该很容易导入和运行。](https://web.archive.org/web/20220706105622/https://github.com/eugenp/tutorials/tree/master/testing-modules/mockito)

当然，更多的 mock ITO good，[看看这里的系列](/web/20220706105622/https://www.baeldung.com/tag/mockito/)。