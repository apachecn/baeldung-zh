# 对 Lombok Builders 使用@Singular 注释

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/lombok-builder-singular>

## 1.概观

Lombok 库提供了一种简化数据对象的好方法。[项目 Lombok](/web/20220918134852/https://www.baeldung.com/intro-to-project-lombok) 的关键特性之一是 [`@Builder`注释](/web/20220918134852/https://www.baeldung.com/lombok-builder)，它自动创建用于创建不可变对象的构建器类。然而，用标准的 Lombok 生成的`Builder`类在我们的对象中填充集合会很笨拙。

在本教程中，我们将看一下`@Singular`注释，它**帮助我们处理数据对象中的集合。**正如我们将看到的，它还强制执行良好的实践。

## 2.建筑商和收藏品

类以其简单、流畅的语法使得构造不可变的数据对象变得容易。让我们看一个用 Lombok 的`@Builder`注释注释的示例类:

```java
@Getter
@Builder
public class Person {
    private final String givenName;
    private final String additionalName;
    private final String familyName;
    private final List<String> tags;
}
```

我们现在可以使用构建器模式创建`Person`的实例。注意这里的`tags`属性是一个`List`。此外，标准 Lombok `@Builder`将提供设置该属性的方法，就像非列表属性一样:

```java
Person person = Person.builder()
  .givenName("Aaron")
  .additionalName("A")
  .familyName("Aardvark")
  .tags(Arrays.asList("fictional","incidental"))
  .build();
```

这是一个可行但相当笨拙的语法。我们可以像上面所做的那样，以内联方式创建集合。或者，我们可以提前申报。无论哪种方式，它都会中断我们的对象创建流程。这就是`@Singular`注释派上用场的地方。

### 2.1.使用带有`List` s 的`@Singular`注释

让我们给我们的`Person`对象添加另一个`List`，并用`@Singular`对其进行注释。这将为我们提供一个带注释和不带注释的字段的并列视图。除了一般的`tags`属性之外，我们还将在`Person`中添加一个`interests`列表:

```java
@Singular private final List<String> interests;
```

**我们现在可以一次建立一个值列表:**

```java
Person person = Person.builder()
  .givenName("Aaron")
  .additionalName("A")
  .familyName("Aardvark")
  .interest("history")
  .interest("sport")
  .build();
```

构建器会将每个元素存储在一个`List`中，并在我们调用`build()`时创建适当的`Collection`。

### 2.2.使用其他`Collection`类型

我们已经在这里用一个`java.util.List`说明了`@Singular`的工作，但是**它也可以应用于其他 Java `Collection`类**。让我们为我们的`Person`添加更多成员:

```java
@Singular private final Set<String> skills;
@Singular private final Map<String, LocalDate> awards;
```

就`Builder` s 而言,`Set`的行为与`List`非常相似——我们可以一个一个地添加元素:

```java
Person person = Person.builder()
  .givenName("Aaron")
  .skill("singing")
  .skill("dancing")
  .build();
```

因为`Set`不支持重复，我们需要注意多次添加相同的元素不会创建多个元素。`Builder`会宽大处理这种情况。我们可以多次添加一个元素，但是创建的`Set`只会出现一次。

`Map`的处理略有不同，其中`Builder`公开的方法采用适当类型的键和值:

```java
Person person = Person.builder()
  .givenName("Aaron")
  .award("Singer of the Year", LocalDate.now().minusYears(5))
  .award("Best Dancer", LocalDate.now().minusYears(2))
  .build();
```

正如我们在`Set` s 中看到的，构建器对重复的`Map`键很宽容，如果同一个键被赋值多次，将使用最后一个值。

## 3.`@Singular`方法的命名

到目前为止，我们一直依赖于`@Singular`注释中的一点魔法，而没有引起人们的注意。`Builder`本身提供了一种使用复数形式一次性分配整个集合的方法，例如“T2”。**注释添加的额外方法使用单数形式**，例如，“`award`”。

Lombok 足够聪明，可以识别英语中简单的复数单词，它们遵循一种规则的模式。在我们到目前为止使用的所有例子中，它只是去掉了最后的“s”。

它还知道，对于一些以“es”结尾的单词，要去掉最后两个字母。例如，它知道“grass”是“grapes”的单数，而“grape”是“grapes”的单数，而不是“grap”。然而，在某些情况下，我们必须给它一些帮助。

让我们建立一个简单的海洋模型，包含鱼和海草:

```java
@Getter
@Builder
public class Sea {
    @Singular private final List<String> grasses;
    @Singular private final List<String> fish;
}
```

Lombok 可以处理“草”这个词，但与“鱼”一起丢失了。在英语中，单数和复数形式是一样的，奇怪的是。这段代码无法编译，我们会得到一个错误:

```java
Can't singularize this name; please specify the singular explicitly (i.e. @Singular("sheep"))
```

我们可以通过在注释中添加一个值作为单一的方法名来进行分类:

```java
@Singular("oneFish") private final List<String> fish;
```

我们现在可以编译代码并使用`Builder`:

```java
Sea sea = Sea.builder()
  .grass("Dulse")
  .grass("Kelp")
  .oneFish("Cod")
  .oneFish("Mackerel")
  .build();
```

在这种情况下，我们选择了相当不自然的`oneFish()`，但是同样的方法也可以用于具有明显复数的非标准单词。例如，`children`的`List`可以被提供一个方法`child()`。

## 4.不变

我们已经看到了`@Singular`注释如何帮助我们使用 Lombok 中的集合。除了提供便利和表达能力，它还可以帮助我们保持代码的整洁。

不可变对象被定义为一旦创建就不能修改的对象。例如，不变性在反应式架构中很重要，因为它允许我们将一个对象传递给一个方法，并保证没有副作用。为了支持不变性，Builder 模式最常用来替代 POJO 的 getters 和 setters。

当我们的数据对象包含`Collection`类时，很容易忽略不变性。基本集合接口——`List`、`Set`和`Map`——都有可变和不可变的实现。如果我们依赖标准的 Lombok 构建器，我们可以意外地传入一个可变的集合，然后修改它:

```java
List<String> tags= new ArrayList();
tags.add("fictional");
tags.add("incidental");
Person person = Person.builder()
  .givenName("Aaron")
  .tags(tags)
  .build();
person.getTags().clear();
person.getTags().add("non-fictional");
person.getTags().add("important");
```

在这个简单的例子中，我们不得不非常努力地去犯这个错误。例如，如果我们使用`Arrays.asList()`来构造变量`tags`，我们将会免费获得一个不可变列表，并且对`add()`或`clear()`的调用将会抛出一个`UnsupportedOperationException`。

例如，在实际编码中，如果集合作为参数传入，则更有可能发生错误。然而，**很高兴知道有了`@Singular`，我们可以使用基本的`Collection`接口，并在调用`build()`** 时获得不可变的实例。

## 5.结论

在本教程中，我们已经看到了 Lombok `@Singular`注释如何提供一种使用构建器模式处理`List`、`Set`和`Map`接口的便捷方式。构建器模式支持不变性，`@Singular`为此提供了一流的支持。

像往常一样，完整的代码示例可以在 GitHub 的[上找到。](https://web.archive.org/web/20220918134852/https://github.com/eugenp/tutorials/tree/master/lombok-modules/lombok)