# 使用 Lombok 的@Builder 注释

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/lombok-builder>

## 1。概述

Project Lombok 的 [`@Builder`](https://web.archive.org/web/20221208143832/https://projectlombok.org/features/Builder) 是使用[构建器模式](/web/20221208143832/https://www.baeldung.com/creational-design-patterns#builder)而无需编写样板代码的有用机制。我们可以将这个注释应用于一个`Class`或者一个方法。

在这个快速教程中，我们将看看`@Builder`的不同用例。

## 2。Maven 依赖关系

首先，我们需要将[项目龙目岛](https://web.archive.org/web/20221208143832/https://projectlombok.org/)添加到我们的`pom.xml`中:

```
<dependency>
    <groupId>org.projectlombok</groupId>
    <artifactId>lombok</artifactId>
    <version>1.18.20</version>
</dependency>
```

Maven Central 在这里有最新版本的 Project Lombok。

## 3。在一个类上使用`@Builder`

在第一个用例中，我们仅仅实现了一个`Class`，并且我们想要使用一个构建器来创建我们的类的实例。

第一步也是唯一的一步是将注释添加到类声明中:

```
@Getter
@Builder
public class Widget {
    private final String name;
    private final int id;
} 
```

**龙目岛为我们做了所有的工作。**我们现在可以建造一个`Widget`并测试它:

```
Widget testWidget = Widget.builder()
  .name("foo")
  .id(1)
  .build();

assertThat(testWidget.getName())
  .isEqualTo("foo");
assertThat(testWidget.getId())
  .isEqualTo(1);
```

**如果我们想要创建对象的副本或近似副本，我们可以将属性** **`toBuilder = true`添加到`@Builder`注释**:

```
@Builder(toBuilder = true)
public class Widget {
//...
}
```

这告诉 Lombok 将`toBuilder()`方法添加到我们的`Class`中。当我们调用`toBuilder()`方法时，它**返回一个构建器，用它所调用的实例的属性初始化:**

```
Widget testWidget = Widget.builder()
  .name("foo")
  .id(1)
  .build();

Widget.WidgetBuilder widgetBuilder = testWidget.toBuilder();

Widget newWidget = widgetBuilder.id(2).build();
assertThat(newWidget.getName())
  .isEqualTo("foo");
assertThat(newWidget.getId())
  .isEqualTo(2);
```

我们可以在测试代码中看到，由 Lombok 生成的 builder 类被命名为类似于我们的类，并在其后附加了`“Builder”`、`WidgetBuilder` (在本例中为`.`)，然后我们可以修改我们想要的属性，而`build()`是一个新的实例。

如果我们需要指定必需的字段，我们可以使用注释配置来创建一个辅助构建器:

```
@Builder(builderMethodName = "internalBuilder")
public class RequiredFieldAnnotation {
    @NonNull
    private String name;
    private String description;

    public static RequiredFieldAnnotationBuilder builder(String name) {
        return internalBuilder().name(name);
    }
}
```

在这种情况下，我们将默认的`builder`隐藏为`internalBuilder`，并创建我们自己的。因此，当我们创建构建器时，我们必须提供所需的参数:

```
RequiredField.builder("NameField").description("Field Description").build();
```

此外，为了确保我们的字段存在，我们可以添加`@NonNull`注释。

## 4。在方法上使用`@Builder`

假设我们正在使用一个想要用构建器构建的对象，但是我们**不能修改源代码或者扩展**T0

首先，让我们使用 [Lombok 的@Value 注释:](/web/20221208143832/https://www.baeldung.com/intro-to-project-lombok)创建一个简单的例子

```
@Value
final class ImmutableClient {
    private int id;
    private String name;
}
```

现在我们有了一个`final` `Class`，它有两个不可变成员、它们的 getters 和一个全参数构造函数。

**我们介绍了如何在`Class`上使用`@Builder` ，但是我们也可以在方法上使用它。**我们将使用这种能力来解决无法修改或扩展`ImmutableClient`的问题。

然后，我们将创建一个新类，它包含一个创建不变客户端的方法:

```
class ClientBuilder {

    @Builder(builderMethodName = "builder")
    public static ImmutableClient newClient(int id, String name) {
        return new ImmutableClient(id, name);
    }
}
```

该注释创建了一个名为`builder()`的方法，该方法的**返回一个用于创建`ImmutableClients`的`Builder`。**

现在让我们构建一个`ImmutableClient`:

```
ImmutableClient testImmutableClient = ClientBuilder.builder()
  .name("foo")
  .id(1)
  .build();
assertThat(testImmutableClient.getName())
  .isEqualTo("foo");
assertThat(testImmutableClient.getId())
  .isEqualTo(1);
```

## 5。结论

在这篇简短的文章中，我们在一个方法上使用了 Lombok 的`@Builder`注释来创建一个`final` `Class,` 的构建器，并且我们学习了如何使一些`Class`字段成为必需的。

代码样本一如既往地可以在 GitHub 上找到[。](https://web.archive.org/web/20221208143832/https://github.com/eugenp/tutorials/tree/master/lombok-modules/lombok)