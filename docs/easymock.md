# EasyMock 简介

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/easymock>

## 1。简介

在过去，我们已经广泛讨论过 [JMockit](/web/20220703033325/https://www.baeldung.com/jmockit-101) 和 [Mockito](/web/20220703033325/https://www.baeldung.com/mockito-final) 。

在本教程中，我们将介绍另一个模仿工具—[EasyMock](https://web.archive.org/web/20220703033325/http://easymock.org/ "http://easymock.org/")。

## 2。Maven 依赖关系

在我们开始之前，让我们将下面的依赖项添加到我们的`pom.xml`中:

```java
<dependency>
    <groupId>org.easymock</groupId>
    <artifactId>easymock</artifactId>
    <version>3.5.1</version>
    <scope>test</scope>
</dependency>
```

最新版本总是可以在[这里](https://web.archive.org/web/20220703033325/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22org.easymock%22%20AND%20a%3A%22easymock%22 "https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22org.easymock%22%20AND%20a%3A%22easymock%22")找到。

## 3。核心概念

当生成一个 mock 时，**我们可以模拟目标对象，指定它的行为，并最终验证它是否按预期使用。**

使用 EasyMock 的模拟包括四个步骤:

1.  创建目标类的模拟
2.  记录其预期行为，包括动作、结果、异常等。
3.  在测试中使用模拟
4.  验证它是否按预期运行

在我们的记录完成后，我们将其切换到“重放”模式，这样当与任何将使用它的对象协作时，mock 的行为就像记录的一样。

最后，我们验证一切是否如预期的那样进行。

上述四个步骤与 [`org.easymock.EasyMock`](https://web.archive.org/web/20220703033325/http://easymock.org/api/org/easymock/EasyMock.html) 中的方法相关:

1.  **`[mock(…)](https://web.archive.org/web/20220703033325/http://easymock.org/api/org/easymock/EasyMock.html#mock-java.lang.Class- "http://easymock.org/api/org/easymock/EasyMock.html#mock-java.lang.Class-")`** :生成目标类的模拟，可以是具体的类，也可以是接口。一旦创建，模拟就处于“记录”模式，这意味着 EasyMock 将记录模拟对象采取的任何动作，并以“重放”模式重放它们
2.  **`[expect(…)](https://web.archive.org/web/20220703033325/http://easymock.org/api/org/easymock/EasyMock.html#expect-T- "http://easymock.org/api/org/easymock/EasyMock.html#expect-T-")`** :通过这种方法，我们可以为相关的记录动作设置期望，包括调用、结果和异常
3.  **`[replay(…)](https://web.archive.org/web/20220703033325/http://easymock.org/api/org/easymock/EasyMock.html#replay-java.lang.Object...- "http://easymock.org/api/org/easymock/EasyMock.html#replay-java.lang.Object...-")`** :将给定的模拟切换到“重放”模式。然后，任何触发先前记录的方法调用的动作将重放“记录的结果”
4.  **`[verify(…)](https://web.archive.org/web/20220703033325/http://easymock.org/api/org/easymock/EasyMock.html#verify-java.lang.Object...- "http://easymock.org/api/org/easymock/EasyMock.html#verify-java.lang.Object...-")`** :验证所有期望都得到满足，并且没有在模拟上执行意外调用

在下一节中，我们将使用真实的例子展示这些步骤是如何工作的。

## 4。嘲讽的实际例子

在我们继续之前，让我们看一下示例上下文:假设我们有一个 Baeldung 博客的读者，他/她喜欢在网站上浏览文章，然后他/她试图写文章。

让我们从创建以下模型开始:

```java
public class BaeldungReader {

    private ArticleReader articleReader;
    private IArticleWriter articleWriter;

    // constructors

    public BaeldungArticle readNext(){
        return articleReader.next();
    }

    public List<BaeldungArticle> readTopic(String topic){
        return articleReader.ofTopic(topic);
    }

    public String write(String title, String content){
        return articleWriter.write(title, content);
    }
}
```

在这个模型中，我们有两个私有成员:`articleReader`(一个具体的类)和`articleWriter` (一个接口)。

接下来，我们将模拟他们来验证`BaeldungReader`的行为。

## 5。用 Java 代码模仿

先来嘲讽一个`ArticleReader`吧。

### 5.1。典型嘲讽

我们期望当读者跳过一篇文章时调用`articleReader.next()`方法:

```java
@Test
public void whenReadNext_thenNextArticleRead(){
    ArticleReader mockArticleReader = mock(ArticleReader.class);
    BaeldungReader baeldungReader
      = new BaeldungReader(mockArticleReader);

    expect(mockArticleReader.next()).andReturn(null);
    replay(mockArticleReader);

    baeldungReader.readNext();

    verify(mockArticleReader);
}
```

在上面的示例代码中，我们严格遵循 4 步程序并模仿了`ArticleReader`类。

**虽然我们真的不在乎`mockArticleReader.next()`返回什么，但是我们仍然需要通过使用`expect(…).andReturn(…).`为`mockArticleReader.next()`** 指定一个返回值

对于`expect(…)`，EasyMock 希望该方法返回值或抛出`Exception.`

如果我们只是:

```java
mockArticleReader.next();
replay(mockArticleReader);
```

EasyMock 会抱怨这一点，因为如果方法返回任何东西，它需要调用`expect(…).andReturn(…)`。

如果是一种`void`方法，我们可以把`expect`它的动作用 [`expectLastCall()`](https://web.archive.org/web/20220703033325/http://easymock.org/api/org/easymock/EasyMock.html#expectLastCall--) 这样表示:

```java
mockArticleReader.someVoidMethod();
expectLastCall();
replay(mockArticleReader);
```

### 5.2。重播顺序

如果我们需要以特定的顺序重放动作，我们可以更加严格:

```java
@Test
public void whenReadNextAndSkimTopics_thenAllAllowed(){
    ArticleReader mockArticleReader
      = strictMock(ArticleReader.class);
    BaeldungReade baeldungReader
      = new BaeldungReader(mockArticleReader);

    expect(mockArticleReader.next()).andReturn(null);
    expect(mockArticleReader.ofTopic("easymock")).andReturn(null);
    replay(mockArticleReader);

    baeldungReader.readNext();
    baeldungReader.readTopic("easymock");

    verify(mockArticleReader);
}
```

在这个片段中，我们**使用`[strictMock(…)](https://web.archive.org/web/20220703033325/http://easymock.org/api/org/easymock/EasyMock.html#strictMock-java.lang.Class- "http://easymock.org/api/org/easymock/EasyMock.html#strictMock-java.lang.Class-")`来检查方法调用**的顺序。对于由`mock(…)`和`strictMock(…)`创建的模拟，任何意外的方法调用都会导致`AssertionError`。

**为了允许模拟的任何方法调用，我们可以使用`niceMock(…)` :**

```java
@Test
public void whenReadNextAndOthers_thenAllowed(){
    ArticleReader mockArticleReader = niceMock(ArticleReader.class);
    BaeldungReade baeldungReader = new BaeldungReader(mockArticleReader);

    expect(mockArticleReader.next()).andReturn(null);
    replay(mockArticleReader);

    baeldungReader.readNext();
    baeldungReader.readTopic("easymock");

    verify(mockArticleReader);
}
```

这里我们没想到`baeldungReader.readTopic(…)`会被调用，但是 EasyMock 不会抱怨。EasyMock 现在只关心目标对象是否执行了预期的动作。

### 5.3。嘲讽`Exception`抛出

现在，让我们继续模仿接口`IArticleWriter`，以及如何处理预期的`Throwables`:

```java
@Test
public void whenWriteMaliciousContent_thenArgumentIllegal() {
    // mocking and initialization

    expect(mockArticleWriter
      .write("easymock","<body onload=alert('baeldung')>"))
      .andThrow(new IllegalArgumentException());
    replay(mockArticleWriter);

    // write malicious content and capture exception as expectedException

    verify(mockArticleWriter);
    assertEquals(
      IllegalArgumentException.class, 
      expectedException.getClass());
}
```

在上面的代码片段中，我们期望`articleWriter`足够可靠，能够检测到 [XSS(跨站点脚本)](https://web.archive.org/web/20220703033325/https://www.owasp.org/index.php/Cross-site_Scripting_(XSS) "https://www.owasp.org/index.php/Cross-site_Scripting_(XSS)")攻击。

所以当读者试图在文章内容中注入恶意代码时，作者应该抛出一个`IllegalArgumentException`。我们使用`expect(…).andThrow(…)`记录了这一预期行为。

## 6。带注释的模拟

EasyMock 还支持使用注释注入模拟。为了使用它们，我们需要用`[EasyMockRunner](https://web.archive.org/web/20220703033325/http://easymock.org/api/org/easymock/EasyMockRunner.html "http://easymock.org/api/org/easymock/EasyMockRunner.html")`运行我们的单元测试，以便它处理 [`@Mock`](https://web.archive.org/web/20220703033325/http://easymock.org/api/org/easymock/Mock.html "http://easymock.org/api/org/easymock/Mock.html") 和 [`@TestSubject`](https://web.archive.org/web/20220703033325/http://easymock.org/api/org/easymock/TestSubject.html "http://easymock.org/api/org/easymock/TestSubject.html") 注释。

让我们重写之前的片段:

```java
@RunWith(EasyMockRunner.class)
public class BaeldungReaderAnnotatedTest {

    @Mock
    ArticleReader mockArticleReader;

    @TestSubject
    BaeldungReader baeldungReader = new BaeldungReader();

    @Test
    public void whenReadNext_thenNextArticleRead() {
        expect(mockArticleReader.next()).andReturn(null);
        replay(mockArticleReader);
        baeldungReader.readNext();
        verify(mockArticleReader);
    }
}
```

相当于`mock(…)`，mock 将被注入到用`@Mock`标注的字段中。这些模拟将被注入到用`@TestSubject`标注的类的字段中。

在上面的代码片段中，当调用`baeldungReader.readNext()`时，我们没有显式初始化`baeldungReader.`中的`articleReader`字段，我们可以调用隐式调用的`mockArticleReader`。

那是因为`mockArticleReader`被注入到`the articleReader`字段。

**注意，如果我们想要使用另一个测试运行器而不是`EasyMockRunner`，我们可以使用 JUnit 测试规则 [`EasyMockRule`](https://web.archive.org/web/20220703033325/http://easymock.org/api/org/easymock/EasyMockRule.html) :**

```java
public class BaeldungReaderAnnotatedWithRuleTest {

    @Rule
    public EasyMockRule mockRule = new EasyMockRule(this);

    //...

    @Test
    public void whenReadNext_thenNextArticleRead(){
        expect(mockArticleReader.next()).andReturn(null);
        replay(mockArticleReader);
        baeldungReader.readNext();
        verify(mockArticleReader);
    }

}
```

## 7。`EasyMockSupport`嘲弄着

有时我们需要在一个测试中引入多个模拟，我们必须手动重复:

```java
replay(A);
replay(B);
replay(C);
//...
verify(A);
verify(B);
verify(C);
```

这是丑陋的，我们需要一个优雅的解决方案。

幸运的是，我们在 EasyMock 中有一个类 **`EasyMockSupport`来帮助处理这个问题。它**有助于跟踪模拟，这样我们就可以在**像这样批量重放和验证它们:**

```java
//...
public class BaeldungReaderMockSupportTest extends EasyMockSupport{

    //...

    @Test
    public void whenReadAndWriteSequencially_thenWorks(){
        expect(mockArticleReader.next()).andReturn(null)
          .times(2).andThrow(new NoSuchElementException());
        expect(mockArticleWriter.write("title", "content"))
          .andReturn("BAEL-201801");
        replayAll();

        // execute read and write operations consecutively

        verifyAll();

        assertEquals(
          NoSuchElementException.class, 
          expectedException.getClass());
        assertEquals("BAEL-201801", articleId);
    }

}
```

这里我们嘲笑了`articleReader`和`articleWriter`。当将这些 mocks 设置为“重放”模式时，我们使用了`EasyMockSupport`提供的静态方法 [`replayAll()`](https://web.archive.org/web/20220703033325/http://easymock.org/api/org/easymock/EasyMockSupport.html#replayAll--) ，并使用 [`verifyAll()`](https://web.archive.org/web/20220703033325/http://easymock.org/api/org/easymock/EasyMockSupport.html#verifyAll--) 来批量验证它们的行为。

我们还在`expect`阶段介绍了 [`times(…)`](https://web.archive.org/web/20220703033325/http://easymock.org/api/org/easymock/IExpectationSetters.html#times-int-) 方法。它有助于指定我们期望该方法被调用的次数，这样我们可以避免引入重复的代码。

我们也可以通过委托使用`EasyMockSupport`:

```java
EasyMockSupport easyMockSupport = new EasyMockSupport();

@Test
public void whenReadAndWriteSequencially_thenWorks(){
    ArticleReader mockArticleReader = easyMockSupport
      .createMock(ArticleReader.class);
    IArticleWriter mockArticleWriter = easyMockSupport
      .createMock(IArticleWriter.class);
    BaeldungReader baeldungReader = new BaeldungReader(
      mockArticleReader, mockArticleWriter);

    expect(mockArticleReader.next()).andReturn(null);
    expect(mockArticleWriter.write("title", "content"))
      .andReturn("");
    easyMockSupport.replayAll();

    baeldungReader.readNext();
    baeldungReader.write("title", "content");

    easyMockSupport.verifyAll();
}
```

以前，我们使用静态方法或注释来创建和管理模拟。在幕后，这些静态和带注释的模拟是由一个全局`EasyMockSupport`实例控制的。

在这里，我们显式地实例化了它，并通过委托将所有这些模拟置于我们的控制之下。如果我们的测试代码与 EasyMock 有任何名称冲突，或者有任何类似的情况，这可能有助于避免混淆。

## 8。结论

在本文中，我们简要介绍了 EasyMock 的基本用法，关于如何生成模拟对象，记录和重放它们的行为，以及验证它们的行为是否正确。

如果您感兴趣，请查看本文中 EasyMock、Mocket 和 JMockit 的比较。

和往常一样，完整的实现可以在 Github 上找到[。](https://web.archive.org/web/20220703033325/https://github.com/eugenp/tutorials/tree/master/testing-modules/mocks)