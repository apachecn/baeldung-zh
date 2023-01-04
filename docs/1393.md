# 具有默认值的 Lombok 生成器

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/lombok-builder-default-value>

## 1。简介

在这个快速教程中，我们将研究如何在使用带有`Lombok`的构建器模式时为属性提供默认值。

请务必查看我们的[龙目岛](/web/20221127130432/https://www.baeldung.com/intro-to-project-lombok)简介。

## 2。依赖性

在本教程中，我们将使用 [Lombok](https://web.archive.org/web/20221127130432/https://search.maven.org/artifact/org.projectlombok/lombok/1.18.2/jar) ，因此我们只需要一个依赖项:

```
<dependency>
    <groupId>org.projectlombok</groupId>
    <artifactId>lombok</artifactId>
    <version>1.18.20</version>
    <scope>provided</scope>
</dependency>
```

## 3。POJO 与 Lombok Builder

首先，让我们看看 Lombok 如何帮助我们摆脱实现构建器模式所需的样板代码。

我们将从一个简单的 POJO 开始:

```
public class Pojo {
    private String name;
    private boolean original;
}
```

为了让这个类有用，我们需要 getters。此外，如果我们希望在 ORM 中使用这个类，我们可能需要一个默认的构造函数。

除此之外，我们希望这个类有一个构建器。使用 Lombok，我们可以通过一些简单的注释实现所有这些:

```
@Getter
@Builder
@NoArgsConstructor
@AllArgsConstructor
public class Pojo {
    private String name;
    private boolean original;
}
```

## 4。定义期望

让我们以单元测试的形式为我们想要达到的目标定义一些期望。

第一个也是最基本的要求是，在我们用构建器构建一个对象之后，要有默认值:

```
@Test
public void givenBuilderWithDefaultValue_ThanDefaultValueIsPresent() {
    Pojo build = Pojo.builder()
        .build();
    Assert.assertEquals("foo", build.getName());
    Assert.assertTrue(build.isOriginal());
}
```

**当然，这个测试失败了，因为`@Builder`注释没有填充值。**我们会尽快解决这个问题。

如果我们使用 ORM，它通常依赖于默认的构造函数。因此，我们应该期望默认构造函数的行为与构建器的行为相同:

```
@Test
public void givenBuilderWithDefaultValue_NoArgsWorksAlso() {
    Pojo build = Pojo.builder()
        .build();
    Pojo pojo = new Pojo();
    Assert.assertEquals(build.getName(), pojo.getName());
    Assert.assertTrue(build.isOriginal() == pojo.isOriginal());
}
```

在这个阶段，这个测试通过了。

现在让我们看看如何通过这两项测试。

## 5。龙目岛的`Builder.Default`注解

从 Lombok v1.16.16 开始，我们可以使用`@Builder`的内部注释:

```
// class annotations as before
public class Pojo {
    @Builder.Default
    private String name = "foo";
    @Builder.Default
    private boolean original = true;
}
```

它很简单，可读性很强，但是有一些缺陷。

缺省值将出现在构建器中，使第一个测试用例通过。**不幸的是，无参数构造函数无法获得默认值，这使得第二个测试用例失败，**即使没有生成无参数构造函数，而是显式编写。

`Builder.Default`注释的这种副作用从一开始就存在，并且可能会伴随我们很长一段时间。

## 6。初始化构建器

我们可以通过在极简构建器实现中定义缺省值来尝试通过这两个测试:

```
// class annotations as before
public class Pojo {
    private String name = "foo";
    private boolean original = true;

    public static class PojoBuilder {
        private String name = "foo";
        private boolean original = true;
    }
}
```

这样，两项测试都会通过。

不幸的是，代价是代码重复。**对于有几十个字段的`POJO`，维护双重初始化可能容易出错。**

但是如果我们愿意付出这个代价，我们还应该注意另外一件事。如果我们在 IDE 中使用重构来重命名我们的类，静态内部类不会被自动重命名。那么龙目岛就找不到它，我们的代码就会被破解。

为了消除这种风险，我们可以装饰构建器的注释:

```
// class annotations as before
@Builder(builderClassName = "PojoBuilder")
public class Pojo {
    private String name = "foo";
    private boolean original = true;

    public static class PojoBuilder {
        private String name = "foo";
        private boolean original = true;
    }
}
```

## 7。使用`toBuilder`

`@Builder `还支持从原始类的实例生成构建器的实例。默认情况下，此功能不启用。我们可以通过在构建器注释中设置`toBuilder`参数来启用它:

```
// class annotations as before
@Builder(toBuilder = true)
public class Pojo {
    private String name = "foo";
    private boolean original = true;
}
```

有了这个，**我们就可以摆脱双重初始化**。

当然，这是有代价的。我们必须实例化这个类来创建一个构建器。所以我们也必须修改我们的测试:

```
@Test
public void givenBuilderWithDefaultValue_ThenDefaultValueIsPresent() {
    Pojo build =  new Pojo().toBuilder()
        .build();
    Assert.assertEquals("foo", build.getName());
    Assert.assertTrue(build.isOriginal());
}

@Test
public void givenBuilderWithDefaultValue_thenNoArgsWorksAlso() {
    Pojo build = new Pojo().toBuilder()
        .build();
    Pojo pojo = new Pojo();
    Assert.assertEquals(build.getName(), pojo.getName());
    Assert.assertTrue(build.isOriginal() == pojo.isOriginal());
}
```

同样，两个测试都通过了，所以我们使用无参数构造函数时的默认值与使用构建器时相同。

## 8。结论

在本文中，我们探讨了为 Lombok 构建器提供默认值的几个选项。

`Builder`的副作用。`Default`注解值得关注。但其他选择也有其缺点，所以我们必须根据目前的情况谨慎选择。

和往常一样，代码可以在 GitHub 的[上获得。](https://web.archive.org/web/20221127130432/https://github.com/eugenp/tutorials/tree/master/lombok-modules/lombok)