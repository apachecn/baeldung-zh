# Vavr 验证 API 简介

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/vavr-validation-api>

## 1。概述

验证是 Java 应用程序中经常出现的任务，因此在验证库的开发中投入了大量的精力。

Vavr (原名 Javaslang)提供了一个成熟的[验证 API](https://web.archive.org/web/20220524003107/http://www.vavr.io/vavr-docs/#_validation) 。它允许我们通过使用对象函数式编程风格，以一种直接的方式验证数据。如果你想看看这个图书馆提供了什么，请随意查看[这篇文章](/web/20220524003107/https://www.baeldung.com/vavr)。

在本教程中，我们将深入了解该库的验证 API，并学习如何使用其最相关的方法。

## 2.`**Validation**`界面

Vavr 的验证接口是基于一个被称为[应用函子](https://web.archive.org/web/20220524003107/https://en.wikipedia.org/wiki/Applicative_functor)的函数式编程概念。它在累积结果的同时执行一系列函数，即使这些函数中的一些或全部在执行链中失败。

该库的应用函子建立在它的`Validation`接口的实现之上。该接口提供了累积验证错误和验证数据的方法，因此允许将它们作为一个批处理来处理。

## 3。验证用户输入

使用验证 API 来验证用户输入(例如，从 web 层收集的数据)是顺利的，因为它归结为创建自定义验证类，该自定义验证类在累积结果错误(如果有的话)的同时验证数据。

让我们验证通过登录表单提交的用户名和电子邮件。首先，我们需要将 [Vavr 的 Maven 神器](https://web.archive.org/web/20220524003107/https://search.maven.org/classic/#search%7Cga%7C1%7Cvavr)包含到`pom.xml`文件中:

```java
<dependency>
    <groupId>io.vavr</groupId>
    <artifactId>vavr</artifactId>
    <version>0.9.0</version>
</dependency>
```

接下来，让我们创建一个对用户对象建模的域类:

```java
public class User {
    private String name;
    private String email;

    // standard constructors, setters and getters, toString
} 
```

最后，让我们定义我们的自定义验证器:

```java
public class UserValidator {
    private static final String NAME_PATTERN = ...
    private static final String NAME_ERROR = ...
    private static final String EMAIL_PATTERN = ...
    private static final String EMAIL_ERROR = ...

    public Validation<Seq<String>, User> validateUser(
      String name, String email) {
        return Validation
          .combine(
            validateField(name, NAME_PATTERN, NAME_ERROR),
            validateField(email, EMAIL_PATTERN, EMAIL_ERROR))
          .ap(User::new);
    }

    private Validation<String, String> validateField
      (String field, String pattern, String error) {

        return CharSeq.of(field)
          .replaceAll(pattern, "")
          .transform(seq -> seq.isEmpty() 
            ? Validation.valid(field) 
            : Validation.invalid(error));		
    }
}
```

`UserValidator`类使用`validateField()`方法分别验证提供的名称和电子邮件。在这种情况下，该方法执行典型的基于正则表达式的模式匹配。

这个例子的本质是使用了`valid()`、`invalid()`和`combine()`方法。

## 4。`valid(),` `invalid()`和`combine()`方法

如果提供的名称和电子邮件与给定的正则表达式匹配，`validateField()`方法调用`valid()`。这个方法返回一个`Validation.Valid`的实例。相反，如果值无效，counter-part `invalid()`方法返回一个`Validation.Invalid`的实例。

这个简单的机制，基于根据验证结果创建不同的`Validation`实例，应该至少给我们一个关于如何处理结果的基本概念(更多信息在第 5 节)。

验证过程最相关的方面是`combine()`方法。在内部，这个方法使用了`Validation.Builder`类，它允许组合多达 8 个不同的`Validation`实例，这些实例可以用不同的方法计算:

```java
static <E, T1, T2> Builder<E, T1, T2> combine(
  Validation<E, T1> validation1, Validation<E, T2> validation2) {
    Objects.requireNonNull(validation1, "validation1 is null");
    Objects.requireNonNull(validation2, "validation2 is null");
    return new Builder<>(validation1, validation2);
}
```

最简单的`Validation.Builder`类有两个验证实例:

```java
final class Builder<E, T1, T2> {

    private Validation<E, T1> v1;
    private Validation<E, T2> v2;

    // standard constructors

    public <R> Validation<Seq<E>, R> ap(Function2<T1, T2, R> f) {
        return v2.ap(v1.ap(Validation.valid(f.curried())));
    }

    public <T3> Builder3<E, T1, T2, T3> combine(
      Validation<E, T3> v3) {
        return new Builder3<>(v1, v2, v3);
    }
}
```

`Validation.Builder,`与`ap(Function)`方法一起，返回一个带有验证结果的单一结果。如果所有结果都有效，`ap(Function)`方法将结果映射到一个值上。这个值通过使用签名中指定的函数存储在一个`Valid`实例中。

在我们的示例中，如果提供的名称和电子邮件是有效的，就会创建一个新的`User`对象。当然，有可能用一个有效的结果做一些完全不同的事情，例如，把它存储到一个数据库中，通过电子邮件发送等等。

## 5。处理验证结果

实现不同的机制来处理验证结果是非常容易的。但是我们首先如何验证数据呢？为此，我们使用了`UserValidator`类:

```java
UserValidator userValidator = new UserValidator(); 
Validation<Seq<String>, User> validation = userValidator
  .validateUser("John", "[[email protected]](/web/20220524003107/https://www.baeldung.com/cdn-cgi/l/email-protection)");
```

一旦获得了`Validation`的实例，我们就可以利用验证 API 的灵活性，以几种方式处理结果。

让我们详细说明一下最常见的方法。

### 5.1。`Valid`和`Invalid`实例

这是迄今为止最简单的方法。它包括用`Valid`和`Invalid`实例检查验证结果:

```java
@Test
public void 
  givenInvalidUserParams_whenValidated_thenInvalidInstance() {
    assertThat(
      userValidator.validateUser(" ", "no-email"), 
      instanceOf(Invalid.class));
}

@Test
public void 
  givenValidUserParams_whenValidated_thenValidInstance() {
    assertThat(
      userValidator.validateUser("John", "[[email protected]](/web/20220524003107/https://www.baeldung.com/cdn-cgi/l/email-protection)"), 
      instanceOf(Valid.class));
}
```

我们应该更进一步，使用`isValid()`和`isInvalid()`方法，而不是用`Valid`和`Invalid`实例来检查结果的有效性。

### 5.2。`isValid()`和`isInvalid()`API

使用串联`isValid()` / `isInvalid()`类似于前面的方法，不同之处在于这些方法根据验证结果返回`true`或`false`:

```java
@Test
public void 
  givenInvalidUserParams_whenValidated_thenIsInvalidIsTrue() {
    assertTrue(userValidator
      .validateUser("John", "no-email")
      .isInvalid());
}

@Test
public void 
  givenValidUserParams_whenValidated_thenIsValidMethodIsTrue() {
    assertTrue(userValidator
      .validateUser("John", "[[email protected]](/web/20220524003107/https://www.baeldung.com/cdn-cgi/l/email-protection)")
      .isValid());
}
```

`Invalid`实例包含所有的验证错误。它们可以通过`getError()`方法获取:

```java
@Test
public void 
  givenInValidUserParams_withGetErrorMethod_thenGetErrorMessages() {
    assertEquals(
      "Name contains invalid characters, Email must be a well-formed email address", 
      userValidator.validateUser("John", "no-email")
        .getError()
        .intersperse(", ")
        .fold("", String::concat));
 }
```

相反，如果结果有效，可以用`get()`方法获取一个`User`实例:

```java
@Test
public void 
  givenValidUserParams_withGetMethod_thenGetUserInstance() {
    assertThat(userValidator.validateUser("John", "[[email protected]](/web/20220524003107/https://www.baeldung.com/cdn-cgi/l/email-protection)")
      .get(), instanceOf(User.class));
 }
```

这种方法如预期的那样工作，但是代码看起来仍然非常冗长。我们可以使用`toEither()`方法进一步压缩它。

### 5.3。`toEither()` API

`toEither()`方法构造了`Either`接口的`Left`和`Right`实例。这个补充接口有几个方便的方法，可以用来缩短验证结果的处理。

如果结果有效，结果将存储在`Right`实例中。在我们的例子中，这相当于一个有效的`User`对象。相反，如果结果无效，错误将存储在`Left`实例中:

```java
@Test
public void 
  givenValidUserParams_withtoEitherMethod_thenRightInstance() {
    assertThat(userValidator.validateUser("John", "[[email protected]](/web/20220524003107/https://www.baeldung.com/cdn-cgi/l/email-protection)")
      .toEither(), instanceOf(Right.class));
}
```

代码现在看起来更加简洁和流畅。但是我们还没完。`Validation`接口提供了`fold()`方法，该方法应用一个适用于有效结果的自定义函数和另一个适用于无效结果的自定义函数。

### 5.4。`fold()` API

让我们看看如何使用`fold()`方法来处理验证结果:

```java
@Test
public void 
  givenValidUserParams_withFoldMethod_thenEqualstoParamsLength() {
    assertEquals(2, (int) userValidator.validateUser(" ", " ")
      .fold(Seq::length, User::hashCode));
}
```

使用`fold()`将验证结果的处理减少到只有一行代码。

值得强调的是，作为参数传递给方法的函数的返回类型必须相同。此外，函数必须得到验证类中定义的类型参数的支持，即`Seq<String>`和`User`。

## 6。结论

在本文中，我们深入探讨了 Vavr 的验证 API，并学习了如何使用它的一些最相关的方法。如需完整列表，请查看[官方文档 API](https://web.archive.org/web/20220524003107/https://www.javadoc.io/doc/io.vavr/vavr/0.9.0) 。

Vavr 的验证控件提供了一个非常吸引人的替代方案，可以替代更传统的 [Java Beans 验证、](https://web.archive.org/web/20220524003107/http://beanvalidation.org/)如 [Hibernate 验证器](https://web.archive.org/web/20220524003107/http://hibernate.org/validator/)。

和往常一样，文章中显示的所有例子都可以在 GitHub 上找到[。](https://web.archive.org/web/20220524003107/https://github.com/eugenp/tutorials/tree/master/vavr)