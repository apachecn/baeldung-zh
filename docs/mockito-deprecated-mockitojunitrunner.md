# 警告:“不推荐使用类型 MockitoJUnitRunner”

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/mockito-deprecated-mockitojunitrunner>

## 1.**简介**

在这个快速教程中，**我们将看看在使用流行的测试框架 [Mockito](https://web.archive.org/web/20220524130930/https://site.mockito.org/) 时可能会看到的一个警告。**

也就是说，引用不赞成使用的`MockitoJUnitRunner`类。我们将了解为什么会出现这个警告以及如何处理它。

最后，让我们提醒一下，我们可以使用`MockitoJUnitRunner`来指示 Mockito 初始化用`@Mock `或`@Spy, `以及其他 Mockito 注释标注的 test-doubles。

要了解更多关于使用 Mockito 进行测试的信息，请点击查看我们的 [Mockito 系列。](/web/20220524130930/https://www.baeldung.com/tag/mockito/)

## 2.**为什么会显示此警告**

如果我们使用的是 2 . 2 . 20(2016 年 11 月)之前的 Mockito 版本，则会出现此弃用警告。

让我们简单地回顾一下它背后的历史。在 Mockito 的早期版本中，如果我们想要使用 Mockito JUnit Runner，我们需要导入的包是:

```java
import org.mockito.runners.MockitoJUnitRunner;
```

从版本 2.2.20 开始，JUnit 相关的类被重新组织到一个特定的 JUnit 包中。我们可以在这里找到这个包:

```java
import org.mockito.junit.MockitoJUnitRunner;
```

因此，最初的`org.mockito.runners.MockitoJUnitRunner`现在已被废弃。这个类的逻辑现在属于`org.mockito.junit.runners.MockitoJUnitRunner`。

虽然删除警告不是强制性的，但建议这样做。Mockito 3 会移除这个类。

## 3.**解决方案**

在本节中，我们将解释解决这一弃用警告的三种不同解决方案:

*   更新以使用正确的导入
*   使用`MockitoAnnotations`初始化字段
*   使用`MockitoRule`

### 3.1.**更新导入**

让我们从最简单的解决方案开始，简单地**改变包导入语句**:

```java
import org.mockito.junit.MockitoJUnitRunner;

@RunWith(MockitoJUnitRunner.class)
public class ExampleTest {
    //...
}
```

仅此而已！这种改变应该很容易实现。

### 3.2.**使用`MockitoAnnotations`** 初始化字段

在下一个例子中，**我们将使用`MockitoAnnotations`类**以不同的方式初始化我们的模拟:

```java
import org.junit.Before;
import org.mockito.MockitoAnnotations;

public class ExampleTest {

    @Before 
    public void initMocks() {
        MockitoAnnotations.initMocks(this);
    }

    //...
}
```

首先，我们移除了对`MockitoJUnitRunner.`的引用，我们调用了`MockitoAnnotations`类的静态`initMocks()`方法。

我们在 test 的类的 JUnit `@Before`方法中这样做。在执行每个测试之前，用 Mockito 注释初始化所有字段。

### 3.3.**使用`MockitoRule`**

然而，正如我们已经提到的，`MockitoJUnitRunner`绝不是强制性的。在最后一个例子中，**我们将看看使用** `**MockitoRule**:`让`@Mock`工作的另一种方式

```java
import org.junit.Rule;
import org.mockito.junit.MockitoJUnit;
import org.mockito.junit.MockitoRule;

public class ExampleTest {

    @Rule
    public MockitoRule rule = MockitoJUnit.rule();

    //...
}
```

最后，在这个例子中，JUnit 规则初始化任何用`@Mock`注释的模拟。

因此，这意味着没有必要明确使用`MockitoAnnotations#initMocks(Object)`或`@RunWith(MockitoJUnitRunner.class)`。

## 4.**结论**

总而言之，在这篇短文中，我们看到了几个关于如何修复`MockitoJUnitRunner`类弃用警告的选项。