# 使用 Lombok 的@Accessors 注释

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/lombok-accessors>

## 1.概观

在我们的域对象中使用`get` 和`set` 方法是非常典型的，但是我们可能会发现其他更有表现力的方法。

在本教程中，我们将了解 [Project Lombok](/web/20220926200324/https://www.baeldung.com/intro-to-project-lombok) 的 [`@Accessors`注释](https://web.archive.org/web/20220926200324/https://projectlombok.org/features/experimental/Accessors)及其对流畅、链式和自定义访问器的支持。

不过，在继续之前，我们的 [IDE 需要安装 Lombok](/web/20220926200324/https://www.baeldung.com/lombok-ide)。

## 2.标准访问器

在我们查看`@Accessors` 注释、**之前，让我们回顾一下 Lombok 在默认情况下是如何处理`@Getter` 和`@Setter` 注释的。**

首先，让我们创建我们的类:

```java
@Getter
@Setter
public class StandardAccount {
    private String name;
    private BigDecimal balance;
}
```

现在让我们创建一个测试用例。我们可以在测试中看到，Lombok 添加了典型的 getter 和 setter 方法:

```java
@Test
public void givenStandardAccount_thenUseStandardAccessors() {
    StandardAccount account = new StandardAccount();
    account.setName("Basic Accessors");
    account.setBalance(BigDecimal.TEN);

    assertEquals("Basic Accessors", account.getName());
    assertEquals(BigDecimal.TEN, account.getBalance()); 
}
```

**当我们查看`@Accessor`的选项时，我们将看到这个测试用例是如何变化的。**

## 3.流畅的访问者

让我们从`fluent`选项开始:

```java
@Accessors(fluent = true)
```

**`fluent` 选项为我们提供了没有`get`或`set`前缀的访问器。**

我们一会儿会看一下`chain` 选项，但是因为它是默认启用的，所以现在让我们显式地禁用它:

```java
@Accessors(fluent = true, chain = false)
@Getter
@Setter
public class FluentAccount {
    private String name;
    private BigDecimal balance;
}
```

现在，我们的测试仍然表现相同，但是我们改变了访问和改变状态的方式:

```java
@Test
public void givenFluentAccount_thenUseFluentAccessors() {
    FluentAccount account = new FluentAccount();
    account.name("Fluent Account");
    account.balance(BigDecimal.TEN);

    assertEquals("Fluent Account", account.name()); 
    assertEquals(BigDecimal.TEN, account.balance());
}
```

注意前缀是如何消失的。

## 4.链式存取器

现在让我们来看看`chain`选项:

```java
@Accessors(chain = true)
```

**`chain` 选项为我们提供了返回** `**this**. `的设置器。再次注意，它默认为`true`，但是为了清楚起见，我们将显式地设置它。

这意味着我们可以在一个语句中将多个`set` 操作链接在一起。

让我们构建我们的`fluent`访问器，并将`chain`选项改为`true`:

```java
@Accessors(fluent = true, chain = true)
@Getter 
@Setter 
public class ChainedFluentAccount { 
    private String name; 
    private BigDecimal balance;
} 
```

如果我们省略`chain`并且只指定:

```java
@Accessors(fluent = true)
```

现在让我们看看这对我们的测试用例有什么影响:

```java
@Test
public void givenChainedFluentAccount_thenUseChainedFluentAccessors() {
    ChainedFluentAccount account = new ChainedFluentAccount()
      .name("Fluent Account")
      .balance(BigDecimal.TEN);

    assertEquals("Fluent Account", account.name()); 
    assertEquals(BigDecimal.TEN, account.balance());
}
```

注意`new` 语句是如何随着`setters` 链接在一起而变长的，去掉了一些样板文件。

当然，这就是龙目岛的`@Builder` 如何使用`chain` ed `fluent`访问器。

## 5.前缀访问器

最后，有时我们的字段可能有不同的命名约定，而不是我们希望通过 getters 和 setters 公开的。

让我们考虑下面的类，它的字段使用匈牙利符号:

```java
public class PrefixedAccount { 
    private String sName; 
    private BigDecimal bdBalance; 
}
```

如果我们用`@Getter` 和`@Setter`来公开它，我们会得到像`getSName`这样的方法，可读性不是很好。

`prefix`选项允许我们告诉 Lombok 要忽略哪些前缀:

```java
@Accessors(prefix = {"s", "bd"})
@Getter
@Setter
public class PrefixedAccount {
    private String sName;
    private BigDecimal bdBalance;
}
```

因此，让我们看看这是如何影响我们的测试用例的:

```java
@Test
public void givenPrefixedAccount_thenRemovePrefixFromAccessors() {
    PrefixedAccount account = new PrefixedAccount();
    account.setName("Prefixed Fields");
    account.setBalance(BigDecimal.TEN);

    assertEquals("Prefixed Fields", account.getName()); 
    assertEquals(BigDecimal.TEN, account.getBalance());
}
```

注意我们的`sName`字段(`setName,` `getName`)的访问器省略了前导`s`，而`bdBalance`的访问器省略了前导`bd`。

然而，Lombok **只在前缀后面不是小写字母时才使用前缀。**

这确保了如果我们有一个字段没有使用匈牙利符号，比如`state,`，而是以我们的前缀之一`s`开始，我们不会以`getTate()!`结束

最后，假设我们想在我们的符号中使用下划线，但是还想在它后面跟一个小写字母。

让我们添加一个前缀为`s_:`的字段`s_notes`

```java
@Accessors(prefix = "s_")
private String s_notes;
```

遵循小写字母规则，我们会得到类似于`getS_Notes()`的方法，所以 **Lombok 也会在前缀本身以非字母**结尾时应用前缀。

## 6.配置属性

我们可以通过向一个`lombok.config`文件添加配置属性，为我们最喜欢的设置组合设置一个项目或目录范围的缺省值:

```java
lombok.accessors.chain=true
lombok.accessors.fluent=true
```

更多详情参见 Lombok [功能配置指南](https://web.archive.org/web/20220926200324/https://projectlombok.org/features/configuration)。

## 7.结论

在本文中，我们以不同的组合使用了 Lombok 的`@Accessors`注释的`fluent, chain,` 和`prefix`选项，来看看它如何影响生成的代码。

要了解更多信息，请务必查看一下 [Lombok 访问器 JavaDoc](https://web.archive.org/web/20220926200324/https://projectlombok.org/api/lombok/experimental/Accessors.html) 和[实验特性指南](https://web.archive.org/web/20220926200324/https://projectlombok.org/features/experimental/Accessors)。

和往常一样，这篇文章的源代码可以在 GitHub 上找到[。](https://web.archive.org/web/20220926200324/https://github.com/eugenp/tutorials/tree/master/lombok-modules/lombok-2)