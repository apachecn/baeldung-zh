# Java 14 记录关键字

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-record-keyword>

## 1.介绍

在对象之间传递不可变数据是许多 Java 应用程序中最常见但又最普通的任务之一。

在 Java 14 之前，这需要创建一个带有样板字段和方法的类，容易出现小错误和混乱的意图。

随着 Java 14 的发布，我们现在可以使用记录来解决这些问题。

在本教程中，**我们将了解记录的基础知识**、**包括它们的用途、**、**生成方法和定制技术**。

## 2.目的

通常，我们编写类只是为了保存数据，比如数据库结果、查询结果或来自服务的信息。

在很多情况下，这个数据是[不可变的](/web/20221018104148/https://www.baeldung.com/java-immutable-object)，因为**的不变性保证了数据的[有效性，而不需要同步](/web/20221018104148/https://www.baeldung.com/java-immutable-object#benefits-of-immutability)T5。**

为此，我们使用以下内容创建数据类:

1.  `private`、`final`字段为每条数据
2.  每个字段的 getter
3.  `public`每个字段都有相应参数的构造函数
4.  `equals`方法，当所有字段匹配时，该方法为同一类的对象返回`true`
5.  `hashCode`当所有字段匹配时返回相同值的方法
6.  `toString`方法，包括类名和每个字段的名称及其对应的值

例如，我们可以创建一个带有名称和地址的简单的`Person`数据类:

```java
public class Person {

    private final String name;
    private final String address;

    public Person(String name, String address) {
        this.name = name;
        this.address = address;
    }

    @Override
    public int hashCode() {
        return Objects.hash(name, address);
    }

    @Override
    public boolean equals(Object obj) {
        if (this == obj) {
            return true;
        } else if (!(obj instanceof Person)) {
            return false;
        } else {
            Person other = (Person) obj;
            return Objects.equals(name, other.name)
              && Objects.equals(address, other.address);
        }
    }

    @Override
    public String toString() {
        return "Person [name=" + name + ", address=" + address + "]";
    }

    // standard getters
}
```

虽然这实现了我们的目标，但有两个问题:

1.  有很多样板代码
2.  我们模糊了类的目的:用名字和地址来表示一个人

在第一种情况下，我们必须为每个数据类重复相同的繁琐过程，单调地为每条数据创建一个新字段；创建`equals`、`hashCode`和`toString`方法；并创建一个接受每个字段的构造函数。

虽然 ide 可以自动生成许多这样的类，但是当我们添加一个新字段时，**它们不能自动更新我们的类。例如，如果我们添加一个新字段，我们必须更新我们的`equals`方法来合并这个字段。**

在第二种情况下，**额外的代码掩盖了我们的类只是一个数据类**，它有两个`String`字段，`name`和`address`。

更好的方法是显式声明我们的类是一个数据类。

## 3.基础知识

从 JDK 14 开始，我们可以用记录代替重复的数据类。记录是不可变的数据类，只需要字段的类型和名称。

Java 编译器生成`equals`、`hashCode`和`toString`方法，以及`private,`、`final`字段和`public`构造函数。

为了创建一个`Person`记录，我们将使用`record`关键字:

```java
public record Person (String name, String address) {}
```

### 3.1.构造器

使用记录，为我们生成一个公共构造函数，每个字段都有一个参数。

在我们的`Person`记录中，等价的构造函数是:

```java
public Person(String name, String address) {
    this.name = name;
    this.address = address;
}
```

此构造函数可以像类一样用于从记录中实例化对象:

```java
Person person = new Person("John Doe", "100 Linda Ln.");
```

### 3.2.吸气剂

我们还免费获得公共 getters 方法，它们的名称与我们的字段名称相匹配。

在我们的`Person`记录中，这意味着一个`name()`和`address()` getter:

```java
@Test
public void givenValidNameAndAddress_whenGetNameAndAddress_thenExpectedValuesReturned() {
    String name = "John Doe";
    String address = "100 Linda Ln.";

    Person person = new Person(name, address);

    assertEquals(name, person.name());
    assertEquals(address, person.address());
}
```

### 3.3.`equals`

另外，为我们生成了一个`equals`方法。

**如果提供的对象属于同一类型，并且其所有字段的值都匹配:**，则该方法返回`true`

```java
@Test
public void givenSameNameAndAddress_whenEquals_thenPersonsEqual() {
    String name = "John Doe";
    String address = "100 Linda Ln.";

    Person person1 = new Person(name, address);
    Person person2 = new Person(name, address);

    assertTrue(person1.equals(person2));
}
```

如果两个`Person`实例之间的任何字段不同，`equals`方法将返回`false`。

### 3.4.`hashCode`

类似于我们的`equals`方法，也为我们生成了相应的`hashCode`方法。

**如果两个对象的所有字段值都匹配**(由于[生日悖论](https://web.archive.org/web/20221018104148/https://en.wikipedia.org/wiki/Birthday_problem)导致的冲突除外) **:** ，我们的`hashCode`方法为两个`Person`对象返回相同的值

```java
@Test
public void givenSameNameAndAddress_whenHashCode_thenPersonsEqual() {
    String name = "John Doe";
    String address = "100 Linda Ln.";

    Person person1 = new Person(name, address);
    Person person2 = new Person(name, address);

    assertEquals(person1.hashCode(), person2.hashCode());
} 
```

如果任何字段值不同，`hashCode`值也会不同。

### 3.5.`toString`

最后，我们还收到一个 **`toString`方法，该方法产生一个包含记录名称的字符串，后跟每个字段的名称及其在方括号**中对应的值。

因此，用名称`“John Doe”`和地址`“100 Linda Ln.`实例化一个`Person`会导致下面的`toString`结果:

```java
Person[name=John Doe, address=100 Linda Ln.]
```

## 4.构造器

虽然为我们生成了一个公共构造函数，但是我们仍然可以定制我们的构造函数实现。

该定制旨在用于验证，并应尽可能保持简单。

例如，我们可以使用下面的构造函数实现来确保提供给我们的`Person`记录的`name`和`address`不是`null`:

```java
public record Person(String name, String address) {
    public Person {
        Objects.requireNonNull(name);
        Objects.requireNonNull(address);
    }
}
```

我们还可以通过提供不同的参数列表来创建具有不同参数的新构造函数:

```java
public record Person(String name, String address) {
    public Person(String name) {
        this(name, "Unknown");
    }
}
```

与类构造函数一样，**字段可以使用`this`关键字**来引用(例如，`this.name`和`this.address`)，并且**参数匹配字段**的名称(即，`name`和`address`)。

注意**创建一个与生成的公共构造函数具有相同参数的构造函数是有效的，但是这需要手动初始化每个字段**:

```java
public record Person(String name, String address) {
    public Person(String name, String address) {
        this.name = name;
        this.address = address;
    }
}
```

此外，**声明一个无参数构造函数和一个参数列表与生成的构造函数匹配的构造函数会导致编译错误**。

因此，下面的代码不会被编译:

```java
public record Person(String name, String address) {
    public Person {
        Objects.requireNonNull(name);
        Objects.requireNonNull(address);
    }

    public Person(String name, String address) {
        this.name = name;
        this.address = address;
    }
}
```

## 5.静态变量和方法

与常规 Java 类一样，**我们也可以在记录中包含静态变量和方法**。

我们使用与类相同的语法声明静态变量:

```java
public record Person(String name, String address) {
    public static String UNKNOWN_ADDRESS = "Unknown";
}
```

同样，我们使用与类相同的语法声明静态方法:

```java
public record Person(String name, String address) {
    public static Person unnamed(String address) {
        return new Person("Unnamed", address);
    }
}
```

然后，我们可以使用记录的名称来引用静态变量和静态方法:

```java
Person.UNKNOWN_ADDRESS
Person.unnamed("100 Linda Ln.");
```

## 6.结论

在本文中，我们研究了 Java 14 中引入的关键字`record`，包括基本概念和复杂性。

使用记录及其编译器生成的方法，我们可以减少样板代码并提高不可变类的可靠性。

这篇文章的代码和例子可以在 GitHub 上找到[。](https://web.archive.org/web/20221018104148/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-14)