# 用 JUnit 测试抽象类

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/junit-test-abstract-class>

## 1。概述

在本教程中，我们将分析用非抽象方法对抽象类进行单元测试的各种用例以及可能的替代解决方案。

注意**测试抽象类应该总是通过具体实现的公共 API**，所以不要应用下面的技术，除非你确定你在做什么。

## 2。Maven 依赖关系

让我们从 Maven 依赖项开始:

```java
<dependency>
    <groupId>org.junit.jupiter</groupId>
    <artifactId>junit-jupiter-engine</artifactId>
    <version>5.8.1</version>
    <scope>test</scope>
</dependency>
<dependency>
    <groupId>org.mockito</groupId>
    <artifactId>mockito-core</artifactId>
    <version>2.8.9</version>
    <scope>test</scope>
</dependency>
<dependency>
    <groupId>org.powermock</groupId>
    <artifactId>powermock-module-junit4</artifactId>
    <version>1.7.4</version>
    <scope>test</scope>
    <exclusions>
        <exclusion>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
        </exclusion>
    </exclusions>
</dependency>
<dependency>
    <groupId>org.powermock</groupId>
    <artifactId>powermock-api-mockito2</artifactId>
    <version>1.7.4</version>
    <scope>test</scope>
</dependency>
```

你可以在 [Maven Central](https://web.archive.org/web/20220804203719/https://search.maven.org/classic/#search%7Cga%7C1%7C(g%3A%22org.junit.jupiter%22%20AND%20a%3A%22junit-jupiter-engine%22)%20OR%20(g%3A%22org.mockito%22%20AND%20a%3A%22mockito-core%22)%20OR%20(g%3A%22org.powermock%22%20AND%20a%3A%22powermock-module-junit4%22)%20OR%20(g%3A%22org.powermock%22%20AND%20a%3A%22powermock-api-mockito2%22)) 上找到这些库的最新版本。

Junit5 不完全支持 Powermock。此外，`powermock-module-junit4`仅用于第 5 节中的一个示例。

## 3。独立的非抽象方法

让我们考虑这样一种情况，我们有一个带有公共非抽象方法的抽象类:

```java
public abstract class AbstractIndependent {
    public abstract int abstractFunc();

    public String defaultImpl() {
        return "DEFAULT-1";
    }
}
```

我们想要测试方法`defaultImpl()`，我们有两个可能的解决方案——使用一个具体的类，或者使用 Mockito。

### 3.1。使用具体的类

创建一个扩展`AbstractIndependent `类的具体类，并用它来测试方法:

```java
public class ConcreteImpl extends AbstractIndependent {

    @Override
    public int abstractFunc() {
        return 4;
    }
}
```

```java
@Test
public void givenNonAbstractMethod_whenConcreteImpl_testCorrectBehaviour() {
    ConcreteImpl conClass = new ConcreteImpl();
    String actual = conClass.defaultImpl();

    assertEquals("DEFAULT-1", actual);
}
```

这种解决方案的缺点是需要用所有抽象方法的虚拟实现来创建具体的类。

### 3.2。使用 Mockito

或者，我们可以使用`Mockito `来创建一个模拟:

```java
@Test
public void givenNonAbstractMethod_whenMockitoMock_testCorrectBehaviour() {
    AbstractIndependent absCls = Mockito.mock(
      AbstractIndependent.class, 
      Mockito.CALLS_REAL_METHODS);

    assertEquals("DEFAULT-1", absCls.defaultImpl());
}
```

这里最重要的部分是在使用`Mockito.CALLS_REAL_METHODS`调用方法时，准备使用真正的代码。

## 4。从非抽象方法调用的抽象方法

在这种情况下，非抽象方法定义了全局执行流，而抽象方法可以根据用例以不同的方式编写:

```java
public abstract class AbstractMethodCalling {

    public abstract String abstractFunc();

    public String defaultImpl() {
        String res = abstractFunc();
        return (res == null) ? "Default" : (res + " Default");
    }
}
```

为了测试这段代码，我们可以使用与前面相同的两种方法——要么创建一个具体的类，要么使用 Mockito 创建一个 mock:

```java
@Test
public void givenDefaultImpl_whenMockAbstractFunc_thenExpectedBehaviour() {
    AbstractMethodCalling cls = Mockito.mock(AbstractMethodCalling.class);
    Mockito.when(cls.abstractFunc())
      .thenReturn("Abstract");
    Mockito.doCallRealMethod()
      .when(cls)
      .defaultImpl();

    assertEquals("Abstract Default", cls.defaultImpl());
}
```

这里，`abstractFunc()`用我们更喜欢的测试返回值填充。这意味着当我们调用非抽象方法`defaultImpl()`时，它将使用这个存根。

## 5。带有测试障碍的非抽象方法

在某些场景中，我们想要测试的方法调用一个私有方法，该方法包含一个测试障碍。

在测试目标方法之前，我们需要绕过阻碍测试的方法:

```java
public abstract class AbstractPrivateMethods {

    public abstract int abstractFunc();

    public String defaultImpl() {
        return getCurrentDateTime() + "DEFAULT-1";
    }

    private String getCurrentDateTime() {
        return LocalDateTime.now().toString();
    }
}
```

在这个例子中， *defaultImpl()* 方法调用私有方法 *getCurrentDateTime()* 。这个私有方法在运行时获取当前时间，这在我们的单元测试中应该避免。

现在，为了模仿这个私有方法的标准行为，我们甚至不能使用`Mockito`，因为它不能控制私有方法。

相反，我们需要使用 [PowerMock](/web/20220804203719/https://www.baeldung.com/intro-to-powermock) ( **n** **注意，这个例子只适用于 JUnit 4，因为 JUnit 5** 不支持这种依赖关系):

```java
@RunWith(PowerMockRunner.class)
@PrepareForTest(AbstractPrivateMethods.class)
public class AbstractPrivateMethodsUnitTest {

    @Test
    public void whenMockPrivateMethod_thenVerifyBehaviour() {
        AbstractPrivateMethods mockClass = PowerMockito.mock(AbstractPrivateMethods.class);
        PowerMockito.doCallRealMethod()
          .when(mockClass)
          .defaultImpl();
        String dateTime = LocalDateTime.now().toString();
        PowerMockito.doReturn(dateTime).when(mockClass, "getCurrentDateTime");
        String actual = mockClass.defaultImpl();

        assertEquals(dateTime + "DEFAULT-1", actual);
    }
}
```

本例中的重要信息:

*   `@RunWith `将 PowerMock 定义为测试的运行程序
*   `@PrepareForTest(class) `告诉 PowerMock 为以后的处理准备类

有趣的是，我们要求`PowerMock`存根私有方法`getCurrentDateTime(). ` PowerMock 将使用反射来找到它，因为它不能从外部访问。

所以`, `当我们调用`defaultImpl()`时，为私有方法创建的存根将被调用，而不是实际的方法。

## 6。访问实例字段的非抽象方法

抽象类可以有一个用类字段实现的内部状态。字段的值可能会对被测试的方法产生重大影响。

如果一个字段是公共的或受保护的，我们可以很容易地从测试方法中访问它。

但如果是私人的，我们就得用`PowerMockito`:

```java
public abstract class AbstractInstanceFields {
    protected int count;
    private boolean active = false;

    public abstract int abstractFunc();

    public String testFunc() {
        if (count > 5) {
            return "Overflow";
        } 
        return active ? "Added" : "Blocked";
    }
}
```

这里，`testFunc()`方法在返回之前使用实例级字段`count`和`active `。

当测试`testFunc()`时，我们可以通过访问使用`Mockito. `创建的实例来改变`count`字段的值

另一方面，为了测试私有`active`字段的行为，我们将再次使用`PowerMockito`和它的`Whitebox`类:

```java
@Test
public void whenPowerMockitoAndActiveFieldTrue_thenCorrectBehaviour() {
    AbstractInstanceFields instClass = PowerMockito.mock(AbstractInstanceFields.class);
    PowerMockito.doCallRealMethod()
      .when(instClass)
      .testFunc();
    Whitebox.setInternalState(instClass, "active", true);

    assertEquals("Added", instClass.testFunc());
}
```

我们使用`PowerMockito.mock()`创建一个存根类，并且使用`Whitebox`类来控制对象的内部状态。

`active `字段的值被更改为`true`。

## 7。结论

在本教程中，我们已经看到了涵盖许多用例的多个例子。根据所遵循的设计，我们可以在更多的场景中使用抽象类。

此外，为抽象类方法编写单元测试与为普通类和方法编写单元测试一样重要。我们可以使用不同的技术或者不同的测试支持库来测试它们。

完整的源代码可以在 GitHub 的[上找到。](https://web.archive.org/web/20220804203719/https://github.com/eugenp/tutorials/tree/master/testing-modules/junit-5)