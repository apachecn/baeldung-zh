# JSpec 指南

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/jspec>

## 1。概述

像 [JUnit](https://web.archive.org/web/20221208143856/http://junit.org/) 和 [TestNG](https://web.archive.org/web/20221208143856/http://testng.org/) 这样的测试运行器框架提供了一些基本的断言方法(`assertTrue`、`assertNotNull`等)。).

然后还有像 [Hamcrest](https://web.archive.org/web/20221208143856/http://hamcrest.org/) 、 [AssertJ](https://web.archive.org/web/20221208143856/https://joel-costigliola.github.io/assertj/) 、 [Truth](https://web.archive.org/web/20221208143856/https://github.com/google/truth) 这样的断言框架，提供了流畅丰富的断言方法，名字通常以`“assertThat”`开头。

JSpec 是另一个框架，它允许我们以更接近于我们用自然语言编写规范的方式编写流畅的断言，尽管与其他框架的方式略有不同。

在本文中，我们将学习如何使用 JSpec。我们将演示编写我们的规范所需的方法，以及在测试失败的情况下将打印的消息。

## 2。Maven 依赖关系

让我们导入`javalite-common`依赖项，它包含 JSpec:

```java
<dependency>
    <groupId>org.javalite</groupId>
    <artifactId>javalite-common</artifactId>
    <version>1.4.13</version>
</dependency>
```

最新版本请查看 [Maven 中心库](https://web.archive.org/web/20221208143856/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22org.javalite%22%20AND%20a%3A%22javalite-common%22)。

## 3。比较断言样式

与基于规则的典型断言方式不同，我们只是编写行为规范。让我们看一个在 JUnit、AssertJ 和 JSpec 中断言等式的简单例子。

在 JUnit 中，我们会写:

```java
assertEquals(1 + 1, 2);
```

在 AssertJ 中，我们会写:

```java
assertThat(1 + 1).isEqualTo(2);
```

下面是我们如何在 JSpec 中编写相同的测试:

```java
$(1 + 1).shouldEqual(2);
```

JSpec 使用了与 fluent 断言框架相同的风格，但是省略了前面的`assert` / `assertThat`关键字，而是使用了`should`。

以这种方式编写断言使得**更容易表示真正的规范**，促进了 TDD 和 BDD 的概念。

看看这个例子是如何非常接近我们对规范的自然书写:

```java
String message = "Welcome to JSpec demo";
the(message).shouldNotBe("empty");
the(message).shouldContain("JSpec");
```

## 4。规格结构

**规格说明由两部分组成:**一个期望创建器和一个期望方法。

### 4.1。期望创建者

期望创建者**使用这些静态导入的方法之一生成一个`Expectation`对象**:`a()`、`the()`、`it()`、`$():`

```java
$(1 + 2).shouldEqual(3);
a(1 + 2).shouldEqual(3);
the(1 + 2).shouldEqual(3);
it(1 + 2).shouldEqual(3);
```

所有这些方法本质上都是一样的——它们都只是为了提供各种方式来表达我们的规范。

唯一的区别是**`it()`方法是类型安全的**，只允许比较相同类型的对象:

```java
it(1 + 2).shouldEqual("3");
```

使用`it()`比较不同类型的对象会导致编译错误。

### 4.2。预期方法

规格说明的第二部分是期望方法，**讲述了所需的规格说明**，如`shouldEqual`、`shouldContain`。

当测试失败时，一个类型为`javalite.test.jspec.TestException`的异常显示一个表达性的消息。我们将在接下来的章节中看到这些失败消息的例子。

## 5。固有期望

JSpec 提供了几种期望方法。让我们来看看这些，包括一个显示 JSpec 在测试失败时生成的失败消息的场景。

### 5.1。平等期望

`**shouldEqual(), shouldBeEqual(), shouldNotBeEqual()**`

它们指定两个对象应该/不应该相等，使用`java.lang.Object.equals()`方法检查相等性:

```java
$(1 + 2).shouldEqual(3);
```

**故障场景:**

```java
$(1 + 2).shouldEqual(4);
```

会产生以下消息:

```java
Test object:java.lang.Integer == <3>
and expected java.lang.Integer == <4>
are not equal, but they should be.
```

### 5.2。布尔属性期望

`**shouldHave(), shouldNotHave()**`

我们使用这些方法来指定**对象的一个命名的`boolean`属性是否应该/不应该返回`true` :**

```java
Cage cage = new Cage();
cage.put(tomCat, boltDog);
the(cage).shouldHave("animals");
```

这要求`Cage`类包含一个带有签名的方法:

```java
boolean hasAnimals() {...}
```

**故障场景:**

```java
the(cage).shouldNotHave("animals");
```

会产生以下消息:

```java
Method: hasAnimals should return false, but returned true 
```

`**shouldBe(), shouldNotBe()**`

我们使用这些来指定被测试的对象应该/不应该是:

```java
the(cage).shouldNotBe("empty");
```

这要求`Cage`类包含一个带有签名`“boolean isEmpty()”.`的方法

**故障场景:**

```java
the(cage).shouldBe("empty");
```

会产生以下消息:

```java
Method: isEmpty should return true, but returned false
```

### 5.3。类型预期

`**shouldBeType(), shouldBeA()**`

我们可以使用这些方法来指定对象应该是特定的类型:

```java
cage.put(boltDog);
Animal releasedAnimal = cage.release(boltDog);
the(releasedAnimal).shouldBeA(Dog.class);
```

**故障场景:**

```java
the(releasedAnimal).shouldBeA(Cat.class);
```

会产生以下消息:

```java
class com.baeldung.jspec.Dog is not class com.baeldung.jspec.Cat
```

### 5.4。可空性预期

`**shouldBeNull(), shouldNotBeNull()**`

我们用这些来指定被测对象应该/不应该是`null`:

```java
cage.put(boltDog);
Animal releasedAnimal = cage.release(dogY);
the(releasedAnimal).shouldBeNull();
```

**故障场景:**

```java
the(releasedAnimal).shouldNotBeNull();
```

会产生以下消息:

```java
Object is null, while it is not expected
```

### 5.5。参考期望值

`**shouldBeTheSameAs(), shouldNotBeTheSameAs()**`

这些方法用于指定对象的引用应该与预期的引用相同:

```java
Dog firstDog = new Dog("Rex");
Dog secondDog = new Dog("Rex");
$(firstDog).shouldEqual(secondDog);
$(firstDog).shouldNotBeTheSameAs(secondDog);
```

**故障场景:**

```java
$(firstDog).shouldBeTheSameAs(secondDog);
```

会产生以下消息:

```java
references are not the same, but they should be
```

### 5.6。集合和字符串内容预期

`**shouldContain(), shouldNotContain()**`
我们用这些来指定被测试的`Collection`或`Map`应该/不应该包含给定的元素:

```java
cage.put(tomCat, felixCat);
the(cage.getAnimals()).shouldContain(tomCat);
the(cage.getAnimals()).shouldNotContain(boltDog);
```

**故障场景:**

```java
the(animals).shouldContain(boltDog);
```

会产生以下消息:

```java
tested value does not contain expected value: Dog [name=Bolt]
```

**我们也可以使用这些方法来指定`String`应该/不应该包含给定的子串:**

```java
$("Welcome to JSpec demo").shouldContain("JSpec");
```

虽然这看起来很奇怪，但是我们可以将这种行为扩展到其他对象类型，使用它们的`toString()`方法进行比较:

```java
cage.put(tomCat, felixCat);
the(cage).shouldContain(tomCat);
the(cage).shouldNotContain(boltDog);
```

澄清一下，`Cat`对象`tomCat`的`toString()`方法会产生:

```java
Cat [name=Tom]
```

它是`cage`对象的`toString()`输出的子串:

```java
Cage [animals=[Cat [name=Tom], Cat[name=Felix]]]
```

## 6。定制期望

除了内置的期望，JSpec 还允许我们编写定制的期望。

### 6.1。差异预期

我们可以写一个`DifferenceExpectation`来指定执行某个代码的返回值不应该等于某个特定的值。

在这个简单的例子中，我们确保运算(2 + 3)不会产生结果(4):

```java
expect(new DifferenceExpectation<Integer>(4) {
    @Override
    public Integer exec() {
        return 2 + 3;
    }
});
```

我们还可以使用它来确保执行一些代码会改变一些变量或方法的状态或值。

例如，当从包含两只动物的`Cage`中释放一只动物时，大小应该不同:

```java
cage.put(tomCat, boltDog);
expect(new DifferenceExpectation<Integer>(cage.size()) {
    @Override
    public Integer exec() {
        cage.release(tomCat);
        return cage.size();
    }
});
```

**故障场景:**

在这里，我们试图释放一种不存在于`Cage`中的动物:

```java
cage.release(felixCat);
```

大小不会改变，我们会得到以下消息:

```java
Objects: '2' and '2' are equal, but they should not be 
```

### 6.2。异常预期

我们可以写一个`ExceptionExpectation`来指定被测试的代码应该抛出一个`Exception`。

我们只是将预期的异常类型传递给构造函数，并将其作为泛型类型提供:

```java
expect(new ExceptionExpectation<ArithmeticException>(ArithmeticException.class) {
    @Override
    public void exec() throws ArithmeticException {
        System.out.println(1 / 0);
    }
});
```

**故障场景#1:**

```java
System.out.println(1 / 1);
```

因为这一行不会导致任何异常，所以执行它会产生以下消息:

```java
Expected exception: class java.lang.ArithmeticException, but instead got nothing
```

**故障场景#2:**

```java
Integer.parseInt("x");
```

这将导致与预期异常不同的异常:

```java
class java.lang.ArithmeticException,
but instead got: java.lang.NumberFormatException: For input string: "x"
```

## 7。结论

其他流畅的断言框架为集合断言、异常断言和 Java 8 集成提供了更好的方法，但 JSpec 提供了一种以规范形式编写断言的独特方式。

它有一个简单的 API，让我们像自然语言一样编写断言，并提供描述性的测试失败消息。

所有这些例子的完整源代码可以在 GitHub 的[中找到——在包`com.baeldung.jspec`中。](https://web.archive.org/web/20221208143856/https://github.com/eugenp/tutorials/tree/master/testing-modules/assertion-libraries)