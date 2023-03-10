# 在 Java 中按多个字段对对象集合进行排序

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-sort-collection-multiple-fields>

## 1.概观

编程时，我们经常需要对对象集合进行排序。如果我们想要对多个字段上的对象进行排序，排序逻辑有时会变得难以实现。在本教程中，我们将讨论解决该问题的几种不同方法，以及它们的优缺点。

## 2.示例`Person`类

让我们用两个字段定义一个`Person`类，`name`和`age`。在我们的例子中，我们将首先基于`name`然后基于`age`来比较`Person`对象:

```java
public class Person {
    @Nonnull private String name;
    private int age;

    // constructor
    // getters and setters
}
```

这里，我们添加了一个`@Nonnull`注释来保持例子的简单。但是在生产代码中，我们可能需要处理可空字段的比较。

## 3.使用`Comparator.compare()`

Java 提供了`[Comparator](/web/20221120022818/https://www.baeldung.com/java-comparator-comparable)`接口来比较两个相同类型的对象。我们可以用定制的逻辑实现它的`compare(T o1, T o2)`方法来执行期望的比较。

### 3.1.逐一检查不同的字段

让我们一个接一个地比较这些字段:

```java
public class CheckFieldsOneByOne implements Comparator<Person> {
    @Override
    public int compare(Person o1, Person o2) {
        int nameCompare = o1.getName().compareTo(o2.getName());
        if(nameCompare != 0) {
            return nameCompare;
        }
        return Integer.compare(o1.getAge(), o2.getAge());
    }
}
```

这里，我们使用`String`类的`compareTo()`方法和`Integer`类的`compare()`方法来逐个比较`name`和`age`字段。

这需要大量的打字工作，有时还需要处理许多特殊情况。所以，当我们有更多的领域可以比较的时候，就很难维护和规模化了。一般情况下，不建议在生产代码中使用这种方法。

### 3.2.用番石榴的`ComparisonChain`

首先，让我们将谷歌番石榴库[依赖项](https://web.archive.org/web/20221120022818/https://search.maven.org/artifact/com.google.guava/guava-bom/31.1-jre/pom)添加到我们的`pom.xml`:

```java
<dependency>
    <groupId>com.google.guava</groupId>
    <artifactId>guava</artifactId>
    <version>31.1-jre</version>
</dependency>
```

我们可以通过使用该库中的`ComparisonChain`类来简化逻辑:

```java
public class ComparisonChainExample implements Comparator<Person> {
    @Override
    public int compare(Person o1, Person o2) {
        return ComparisonChain.start()
          .compare(o1.getName(), o2.getName())
          .compare(o1.getAge(), o2.getAge())
          .result();
    }
}
```

在这里，我们使用`ComparisonChain`中的`compare(int left, int right)`和`compare(Comparable<?> left, Comparable<?> right)`方法来分别比较`name`和`age`。

**这种方法隐藏了比较的细节，只暴露了我们关心的东西——我们想要比较的字段以及它们应该被比较的顺序。**此外，我们应该注意，我们不需要任何额外的逻辑来进行`null`处理，因为库方法会处理它。因此，维护和扩展变得更加容易。

### 3.3.使用 Apache Commons 排序`CompareToBuilder`

首先，让我们将 Apache Commons 的[依赖项](https://web.archive.org/web/20221120022818/https://search.maven.org/artifact/org.apache.commons/commons-lang3/3.12.0/jar)添加到`pom.xml`中:

```java
<dependency>
    <groupId>org.apache.commons</groupId>
    <artifactId>commons-lang3</artifactId>
    <version>3.12.0</version>
</dependency>
```

与前面的例子类似，我们可以使用 Apache Commons 中的`CompareToBuilder`来减少所需的样板代码:

```java
public class CompareToBuilderExample implements Comparator<Person> {
    @Override
    public int compare(Person o1, Person o2) {
        return new CompareToBuilder()
          .append(o1.getName(), o2.getName())
          .append(o1.getAge(), o2.getAge())
          .build();
    }
}
```

这种方法与 Guava 的`ComparisonChain` — **非常相似，它也隐藏了比较细节，易于维护和扩展`.`—**

## 4.使用`Comparator.comparing()`和λ表达式

从 Java 8 开始，`Comparator`接口中增加了几个`static`方法，它们可以使用 lambda 表达式创建一个`Comparator`对象。我们可以用它的 [`comparing()`方法](/web/20221120022818/https://www.baeldung.com/java-8-comparator-comparing)来构造我们需要的`Comparator`:

```java
public static Comparator<Person> createPersonLambdaComparator() {
    return Comparator.comparing(Person::getName)
      .thenComparing(Person::getAge);
}
```

**这种方法更加简洁易读，因为它直接获取了`Person`类**的 getters。

它还保持了我们前面看到的方法的可维护性和可伸缩性特征。此外，**这里的 getters 是延迟评估的**，与之前方法中的直接评估相比。因此，它的性能更好，更适合需要大量大数据比较的延迟敏感型系统。

此外，这种方法只使用核心 Java 类，不需要任何第三方库作为依赖。总的来说，**这是最值得推荐的方法**。

## 5.检查比较结果

让我们测试我们看到的四个比较器，并检查它们的行为。所有这些比较器都可以以相同的方式调用，并且应该产生相同的结果:

```java
@Test
public void testComparePersonsFirstNameThenAge() {
    Person person1 = new Person("John", 21);
    Person person2 = new Person("Tom", 20);
    // Another person named John
    Person person3 = new Person("John", 22);

    List<Comparator<Person>> comparators =
      Arrays.asList(new CheckFieldsOneByOne(),
        new ComparisonChainExample(),
        new CompareToBuilderExample(),
        createPersonLambdaComparator());
    // All comparators should produce the same result
    for(Comparator<Person> comparator : comparators) {
        Assertions.assertIterableEquals(
          Arrays.asList(person1, person2, person3)
            .stream()
            .sorted(comparator)
            .collect(Collectors.toList()),
          Arrays.asList(person1, person3, person2));
    }
}
```

在这里，`person1`与`person3,`同名(“约翰”)但更年轻(21 < 22)，而*person 3′*的名字(“约翰”)在字典上比 *person2* 的名字(“汤姆”)少。所以，最后的排序是`person1`、`person3`、`person2`。

此外，我们应该**注意，如果我们在类变量`name`上没有`@Nonnull`注释，我们需要添加额外的逻辑来处理所有方法中的空情况，除了 Apache Commons 的`CompareToBuilder`(内置了原生空处理)**。

## 6.结论

在本文中，我们学习了在对对象集合进行排序时对多个字段进行比较的不同方法。

与往常一样，GitHub 上的[提供了示例的源代码。](https://web.archive.org/web/20221120022818/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-collections-4)