# 比较 Java 中的对象

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-comparing-objects>

## 1.介绍

比较对象是面向对象编程语言的一个基本特性。

在本教程中，我们将探索 Java 语言中允许我们比较对象的一些特性。我们还将看看外部库中的这些特性。

## `2\. ==`和`!=`运算符

让我们从`==`和`!=`操作符开始，它们可以分别判断两个 Java 对象是否相同。

### 2.1.基元

**对于原始类型，相同意味着具有相等的值:**

```java
assertThat(1 == 1).isTrue();
```

多亏了自动拆箱，**当比较一个原始值和它的包装器类型对应物**时，这也是有效的:

```java
Integer a = new Integer(1);
assertThat(1 == a).isTrue();
```

如果两个整数的值不同，`==`运算符将返回`false`，而`!=`运算符将返回`true`。

### 2.2.目标

假设我们想要比较两个具有相同值的`Integer`包装器类型:

```java
Integer a = new Integer(1);
Integer b = new Integer(1);

assertThat(a == b).isFalse();
```

通过比较两个对象，**这些对象的值不是 1。相反，它们在堆栈** 中的[内存地址是不同的，因为这两个对象都是使用`new`操作符创建的。如果我们将`a`分配给`b`，那么我们会得到不同的结果:](/web/20220628145611/https://www.baeldung.com/java-stack-heap)

```java
Integer a = new Integer(1);
Integer b = a;

assertThat(a == b).isTrue();
```

现在让我们看看使用`Integer#valueOf` 工厂方法时会发生什么:

```java
Integer a = Integer.valueOf(1);
Integer b = Integer.valueOf(1);

assertThat(a == b).isTrue();
```

在这种情况下，它们被认为是相同的。这是因为`valueOf()`方法将`Integer`存储在缓存中，以避免创建太多具有相同值的包装对象。因此，该方法为两个调用返回相同的`Integer`实例。

Java 也为`String`这样做:

```java
assertThat("Hello!" == "Hello!").isTrue();
```

然而，如果它们是使用`new`操作符创建的，那么它们就不会相同。

最后，**两个`null`引用被认为是相同的，而任何非`null`对象被认为与`null`** 不同:

```java
assertThat(null == null).isTrue();

assertThat("Hello!" == null).isFalse();
```

当然，相等运算符的行为可能是限制性的。如果我们想要比较映射到不同地址的两个对象，并且根据它们的内部状态认为它们是相等的，该怎么办？我们将在下一节中看到如何做到这一点。

## `3\. Object#equals`方法

现在让我们用`equals()`方法来讨论一个更广泛的平等概念。

这个方法在`Object`类中定义，这样每个 Java 对象都可以继承它。默认情况下，**的实现会比较对象内存地址，因此它的工作方式与`==` 操作符**相同。但是，我们可以覆盖这个方法，以便定义等式对我们的对象意味着什么。

首先，让我们看看它对现有对象如`Integer`的表现:

```java
Integer a = new Integer(1);
Integer b = new Integer(1);

assertThat(a.equals(b)).isTrue();
```

当两个对象相同时，该方法仍然返回`true`。

我们应该注意，我们可以传递一个`null`对象作为方法的参数，但不能作为我们调用方法的对象。

我们也可以对自己的对象使用`equals()`方法。假设我们有一个`Person`类:

```java
public class Person {
    private String firstName;
    private String lastName;

    public Person(String firstName, String lastName) {
        this.firstName = firstName;
        this.lastName = lastName;
    }
}
```

我们可以覆盖这个类的`equals()`方法，这样我们就可以根据两个`Person`的内部细节来比较它们:

```java
@Override
public boolean equals(Object o) {
    if (this == o) return true;
    if (o == null || getClass() != o.getClass()) return false;
    Person that = (Person) o;
    return firstName.equals(that.firstName) &&
      lastName.equals(that.lastName);
}
```

欲了解更多信息，请查看我们关于这个话题的[文章。](/web/20220628145611/https://www.baeldung.com/java-equals-hashcode-contracts)

## `4\. Objects#equals`静态法

现在我们来看看 [`Objects#equals` 静法](https://web.archive.org/web/20220628145611/https://docs.oracle.com/en/java/javase/14/docs/api/java.base/java/util/Objects.html#equals(java.lang.Object,java.lang.Object))。我们之前提到过不能用`null`作为第一个对象的值，否则会抛出一个`NullPointerException`。

**助手类的`equals()`方法解决了这个问题。它接受两个参数并比较它们，还处理`null`值。**

让我们再来比较一下`Person`物体:

```java
Person joe = new Person("Joe", "Portman");
Person joeAgain = new Person("Joe", "Portman");
Person natalie = new Person("Natalie", "Portman");

assertThat(Objects.equals(joe, joeAgain)).isTrue();
assertThat(Objects.equals(joe, natalie)).isFalse();
```

正如我们解释的，这个方法处理`null`值。因此，如果两个参数都是`null,`，它将返回`true`，如果其中只有一个是`null`，它将返回`false`。

这真的很方便。假设我们想在我们的`Person`类中添加一个可选的出生日期:

```java
public Person(String firstName, String lastName, LocalDate birthDate) {
    this(firstName, lastName);
    this.birthDate = birthDate;
}
```

然后我们必须更新我们的`equals()`方法，但是使用`null`处理。我们可以通过将条件添加到我们的`equals()`方法中来做到这一点:

```java
birthDate == null ? that.birthDate == null : birthDate.equals(that.birthDate);
```

然而，如果我们在类中添加了太多的可空字段，就会变得非常混乱。在我们的`equals()`实现中使用`Objects#equals`方法要干净得多，并且提高了可读性:

```java
Objects.equals(birthDate, that.birthDate);
```

## `5\. Comparable`界面

比较逻辑也可以用于按特定顺序放置对象。**[`Comparable`接口](https://web.archive.org/web/20220628145611/https://docs.oracle.com/en/java/javase/14/docs/api/java.base/java/lang/Comparable.html)允许我们通过确定一个对象是大于、等于还是小于另一个来定义对象**之间的排序。

`Comparable`接口是泛型的，只有一个方法`compareTo()`，它接受泛型类型的参数并返回一个`int`。如果`this`小于参数，返回值为负，如果相等，返回值为 0，否则为正。

比方说，在我们的`Person`类中，我们希望通过对象的姓氏来比较`Person`对象:

```java
public class Person implements Comparable<Person> {
    //...

    @Override
    public int compareTo(Person o) {
        return this.lastName.compareTo(o.lastName);
    }
}
```

如果调用的`Person`的姓氏比`this`的姓氏大，那么`compareTo()`方法将返回负的 *int* ，如果相同的姓氏，则返回零，否则返回正的。

要了解更多信息，请看我们关于这个话题的[文章。](/web/20220628145611/https://www.baeldung.com/java-comparator-comparable)

## `6\. Comparator` 界面

[`Comparator`接口](https://web.archive.org/web/20220628145611/https://docs.oracle.com/en/java/javase/14/docs/api/java.base/java/util/Comparator.html)是泛型的，并且有一个`compare`方法，该方法接受该泛型类型的两个参数并返回一个`integer`。我们已经在前面的`Comparable`接口中看到了这种模式。

`Comparator`类似；然而，它与类的定义是分开的。因此，**我们可以为一个类定义尽可能多的`Comparators`，这里我们只能提供一个`Comparable`实现。**

假设我们有一个以表格视图显示人员的 web 页面，我们希望为用户提供按名字而不是姓氏排序的可能性。如果我们还想保持我们当前的实现，这对于`Comparable`是不可能的，但是我们可以实现我们自己的`Comparators`。

让我们创建一个`Person` `Comparator`来仅通过他们的名字来比较他们:

```java
Comparator<Person> compareByFirstNames = Comparator.comparing(Person::getFirstName);
```

现在让我们对使用`Comparator`的`List`人进行排序:

```java
Person joe = new Person("Joe", "Portman");
Person allan = new Person("Allan", "Dale");

List<Person> people = new ArrayList<>();
people.add(joe);
people.add(allan);

people.sort(compareByFirstNames);

assertThat(people).containsExactly(allan, joe);
```

在我们的`compareTo()` 实现中，还可以使用`Comparator`接口上的其他方法:

```java
@Override
public int compareTo(Person o) {
    return Comparator.comparing(Person::getLastName)
      .thenComparing(Person::getFirstName)
      .thenComparing(Person::getBirthDate, Comparator.nullsLast(Comparator.naturalOrder()))
      .compare(this, o);
}
```

在这种情况下，我们首先比较姓氏，然后是名字。接下来，我们比较出生日期，但是由于它们是可空的，我们必须说明如何处理这个问题。为了做到这一点，我们给出了第二个参数，即应该根据它们的自然顺序进行比较，最后是`null` 值。

## 7.Apache common(Apache 公共)

让我们来看看 [Apache Commons 库](/web/20220628145611/https://www.baeldung.com/java-commons-lang-3)。首先，我们来导入 [Maven 依赖](https://web.archive.org/web/20220628145611/https://search.maven.org/search?q=g:org.apache.commons%20AND%20a:commons-lang3):

```java
<dependency>
    <groupId>org.apache.commons</groupId>
    <artifactId>commons-lang3</artifactId>
    <version>3.12.0</version>
</dependency>
```

### `7.1\.` `ObjectUtils#notEqual`法

首先说一下`ObjectUtils#notEqual`法。根据它们自己的`equals()`方法实现，需要两个`Object` 参数来确定它们是否不相等。它还处理`null` 值。

让我们重复使用我们的`String`例子:

```java
String a = new String("Hello!");
String b = new String("Hello World!");

assertThat(ObjectUtils.notEqual(a, b)).isTrue(); 
```

需要注意的是，`ObjectUtils`有一个`equals()`方法。然而，自从 Java 7,`Objects#equals`出现后，这种做法就被否决了

### `7.2\. ObjectUtils#compare`方法

现在让我们比较一下对象顺序和`ObjectUtils#compare`方法。**这是一个泛型方法，它接受该泛型类型的两个`Comparable`参数，并返回一个`Integer`。**

让我们再次使用`Strings`来看看:

```java
String first = new String("Hello!");
String second = new String("How are you?");

assertThat(ObjectUtils.compare(first, second)).isNegative();
```

默认情况下，该方法通过认为值更大来处理`null`值。它还提供了一个重载版本，提供了一个`boolean` 参数来反转该行为，并认为它们不那么重要。

## 8.番石榴

我们来看看[番石榴](https://web.archive.org/web/20220628145611/https://guava.dev/)。首先，我们来导入[的依赖关系](https://web.archive.org/web/20220628145611/https://search.maven.org/search?q=g:com.google.guava%20AND%20a:guava):

```java
<dependency>
    <groupId>com.google.guava</groupId>
    <artifactId>guava</artifactId>
    <version>31.0.1-jre</version>
</dependency>
```

### `8.1\. Objects#equal`方法

**类似于 Apache Commons 库，Google 为我们提供了一种判断两个对象是否相等的方法，`Objects#equal`。尽管它们有不同的实现，但它们返回相同的结果:**

```java
String a = new String("Hello!");
String b = new String("Hello!");

assertThat(Objects.equal(a, b)).isTrue();
```

尽管它没有被标记为不推荐使用，但是这个方法的 JavaDoc 说它应该被认为是不推荐使用的，因为 Java 7 提供了`Objects#equals`方法。

### 8.2.比较方法

Guava 库没有提供比较两个对象的方法(我们将在下一节中看到如何实现这一点)，但是**提供了比较原始值的方法**。让我们看看`Ints`助手类，看看它的`compare()`方法是如何工作的:

```java
assertThat(Ints.compare(1, 2)).isNegative();
```

和往常一样，如果第一个参数小于、等于或大于第二个参数，它将返回一个可能为负、零或正的`integer`。除了`bytes`之外，所有的基本类型都有类似的方法。

### `8.3\. ComparisonChain` 阶级

最后，Guava 库提供了`ComparisonChain`类，允许我们通过一系列比较来比较两个对象。我们可以很容易地通过名字和姓氏来比较两个`Person`对象:

```java
Person natalie = new Person("Natalie", "Portman");
Person joe = new Person("Joe", "Portman");

int comparisonResult = ComparisonChain.start()
  .compare(natalie.getLastName(), joe.getLastName())
  .compare(natalie.getFirstName(), joe.getFirstName())
  .result();

assertThat(comparisonResult).isPositive();
```

底层的比较是使用`compareTo()`方法实现的，所以传递给`compare()` 方法的参数必须是原语或者是`Comparable`

## 9.结论

在本文中，我们学习了在 Java 中比较对象的不同方法。我们研究了相同性、平等性和有序性之间的区别。我们还研究了 Apache Commons 和 Guava 库中的相应特性。

像往常一样，这篇文章的完整代码可以在 GitHub 上找到[。](https://web.archive.org/web/20220628145611/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-lang-2)