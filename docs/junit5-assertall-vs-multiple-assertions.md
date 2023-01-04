# assertAll()与 JUnit5 中的多个断言

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/junit5-assertall-vs-multiple-assertions>

## 1.概观

当编写单元测试时，我们有时会提供输出的多个断言。当第一个断言失败时，测试停止。这意味着我们不知道后面的断言是通过了还是失败了，这会增加调试时间。

我们可以通过将多个断言打包成一个动作来解决这个问题。

在这个简短的教程中，我们将学习如何使用在 [JUnit5](/web/20221031071901/https://www.baeldung.com/junit-5) 中引入的`assertAll()`方法，并看看它与使用多个断言有何不同。

## 2.模型

我们将使用一个`User`类来帮助我们的例子:

```java
public class User {
    String username;
    String email;
    boolean activated;
    //constructors
   //getters and setters
}
```

## 3.使用多个断言

让我们从一个所有断言都失败的例子开始:

```java
User user = new User("baeldung", "[[email protected]](/web/20221031071901/https://www.baeldung.com/cdn-cgi/l/email-protection)", false);
assertEquals("admin", user.getUsername(), "Username should be admin");
assertEquals("[[email protected]](/web/20221031071901/https://www.baeldung.com/cdn-cgi/l/email-protection)", user.getEmail(), "Email should be [[email protected]](/web/20221031071901/https://www.baeldung.com/cdn-cgi/l/email-protection)");
assertTrue(user.getActivated(), "User should be activated");
```

运行测试后，只有第一个断言失败:

```java
org.opentest4j.AssertionFailedError: Username should be admin ==> 
Expected :admin
Actual   :baeldung
```

假设我们修复了失败的代码或测试，并重新运行测试。然后我们会得到第二次失败，以此类推。在这种情况下，最好将所有这些断言归为一个通过/失败。

## 4.使用`assertAll() `方法

我们可以使用 `assertAll()`将断言与 JUnit5 组合在一起。

### 4.1.理解`assertAll()`

`assertAll()` 断言函数接受多个`Executable`对象的集合:

```java
assertAll(
  "Grouped Assertions of User",
  () -> assertEquals("baeldung", user.getUsername(), "Username should be baeldung"),
  // more assertions
  ...
 );
```

因此，我们可以使用 lambdas 来提供我们的每个断言。lambda 将被调用来运行由`assertAll()`提供的分组中的断言。

这里，在`assertAll()`的第一个参数中，我们也提供了一个描述来解释整个组的含义。

### 4.2.使用`assertAll()`对断言进行分组

让我们看看完整的例子:

```java
User user = new User("baeldung", "[[email protected]](/web/20221031071901/https://www.baeldung.com/cdn-cgi/l/email-protection)", false);
assertAll(
  "Grouped Assertions of User",
  () -> assertEquals("admin", user.getUsername(), "Username should be admin"),
  () -> assertEquals("[[email protected]](/web/20221031071901/https://www.baeldung.com/cdn-cgi/l/email-protection)", user.getEmail(), "Email should be [[email protected]](/web/20221031071901/https://www.baeldung.com/cdn-cgi/l/email-protection)"),
  () -> assertTrue(user.getActivated(), "User should be activated")
);
```

现在，让我们看看运行测试时会发生什么:

```java
org.opentest4j.MultipleFailuresError: Grouped Assertions of User (3 failures)
org.opentest4j.AssertionFailedError: Username should be admin ==> expected: <admin> but was: <baeldung>
org.opentest4j.AssertionFailedError: Email should be [[email protected]](/web/20221031071901/https://www.baeldung.com/cdn-cgi/l/email-protection) ==> expected: <[[email protected]](/web/20221031071901/https://www.baeldung.com/cdn-cgi/l/email-protection)> but was: <[[email protected]](/web/20221031071901/https://www.baeldung.com/cdn-cgi/l/email-protection)>
org.opentest4j.AssertionFailedError: User should be activated ==> expected: <true> but was: <false>
```

与多个断言的情况相反，**这一次所有的断言都被执行，它们的失败在`MultipleFailuresError`消息中报告。**

我们要注意的是`assertAll()`只处理` AssertionError`。**如果任何断言以异常结束，而不是通常的`AssertionError`，执行将立即停止**，错误输出将与异常有关，而不是`MultipleFailuresError`。

## 5.结论

在本文中，我们学习了在 JUnit5 中使用`assertAll()`，并看到了它与使用多个独立断言的不同之处。

和往常一样，教程的完整代码可以在 GitHub 上找到[。](https://web.archive.org/web/20221031071901/https://github.com/eugenp/tutorials/tree/master/testing-modules/junit-5)