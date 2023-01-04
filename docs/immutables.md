# 不变量导论

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/immutables>

## 1。简介

在本文中，我们将展示如何使用[不可变](https://web.archive.org/web/20220812050011/https://immutables.github.io/)库。

该库由注释和注释处理器组成，用于生成和处理可序列化和可定制的不可变对象。

## 2.Maven 依赖性

为了在您的项目中使用不变量，您需要将以下依赖项添加到您的`pom.xml`文件的`dependencies`部分:

```
<dependency>
    <groupId>org.immutables</groupId>
    <artifactId>value</artifactId>
    <version>2.2.10</version>
    <scope>provided</scope>
</dependency>
```

因为在运行时不需要这个工件，所以指定`provided`范围是明智的。

这个库的最新版本可以在这里找到。

## 3\. Immutables

该库从抽象类型中生成不可变对象:`Interface`、`Class`、`Annotation`。

实现这一点的关键是正确使用`@Value.Immutable`注释。**它生成一个带注释类型的不可变版本，并在名字前加上关键字`Immutable`和**。

如果我们试图生成一个名为"`X`"的不可变版本的类，它将生成一个名为`“ImmutableX”.` 的类。生成的类不是递归不可变的，所以最好记住这一点。

快速注意——因为 Immutables 利用注释处理，所以需要记住在 IDE 中启用注释处理。

### 3.1。使用`@Value.Immutable`与`Abstract Classes` 和 `Interfaces`与

让我们创建一个简单的`abstract`类`Person`，它由两个代表将要生成的字段的`abstract`访问器方法组成，然后用`@Value.Immutable` 注释对该类进行注释:

```
@Value.Immutable
public abstract class Person {

    abstract String getName();
    abstract Integer getAge();

}
```

注释处理完成后，我们可以在`target/generated-sources`目录中找到一个现成的、**新生成的`ImmutablePerson`类:**

```
@Generated({"Immutables.generator", "Person"})
public final class ImmutablePerson extends Person {

    private final String name;
    private final Integer age;

    private ImmutablePerson(String name, Integer age) {
        this.name = name;
        this.age = age;
    }

    @Override
    String getName() {
        return name;
    }

    @Override
    Integer getAge() {
        return age;
    }

    // toString, hashcode, equals, copyOf and Builder omitted

}
```

生成的类带有已实现的`toString`、`hashcode`、`equals`方法和步进生成器`ImmutablePerson.Builder`。注意，生成的构造函数有`private`访问权。

为了构造一个`ImmutablePerson`类的实例，我们需要使用构建器或静态方法`ImmutablePerson.copyOf,` ，它可以从一个 `Person`对象创建一个 `ImmutablePerson` 副本。

如果我们想使用构建器构建一个实例，我们可以简单地编写代码:

```
ImmutablePerson john = ImmutablePerson.builder()
  .age(42)
  .name("John")
  .build();
```

生成的类是不可变的，这意味着它们不能被修改。如果您想要修改一个已经存在的对象，您可以使用其中一个“`withX`”方法，该方法不会修改一个原始对象，但是会创建一个带有已修改字段的新实例。

让我们更新`john's`年龄并创建一个新的`john43`对象:

```
ImmutablePerson john43 = john.withAge(43); 
```

在这种情况下，下列断言将是正确的:

```
assertThat(john).isNotSameAs(john43);
```

```
assertThat(john.getAge()).isEqualTo(42);
```

## 4。附加实用程序

如果不能定制，这样的类生成将不会非常有用。Immutables 库附带了一组附加注释，可用于定制`@Value.Immutable`的输出。要查看所有这些，请参考 Immutables 的[文档](https://web.archive.org/web/20220812050011/https://immutables.github.io/immutable.html)。

### 4.1。`@Value.Parameter`注解

`@Value.Parameter`注释可以用于指定字段，应该为这些字段生成构造函数方法。

如果您像这样注释您的类:

```
@Value.Immutable
public abstract class Person {

    @Value.Parameter
    abstract String getName();

    @Value.Parameter
    abstract Integer getAge();
}
```

可以通过以下方式实例化它:

```
ImmutablePerson.of("John", 42);
```

### 4.2。`@Value.Default`注解

`@Value.Default`注释允许您指定一个缺省值，当没有提供初始值时应该使用这个缺省值。为此，您需要创建一个返回固定值的非抽象访问器方法，并用`@Value.Default`对其进行注释:

```
@Value.Immutable
public abstract class Person {

    abstract String getName();

    @Value.Default
    Integer getAge() {
        return 42;
    }
}
```

以下断言将是正确的:

```
ImmutablePerson john = ImmutablePerson.builder()
  .name("John")
  .build();

assertThat(john.getAge()).isEqualTo(42);
```

### 4.3。`@Value.Auxiliary`注解

`@Value.Auxiliary`注释可用于注释将存储在对象实例中的属性，但将被`equals`、`hashCode`和`toString`实现忽略。

如果您像这样注释您的类:

```
@Value.Immutable
public abstract class Person {

    abstract String getName();
    abstract Integer getAge();

    @Value.Auxiliary
    abstract String getAuxiliaryField();

}
```

当使用`auxiliary`字段时，以下断言将为真:

```
ImmutablePerson john1 = ImmutablePerson.builder()
  .name("John")
  .age(42)
  .auxiliaryField("Value1")
  .build();

ImmutablePerson john2 = ImmutablePerson.builder()
  .name("John")
  .age(42)
  .auxiliaryField("Value2")
  .build(); 
```

```
assertThat(john1.equals(john2)).isTrue();
```

```
assertThat(john1.toString()).isEqualTo(john2.toString()); 
```

```
assertThat(john1.hashCode()).isEqualTo(john2.hashCode());
```

### 4.4。`@Value.Immutable(Prehash = True)`注解

因为我们生成的类是不可变的，永远不会被修改，`hashCode`结果将永远保持不变，并且在对象的实例化过程中只能计算一次。

如果您像这样注释您的类:

```
@Value.Immutable(prehash = true)
public abstract class Person {

    abstract String getName();
    abstract Integer getAge();

}
```

当检查生成的类时，您可以看到`hashcode`值现在已经预先计算并存储在一个字段中:

```
@Generated({"Immutables.generator", "Person"})
public final class ImmutablePerson extends Person {

    private final String name;
    private final Integer age;
    private final int hashCode;

    private ImmutablePerson(String name, Integer age) {
        this.name = name;
        this.age = age;
        this.hashCode = computeHashCode();
    }

    // generated methods

    @Override
    public int hashCode() {
        return hashCode;
    }
} 
```

`hashCode()`方法返回在构造对象时生成的预计算的`hashcode`。

## 5。结论

在这个快速教程中，我们展示了[不变量](https://web.archive.org/web/20220812050011/https://immutables.github.io/)库的基本工作原理。

文章中的所有源代码和单元测试都可以在 [GitHub 存储库](https://web.archive.org/web/20220812050011/https://github.com/eugenp/tutorials/tree/master/libraries-3)中找到。