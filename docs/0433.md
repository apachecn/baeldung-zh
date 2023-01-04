# 在 Java 中使用可选的

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-optional-uses>

## 1.介绍

在本教程中，我们将看看 Java 中的`Optional`类的用途，以及在构建我们的应用程序时使用它的一些优势。

## 2.`Optional<T>`在 Java 中的用途

`Optional`是表示某物存在或不存在的类。从技术上讲，`Optional`是通用类型`T`的包装类，其中如果`T`是`null`，那么`Optional`实例为空。不然就满了。

[根据 Java 11 文档](https://web.archive.org/web/20221128082835/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/Optional.html),**`Optional`的目的是提供一个返回类型，可以表示在返回`null`可能导致意外错误的场景中缺少值，** 就像臭名昭著的`NullPointerException`。

### 2.1.有用的方法

`Optional`类提供了有用的方法来帮助我们使用那个 API。本文的重点是 `of(), orElse(), ` 和 ` empty() ` 方法 :

*   `of(T value)` 返回一个`Optional`的实例，其值在内
*   `orElse(T other) ` 返回一个`Optional`内的值，否则返回 `other`
*   最后， `empty()` 返回一个`Optional` 的空实例

我们将看到这些方法在构建返回`Optional`的代码和使用它的代码的过程中的作用。

## 3.`Optional`的优点

我们已经了解了`Optional`的目的和它的一些方法。但是，我们怎样才能从我们的课程中受益呢？在这一节中，我们将看到一些使用它的方法，帮助我们创建清晰和健壮的 API。

### 3.1.`Optional`对`null`

在创建`Optional`类之前，我们必须用`null`来表示值的缺失。语言并没有强迫我们正确对待`null`事件。换句话说， **`null`检查有时是必要的，而不是强制性的** 。因此，创建返回`null`的方法被证明是一种产生意外运行时错误的方法，比如`NullPointerException` s.

另一方面， **`Optional`的一个实例在编译时应该总是被适当地处理，以获得它里面的值** 。这种在编译时处理`Optional`的义务导致更少的意外`NullPointerException`

让我们尝试一个模拟`User` s: 的数据库的例子

```
public class User {

    public User(String id, String name) {
        this.id = id;
        this.name = name;
    }

    private String id;

    private String name;

    public String getName() {
        return name;
    }

    public String getId() {
        return id;
    }
}
```

让我们定义一个存储库类，如果找到它，它将返回一个`User`。否则，它返回`null`:

```
public class UserRepositoryWithNull {

    private final List<User> dbUsers = Arrays.asList(new User("1", "John"), new User("2", "Maria"), new User("3", "Daniel"));

    public User findById(String id) {

        for (User u : dbUsers) {
            if (u.getId().equals(id)) {
                return u;
            }
        }

        return null;
    }
}
```

现在，我们将编写一个单元测试，展示如果我们不使用`null`检查来寻址`null`，代码如何被`NullPointerException`破坏:

```
@Test
public void givenNonExistentUserId_whenSearchForUser_andNoNullCheck_thenThrowException() {

    UserRepositoryWithNull userRepositoryWithNull = new UserRepositoryWithNull();
    String nonExistentUserId = "4";

    assertThrows(NullPointerException.class, () -> {
        System.out.println("User name: " + userRepositoryWithNull.findById(nonExistentUserId)
          .getName());
    });
}
```

**Java 允许我们从由`findById()`返回的对象中使用`getName()`方法，而无需进行`null`检查。在这种情况下，我们只能在运行时发现问题。**

为了避免这种情况，我们可以创建另一个存储库，如果我们找到一个`User,`，那么我们返回一个完整的`Optional`。否则，我们返回一个空的。让我们在实践中看看这个:

```
public class UserRepositoryWithOptional {

    private final List<User> dbUsers = Arrays.asList(new User("1", "John"), new User("2", "Maria"), new User("3", "Daniel"));

    public Optional<User> findById(String id) {

        for (User u : dbUsers) {
            if (u.getId().equals(id)) {
                return Optional.of(u);
            }
        }

        return Optional.empty();
    }
}
```

现在，当我们重写单元测试时，我们看到当我们找不到任何`User`时，我们必须如何首先处理`Optional`来获得它的值:

```
@Test
public void givenNonExistentUserId_whenSearchForUser_thenOptionalShouldBeTreatedProperly() {

    UserRepositoryWithOptional userRepositoryWithOptional = new UserRepositoryWithOptional();
    String nonExistentUserId = "4";

    String userName = userRepositoryWithOptional.findById(nonExistentUserId)
      .orElse(new User("0", "admin"))
      .getName();

    assertEquals("admin", userName);
}
```

在上面的案例中，我们没有找到任何`User`，所以我们可以使用 `orElse()` 方法返回一个默认用户。为了获得它的值，必须在编译时正确处理`Optional`。这样，我们可以减少运行时的意外错误。

除了用`orElse()`方法提供默认值，我们还可以使用另外两个选项:使用`orElseThrow()`[抛出异常，或者使用](/web/20221128082835/https://www.baeldung.com/java-optional-throw-exception)[`orElseGet()`调用`Supplier`函数](/web/20221128082835/https://www.baeldung.com/java-optional-or-else-vs-or-else-get)。

### 3.2.设计意图明确的 API

正如我们之前讨论的，`null`被广泛用于表示无。但是，`null`的含义只有创建 API 的人自己清楚。其他开发者浏览这个 API 可能会发现`null`的不同含义。

可能“可选”这个名字是`Optional`在构建我们的 API 时是一个有用工具的主要原因。 **`Optional`在一个方法的返回中提供了一个我们应该从那个方法中期待什么的清晰意图:它返回一些东西或者什么都没有** 。不需要任何文档来解释该方法的返回类型。代码会自己解释。

使用返回`null`的存储库，我们可能会以最坏的方式发现`null`表示在数据库中没有找到`User`。或者它可能代表其他事情，比如连接到数据库时出错，或者对象还没有初始化。很难知道。

另一方面，使用返回一个`Optional`实例的存储库，仅仅通过查看方法签名就可以清楚地知道，我们可能在数据库中找到一个用户，也可能找不到。

**设计清晰 API 的一个重要实践是永远不返回一个`null` `Optional`** 。使用`static`方法，方法应该总是返回一个有效的`Optional`实例。

### 3.3.声明式编程

使用`Optional` 类的另一个好理由是能够使用一系列流畅的方法。它提供了一个类似于集合中的 `stream()` 的“伪流”,但是只有一个值。这意味着我们可以调用类似于 `map()`和`filter() ` 的方法，对其中的值进行操作。这有助于创建更多的声明式程序，而不是命令式程序。

假设要求是，如果名字以字母‘M’开头，将`User`的`name`的大小写改为大写。

首先，让我们看看命令式方法，使用不返回`Optional` : 的存储库

```
@Test
public void givenExistentUserId_whenFoundUserWithNameStartingWithMInRepositoryUsingNull_thenNameShouldBeUpperCased() {

    UserRepositoryWithNull userRepositoryWithNull = new UserRepositoryWithNull();

    User user = userRepositoryWithNull.findById("2");
    String upperCasedName = "";

    if (user != null) {
        if (user.getName().startsWith("M")) {
            upperCasedName = user.getName().toUpperCase();
        }
    }

    assertEquals("MARIA", upperCasedName);
}
```

现在，让我们来看看声明性的方式，使用存储库的`Optional`版本:

```
@Test
public void givenExistentUserId_whenFoundUserWithNameStartingWithMInRepositoryUsingOptional_thenNameShouldBeUpperCased() {

    UserRepositoryWithOptional userRepositoryWithOptional = new UserRepositoryWithOptional();

    String upperCasedName = userRepositoryWithOptional.findById("2")
      .filter(u -> u.getName().startsWith("M"))
      .map(u -> u.getName().toUpperCase())
      .orElse("");

    assertEquals("MARIA", upperCasedName);
}
```

命令式方式需要两个嵌套的`if`语句来检查对象是否不是`null`并过滤用户名。如果没有找到`User`，大写字符串保持为空。

在声明方式中，我们使用 lambda 表达式来过滤名称，并将大写函数映射到找到的`User`。如果没有找到`User`，我们使用 `orElse()` 返回一个空字符串。

我们使用哪一种仍然是个人喜好的问题。他们两个达到了同样的结果。命令式方法需要更多的挖掘来理解代码的含义。例如，如果我们在第一个或第二个`if`语句中添加更多的逻辑，可能会对代码的意图造成一些混淆。在这种场景中，**声明式编程清楚地表明了代码**的意图:返回大写的名称，否则返回空字符串。

## 4.结论

在本文中，我们研究了`Optional`类的用途以及如何有效地使用它来设计清晰和健壮的 API。

与往常一样，该示例的源代码可以在 GitHub 上的[处获得。](https://web.archive.org/web/20221128082835/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-optional)