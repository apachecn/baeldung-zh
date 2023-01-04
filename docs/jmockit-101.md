# JMockit 101

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/jmockit-101>

## 1。简介

通过这篇文章，我们将围绕嘲讽工具包 [JMockit](https://web.archive.org/web/20220812064337/https://jmockit.github.io/) 开始一个新的系列。

在第一部分中，我们将讨论什么是 JMockit，它的特性以及如何创建和使用 mocks。

后面的文章将重点关注并深入探讨它的功能。

## 2\. JMockit

### 2.1。简介

首先说一下 JMockit 是什么:一个在测试中嘲讽对象的 Java 框架(你可以用它来做 [JUnit](https://web.archive.org/web/20220812064337/http://junit.org/junit4/) 和 [TestNG](https://web.archive.org/web/20220812064337/http://testng.org/doc/index.html) 的)。

它使用 Java 的检测 API 在运行时修改类的字节码，以便动态地改变它们的行为。它的一些优点是它的可表达性和模仿静态和私有方法的现成能力。

也许您不熟悉 JMockit，但这肯定不是因为它是新的。JMockit 的开发始于 2006 年 6 月，它的第一个稳定版本发布于 2012 年 12 月，所以它已经存在了一段时间(在撰写本文时，当前版本是 1.24)。

### 2.2.Maven 依赖性

首先，我们需要将 [jmockit](https://web.archive.org/web/20220812064337/https://search.maven.org/search?q=a:jmockit%20AND%20g:org.jmockit) 依赖项添加到项目中:

```
<dependency> 
    <groupId>org.jmockit</groupId> 
    <artifactId>jmockit</artifactId> 
    <version>1.41</version>
</dependency>
```

### 2.3。JMockit 的可表达性

如前所述，JMockit 最强的一点是它的可表达性。为了创建模仿并定义它们的行为，您只需要直接定义它们，而不是从模仿 API 调用方法。

这意味着你不会做这样的事情:

```
API.expect(mockInstance.method()).andThenReturn(value).times(2);
```

相反，应该期待这样的事情:

```
new Expectation() {
    mockInstance.method(); 
    result = value; 
    times = 2;
}
```

看起来代码更多，但是你可以简单地把所有三行代码放在一行中。真正重要的是，你不会以一大串连锁的方法调用而告终。相反，您最终会得到一个定义，您希望 mock 在被调用时如何表现。

如果考虑到在`result = value`部分可以返回任何东西(固定值、动态生成的值、异常等)，JMockit 的表达能力就更加明显了。

### 2.4。记录-重放-验证模式

使用 JMockit 的测试分为三个不同的阶段:记录、重放和验证。

1.  在**记录**阶段，在测试准备期间和调用我们想要执行的方法之前，我们将为下一阶段使用的所有测试定义预期的行为。
2.  **replay** 阶段是被测代码被执行的阶段。现在将重放先前在先前阶段记录的被模仿的方法/构造函数的调用。
3.  最后，在 **verify** 阶段，我们将断言测试的结果是我们所期望的(并且模拟的行为和使用是根据记录阶段中定义的)。

对于一个代码示例，一个测试的线框如下所示:

```
@Test
public void testWireframe() {
   // preparation code not specific to JMockit, if any

   new Expectations() {{ 
       // define expected behaviour for mocks
   }};

   // execute code-under-test

   new Verifications() {{ 
       // verify mocks
   }};

   // assertions
}
```

## 3。创建模拟

### 3.1。JMockit 的注释

使用 JMockit 时，使用模拟的最简单方法是使用注释。有三个用于创建模拟(`@Mocked`、`@Injectable` 和`@Capturing`)，一个用于指定测试中的类(`@Tested`)。

当在一个字段上使用`@Mocked`注释时，它将创建该特定类的每个新对象的模拟实例。

另一方面，使用`@Injectable`注释，只会创建一个模拟实例。

最后一个注释`@Capturing`的行为与`@Mocked,`相似，但它将扩展到扩展或实现注释字段类型的每个子类。

### 3.2。向测试传递参数

使用 JMockit 时，可以将模拟作为测试参数传递。这对于为某个特定的测试创建模拟非常有用，比如某个复杂的模型对象需要某个测试的特定行为。大概是这样的:

```
@RunWith(JMockit.class)
public class TestPassingArguments {

   @Injectable
   private Foo mockForEveryTest;

   @Tested
   private Bar bar;

   @Test
   public void testExample(@Mocked Xyz mockForJustThisTest) {
       new Expectations() {{
           mockForEveryTest.someMethod("foo");
           mockForJustThisTest.someOtherMethod();
       }};

       bar.codeUnderTest();
   }
}
```

这种通过将模拟作为参数传递来创建模拟的方式，而不是必须调用一些 API 方法，再次向我们展示了我们从一开始就在谈论的可表达性。

### 3.3。完整示例

在本文的结尾，我们将包含一个使用 JMockit 的完整测试示例。

在这个例子中，我们将测试一个在`perform()`方法中使用`Collaborator`的`Performer`类。这个`perform()`方法接收一个`Model`对象作为参数，从这个参数中它将使用返回一个字符串的`getInfo()`，这个字符串将从`Collaborator`传递给`collaborate()` 方法，后者将为这个特定的测试返回`true`，这个值将从`Collaborator`传递给`receive()`方法。

因此，经过测试的类将如下所示:

```
public class Model {
    public String getInfo(){
        return "info";
    }
}

public class Collaborator {
    public boolean collaborate(String string){
        return false;
    }
    public void receive(boolean bool){
        // NOOP
    }
}

public class Performer {
    private Collaborator collaborator;

    public void perform(Model model) {
        boolean value = collaborator.collaborate(model.getInfo());
        collaborator.receive(value);
    }
}
```

测试的代码将会是这样的:

```
@RunWith(JMockit.class)
public class PerformerTest {

    @Injectable
    private Collaborator collaborator;

    @Tested
    private Performer performer;

    @Test
    public void testThePerformMethod(@Mocked Model model) {
        new Expectations() {{
    	    model.getInfo();result = "bar";
    	    collaborator.collaborate("bar"); result = true;
        }};
        performer.perform(model);
        new Verifications() {{
    	    collaborator.receive(true);
        }};
    }
}
```

## 4。结论

至此，我们将结束对 JMockit 的实用介绍。如果您想了解更多关于 JMockit 的知识，请关注以后的文章。

本教程的完整实现可以在 GitHub 项目中找到。

### 4.1。系列文章

该系列的所有文章:

*   [JMockit 101](/web/20220812064337/https://www.baeldung.com/jmockit-101)
*   [JMockit 指南-期望](/web/20220812064337/https://www.baeldung.com/jmockit-expectations)
*   [JMockit 高级用法](/web/20220812064337/https://www.baeldung.com/jmockit-advanced-usage)