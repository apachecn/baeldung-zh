# Hamcrest 定制匹配器

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/hamcrest-custom-matchers>

## 1。简介

除了内置的匹配器， **Hamcrest 还提供了创建自定义匹配器的支持。**

在本教程中，我们将仔细看看如何创建和使用它们。要先睹为快，请参考本文[。](/web/20221212104153/https://www.baeldung.com/java-junit-hamcrest-guide)

## 2。自定义匹配器设置

为了获得 Hamcrest，我们需要**将下面的 Maven 依赖项添加到我们的`pom.xml`** 中:

```java
<dependency>
    <groupId>org.hamcrest</groupId>
    <artifactId>java-hamcrest</artifactId>
    <version>2.0.0.0</version>
    <scope>test</scope>
</dependency>
```

最新的 Hamcrest 版本可以在 [Maven Central](https://web.archive.org/web/20221212104153/https://search.maven.org/classic/#search%7Cga%7C1%7Ca%3A%22java-hamcrest%22) 上找到。

## 3。`TypeSafeMatcher`介绍

在开始我们的例子之前，**理解类`TypeSafeMatcher`是很重要的。我们必须扩展这个类来创建我们自己的匹配器。**

`TypeSafeMatcher`是一个抽象类，所以所有子类都必须实现以下方法:

*   `matchesSafely(T t)`:包含我们的匹配逻辑
*   `describeTo(Description description)`:定制当我们的匹配逻辑不满足时，客户端将得到的消息

正如我们在第一个方法中看到的， **`TypeSafeMatcher`是参数化的，所以我们在使用它时必须声明一个类型。**那将是我们正在测试的物体的类型。

让我们通过在下一节中查看我们的第一个例子来更清楚地理解这一点。

## 4。创建`onlyDigits`匹配器

对于我们的第一个用例，**,我们将创建一个匹配器，如果某个`String` 只包含数字，它将返回 true。**

所以，`onlyDigits`应用到“123”应该返回`true` 而“`hello1`”和“`bye`”应该返回 false。

我们开始吧！

### 4.1。匹配器创建

从我们的匹配器开始，我们将创建一个扩展`TypeSafeMatcher`的类:

```java
public class IsOnlyDigits extends TypeSafeMatcher<String> {

    @Override
    protected boolean matchesSafely(String s) {
        // ...
    }

    @Override
    public void describeTo(Description description) {
        // ...
    }
}
```

请注意，由于我们要测试的对象是一个文本，我们用类`String.` 来参数化我们的子类`TypeSafeMatcher`

现在我们准备添加我们的实现:

```java
public class IsOnlyDigits extends TypeSafeMatcher<String> {

    @Override
    protected boolean matchesSafely(String s) {
        try {
            Integer.parseInt(s);
            return true;
        } catch (NumberFormatException nfe){
            return false;
        }
    }

    @Override
    public void describeTo(Description description) {
        description.appendText("only digits");
    }
}
```

正如我们所见，`matchesSafey` 试图将我们的输入`String` 解析成一个`Integer`。如果成功，则返回`true`。如果失败，它返回`false`。它成功地响应了我们的用例。

另一方面，`describeTo` 正在附上代表我们期望的文本。接下来，当我们使用匹配器时，我们将看到它是如何显示的。

我们只需要再做一件事来完成我们的匹配器:一个静态方法来访问它，所以它的行为和其他内置匹配器一样。

因此，我们将添加类似这样的内容:

```java
public static Matcher<String> onlyDigits() {
    return new IsOnlyDigits();
}
```

我们完事了。让我们在下一节看看如何使用这个匹配器。

### 4.2。匹配器用法

为了**使用我们全新的匹配器，我们将创建一个测试**:

```java
@Test
public void givenAString_whenIsOnlyDigits_thenCorrect() {
    String digits = "1234";

    assertThat(digits, onlyDigits());
}
```

仅此而已。这个测试将通过，因为输入`String`只包含数字。记住，为了使它更清晰一点，**我们可以使用匹配器`is` 作为任何其他匹配器**的包装器:

```java
assertThat(digits, is(onlyDigits()));
```

最后，如果我们运行相同的测试，但是输入“123ABC”，输出消息将是:

```java
java.lang.AssertionError: 
Expected: only digits
     but: was "123ABC"
```

**这是我们看到的添加到`describeTo` 方法中的文本。**正如我们可能已经注意到的，**对测试中的预期内容进行恰当的描述非常重要。**

## 5。`divisibleBy`

那么，如果我们想创建一个匹配器来定义一个数是否能被另一个数整除呢？对于这种情况，**我们必须在某个地方存储一个参数。**

让我们看看如何做到这一点:

```java
public class IsDivisibleBy extends TypeSafeMatcher<Integer> {

    private Integer divider;

    // constructors

    @Override
    protected boolean matchesSafely(Integer dividend) {
        if (divider == 0) {
            return false;
        }
        return ((dividend % divider) == 0);
    }

    @Override
    public void describeTo(Description description) {
        description.appendText("divisible by " + divider);
    }

    public static Matcher<Integer> divisibleBy(Integer divider) {
        return new IsDivisibleBy(divider);
    }
}
```

很简单，**我们只是在我们的类中添加了一个新属性，并在构造过程中赋予它**。然后，我们将它作为参数传递给静态方法:

```java
@Test
public void givenAnEvenInteger_whenDivisibleByTwo_thenCorrect() {
    Integer ten = 10;
    Integer two = 2;

    assertThat(ten,is(divisibleBy(two)));
}

@Test
public void givenAnOddInteger_whenNotDivisibleByTwo_thenCorrect() {
    Integer eleven = 11;
    Integer two = 2;

    assertThat(eleven,is(not(divisibleBy(two))));
}
```

就是这样！我们已经有了使用多个输入的匹配器！

## 6。结论

Hamcrest 提供的匹配器涵盖了开发人员在创建断言时通常必须处理的大多数用例。

此外，如果没有涵盖任何特定情况， **Hamcrest 还支持创建在特定场景下使用的自定义匹配器**——正如我们在这里探讨的**。**它们创建起来很简单，使用起来和库中包含的完全一样。

要获得这个例子的完整实现，请参考[GitHub 项目](https://web.archive.org/web/20221212104153/https://github.com/eugenp/tutorials/tree/master/testing-modules/hamcrest)。