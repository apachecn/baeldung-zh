# JMockit 高级用法

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/jmockit-advanced-usage>

## 1。简介

在本文中，我们将超越 JMockit 基础知识，开始研究一些高级场景，例如:

*   伪造(或`MockUp` API)
*   `Deencapsulation`实用程序类
*   如何仅使用一个模拟来模拟多个接口
*   如何重用期望和验证

如果您想了解 JMockit 的基础知识，请查看本系列的其他文章。你可以在页面底部找到相关链接。

## 2.Maven 依赖性

首先，我们需要将 [jmockit](https://web.archive.org/web/20220807183407/https://search.maven.org/search?q=a:jmockit%20AND%20g:org.jmockit) 依赖项添加到项目中:

```java
<dependency> 
    <groupId>org.jmockit</groupId> 
    <artifactId>jmockit</artifactId> 
    <version>1.41</version>
</dependency> 
```

接下来，我们将继续举例。

## 3。私有方法/内部类模仿

嘲笑和测试私有方法或内部类通常不被认为是好的做法。

其背后的原因是，如果它们是私有的，就不应该直接测试它们，因为它们是类中最内部的部分，但有时仍然需要这样做，尤其是在处理遗留代码时。

使用 JMockit，您有两个选项来处理这些问题:

*   用于改变实际实现的`MockUp` API(对于第二种情况)
*   `Deencapsulation`实用程序类，直接调用任何方法(第一种情况)

所有下面的例子都将在下面的类中完成，我们假设它们运行在一个与第一个具有相同配置的测试类上(以避免重复代码):

```java
public class AdvancedCollaborator {
    int i;
    private int privateField = 5;

    // default constructor omitted 

    public AdvancedCollaborator(String string) throws Exception{
        i = string.length();
    }

    public String methodThatCallsPrivateMethod(int i) {
        return privateMethod() + i;
    }
    public int methodThatReturnsThePrivateField() {
        return privateField;
    }
    private String privateMethod() {
        return "default:";
    }

    class InnerAdvancedCollaborator {...}
}
```

### 3.1。`MockUp`作伪与

JMockit 的 Mockup API 为创建假实现或`mock-ups`提供了支持。通常情况下，`mock-up`的目标是要伪造的类中的一些方法和/或构造函数，而其他大多数方法和构造函数保持不变。这允许完全重写一个类，因此任何方法或构造函数(带有任何访问修饰符)都可以成为目标。

让我们看看如何使用模型的 API 重新定义`privateMethod()`:

```java
@RunWith(JMockit.class)
public class AdvancedCollaboratorTest {

    @Tested
    private AdvancedCollaborator mock;

    @Test
    public void testToMockUpPrivateMethod() {
        new MockUp<AdvancedCollaborator>() {
            @Mock
            private String privateMethod() {
                return "mocked: ";
            }
        };
        String res = mock.methodThatCallsPrivateMethod(1);
        assertEquals("mocked: 1", res);
    }
}
```

在这个例子中，我们使用带有匹配签名的方法上的`@Mock`注释为`AdvancedCollaborator`类定义了一个新的`MockUp`。在此之后，对该方法的调用将委托给我们模仿的方法。

我们也可以用它来`mock-up`构造一个需要特定参数或配置的类，以简化测试:

```java
@Test
public void testToMockUpDifficultConstructor() throws Exception{
    new MockUp<AdvancedCollaborator>() {
        @Mock
        public void $init(Invocation invocation, String string) {
            ((AdvancedCollaborator)invocation.getInvokedInstance()).i = 1;
        }
    };
    AdvancedCollaborator coll = new AdvancedCollaborator(null);
    assertEquals(1, coll.i);
}
```

在这个例子中，我们可以看到，对于构造函数模拟，您需要模拟`$init`方法。您可以传递一个额外的类型为`Invocation,`的参数，通过这个参数您可以访问关于被模仿方法的调用的信息，包括正在执行调用的实例。

### 3.2。使用`Deencapsulation`类

JMockit 包含一个测试实用程序类:`Deencapsulation`。顾名思义，它用于解封装对象的状态，使用它，您可以通过访问字段和方法来简化测试，否则无法访问这些字段和方法。

您可以调用一个方法:

```java
@Test
public void testToCallPrivateMethodsDirectly(){
    Object value = Deencapsulation.invoke(mock, "privateMethod");
    assertEquals("default:", value);
}
```

您还可以设置字段:

```java
@Test
public void testToSetPrivateFieldDirectly(){
    Deencapsulation.setField(mock, "privateField", 10);
    assertEquals(10, mock.methodThatReturnsThePrivateField());
}
```

并获取字段:

```java
@Test
public void testToGetPrivateFieldDirectly(){
    int value = Deencapsulation.getField(mock, "privateField");
    assertEquals(5, value);
}
```

并创建类的新实例:

```java
@Test
public void testToCreateNewInstanceDirectly(){
    AdvancedCollaborator coll = Deencapsulation
      .newInstance(AdvancedCollaborator.class, "foo");
    assertEquals(3, coll.i);
}
```

甚至内部类的新实例:

```java
@Test
public void testToCreateNewInnerClassInstanceDirectly(){
    InnerCollaborator inner = Deencapsulation
      .newInnerInstance(InnerCollaborator.class, mock);
    assertNotNull(inner);
}
```

如您所见，`Deencapsulation`类在测试气密类时非常有用。一个例子是设置在私有字段上使用`@Autowired`注释并且没有设置器的类的依赖关系，或者单元测试内部类而不必依赖其容器类的公共接口。

## 4。在同一个模拟中模拟多个接口

让我们假设您想要测试一个类——尚未实现——但是您肯定知道它将实现几个接口。

通常，您不能在实现之前测试这个类，但是使用 JMockit，您可以通过使用一个模拟对象模拟多个接口来预先准备测试。

这可以通过使用泛型并定义一个扩展多个接口的类型来实现。这个泛型类型既可以为整个测试类定义，也可以只为一个测试方法定义。

例如，我们将通过两种方式`:`为接口`List` 和`Comparable` 创建一个模拟

```java
@RunWith(JMockit.class)
public class AdvancedCollaboratorTest<MultiMock
  extends List<String> & Comparable<List<String>>> {

    @Mocked
    private MultiMock multiMock;

    @Test
    public void testOnClass() {
        new Expectations() {{
            multiMock.get(5); result = "foo";
            multiMock.compareTo((List<String>) any); result = 0;
        }};
        assertEquals("foo", multiMock.get(5));
        assertEquals(0, multiMock.compareTo(new ArrayList<>()));
    }

    @Test
    public <M extends List<String> & Comparable<List<String>>>
      void testOnMethod(@Mocked M mock) {
        new Expectations() {{
            mock.get(5); result = "foo";
            mock.compareTo((List<String>) any); result = 0; 
        }};
        assertEquals("foo", mock.get(5));
        assertEquals(0, mock.compareTo(new ArrayList<>()));
    }
}
```

正如您在第 2 行中看到的，我们可以通过在类名上使用泛型来为整个测试定义一个新的测试类型。这样，`MultiMock`将作为一个类型可用，您将能够使用 JMockit 的任何注释为它创建模拟。

在第 7 行到第 18 行中，我们可以看到一个为整个测试类定义的多类模拟的例子。

如果您只需要一个测试的多接口模拟，您可以通过在方法签名上定义泛型类型并传递新泛型的新模拟作为测试方法参数来实现。在第 20 到 32 行中，我们可以看到一个这样做的例子，它针对的是与前面测试中相同的测试行为。

## 5。重用期望和验证

最后，当测试类时，您可能会遇到一遍又一遍地重复相同的`Expectations`和/或`Verifications`的情况。为了缓解这种情况，您可以轻松地重用这两者。

我们将通过一个例子来解释它(我们使用来自我们的 [JMockit 101](/web/20220807183407/https://www.baeldung.com/jmockit-101) 文章中的类`Model, Collaborator`和`Performer`):

```java
@RunWith(JMockit.class)
public class ReusingTest {

    @Injectable
    private Collaborator collaborator;

    @Mocked
    private Model model;

    @Tested
    private Performer performer;

    @Before
    public void setup(){
        new Expectations(){{
           model.getInfo(); result = "foo"; minTimes = 0;
           collaborator.collaborate("foo"); result = true; minTimes = 0; 
        }};
    }

    @Test
    public void testWithSetup() {
        performer.perform(model);
        verifyTrueCalls(1);
    }

    protected void verifyTrueCalls(int calls){
        new Verifications(){{
           collaborator.receive(true); times = calls; 
        }};
    }

    final class TrueCallsVerification extends Verifications{
        public TrueCallsVerification(int calls){
            collaborator.receive(true); times = calls; 
        }
    }

    @Test
    public void testWithFinalClass() {
        performer.perform(model);
        new TrueCallsVerification(1);
    }
}
```

在这个例子中，你可以在第 15 行到第 18 行看到我们为每个测试准备了一个期望，这样`model.getInfo()`总是返回`“foo”`，而`collaborator.collaborate` `()`总是期望`“foo”`作为参数并返回`true`。我们使用了`minTimes = 0`语句，这样在测试中没有实际使用它们时就不会出现故障。

此外，当传递的参数是`true`时，我们创建了方法`verifyTrueCalls(int)`来简化对`collaborator.receive(boolean)`方法的验证。

最后，您还可以创建新类型的特定期望和验证，只需扩展任何一个`Expectations`或`Verifications`类。然后，如果您需要配置行为并在测试中创建所述类型的新实例，您可以定义一个构造函数，就像我们在第 33 行到第 43 行中所做的那样。

## 6。结论

在 JMockit 系列的这一期中，我们讨论了几个高级主题，这些主题肯定会对您的日常模拟和测试有所帮助。

我们可能会发表更多关于 JMockit 的文章，请继续关注以了解更多信息。

和往常一样，本教程的完整实现可以在 GitHub 上找到。

### 6.1。系列文章

该系列的所有文章:

*   [JMockit 101](/web/20220807183407/https://www.baeldung.com/jmockit-101)
*   [JMockit 期望指南](/web/20220807183407/https://www.baeldung.com/jmockit-expectations)
*   [JMockit 高级用法](/web/20220807183407/https://www.baeldung.com/jmockit-advanced-usage)