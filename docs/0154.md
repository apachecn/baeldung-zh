# Hibernate 继承映射

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/hibernate-inheritance>

## 1。概述

关系数据库没有直接的方法将类层次映射到数据库表。

为了解决这个问题，JPA 规范提供了几种策略:

*   `MappedSuperclass`–父类不能是实体
*   单个表–来自具有共同祖先的不同类的实体放在单个表中。
*   连接表——每个类都有自己的表，查询子类实体需要连接这些表。
*   每个类一个表–一个类的所有属性都在它的表中，所以不需要连接。

每种策略都会产生不同的数据库结构。

实体继承意味着当查询超类时，我们可以使用多态查询来检索所有的子类实体。

因为 Hibernate 是一个 JPA 实现，所以它包含了上述所有特性以及一些与继承相关的 Hibernate 特有的特性。

在接下来的部分中，我们将更详细地讨论可用的策略。

## 2。`MappedSuperclass`

使用`MappedSuperclass`策略，继承只在类中明显，而在实体模型中不明显。

让我们首先创建一个代表父类的`Person`类:

```
@MappedSuperclass
public class Person {

    @Id
    private long personId;
    private String name;

    // constructor, getters, setters
}
```

**注意，这个类不再有`@Entity`注释**，因为它不会自己保存在数据库中。

接下来，让我们添加一个`Employee`子类:

```
@Entity
public class MyEmployee extends Person {
    private String company;
    // constructor, getters, setters 
}
```

在数据库中，这将对应于一个有三列的`MyEmployee`表，用于子类的声明和继承字段。

如果我们使用这种策略，祖先不能包含与其他实体的关联。

## 3。单桌

**单表策略为每个类层次结构创建一个表。如果我们没有明确指定，JPA 也会默认选择这个策略。**

我们可以通过向超类添加`@Inheritance`注释来定义我们想要使用的策略:

```
@Entity
@Inheritance(strategy = InheritanceType.SINGLE_TABLE)
public class MyProduct {
    @Id
    private long productId;
    private String name;

    // constructor, getters, setters
}
```

实体的标识符也在超类中定义。

然后我们可以添加子类实体:

```
@Entity
public class Book extends MyProduct {
    private String author;
}
```

```
@Entity
public class Pen extends MyProduct {
    private String color;
}
```

### 3.1。鉴别器值

因为所有实体的记录都在同一个表中， **Hibernate 需要一种方法来区分它们。**

**默认情况下，这是通过一个名为`DTYPE`** 的鉴别器列完成的，该列以实体的名称作为值。

要定制鉴别器列，我们可以使用`@DiscriminatorColumn`注释:

```
@Entity(name="products")
@Inheritance(strategy = InheritanceType.SINGLE_TABLE)
@DiscriminatorColumn(name="product_type", 
  discriminatorType = DiscriminatorType.INTEGER)
public class MyProduct {
    // ...
}
```

这里我们选择通过一个名为`product_type`的`integer`列来区分`MyProduct`子类实体。

接下来，我们需要告诉 Hibernate 每个子类记录对`product_type`列有什么值:

```
@Entity
@DiscriminatorValue("1")
public class Book extends MyProduct {
    // ...
}
```

```
@Entity
@DiscriminatorValue("2")
public class Pen extends MyProduct {
    // ...
}
```

Hibernate 添加了注释可以采用的另外两个预定义值— `null`和`not null`:

*   `@DiscriminatorValue(“null”)`表示任何没有鉴别器值的行将被映射到带有此注释的实体类；这可以应用于层次结构的根类。
*   `@DiscriminatorValue(“not null”)`–任何鉴别器值与实体定义相关的鉴别器值都不匹配的行将被映射到带有此注释的类。

除了列，我们还可以使用特定于 Hibernate 的`@DiscriminatorFormula`注释来确定不同的值:

```
@Entity
@Inheritance(strategy = InheritanceType.SINGLE_TABLE)
@DiscriminatorFormula("case when author is not null then 1 else 2 end")
public class MyProduct { ... }
```

这种策略具有多态查询性能的优势，因为在查询父实体时只需要访问一个表。

另一方面，这也意味着**我们不能再对子类**的实体属性使用`NOT NULL`约束。

## 4。连接表

使用这种策略，层次结构中的每个类都被映射到它的表中。在所有表格中重复出现的唯一一列是标识符，它将在需要时用于连接它们。

让我们创建一个使用这种策略的超类:

```
@Entity
@Inheritance(strategy = InheritanceType.JOINED)
public class Animal {
    @Id
    private long animalId;
    private String species;

    // constructor, getters, setters 
}
```

然后我们可以简单地定义一个子类:

```
@Entity
public class Pet extends Animal {
    private String name;

    // constructor, getters, setters
}
```

两个表都有一个`animalId`标识符列。

`Pet`实体的主键对其父实体的主键也有一个外键约束。

要定制这个列，我们可以添加`@PrimaryKeyJoinColumn`注释:

```
@Entity
@PrimaryKeyJoinColumn(name = "petId")
public class Pet extends Animal {
    // ...
}
```

**这种继承映射方法的缺点是检索实体需要表之间的连接**，这会导致大量记录的性能降低。

当查询父类时，连接的数量更大，因为它将与每个相关的子类连接，所以性能更可能受到影响，因为我们希望检索的记录在层次结构中的位置越高。

## 5。每类表格

**每类表策略将每个实体映射到它的表，该表包含实体的所有属性，包括继承的属性。**

产生的模式类似于使用@MappedSuperclass 的模式。但是每个类的表确实会为父类定义实体，结果允许关联和多态查询。

要使用这种策略，我们只需要向基类添加`@Inheritance`注释:

```
@Entity
@Inheritance(strategy = InheritanceType.TABLE_PER_CLASS)
public class Vehicle {
    @Id
    private long vehicleId;

    private String manufacturer;

    // standard constructor, getters, setters
}
```

然后我们可以用标准的方式创建子类。

这与仅仅映射每个没有继承的实体没有什么不同。当查询基类时，这种区别是明显的，通过在后台使用一个`UNION`语句，基类也将返回所有子类记录。

**使用`UNION`也会导致选择这种策略时表现不佳。**另一个问题是我们不能再使用身份密钥生成。

## 6。多态查询

如上所述，查询基类也将检索所有子类实体。

让我们通过一个 JUnit 测试来看看这种行为:

```
@Test
public void givenSubclasses_whenQuerySuperclass_thenOk() {
    Book book = new Book(1, "1984", "George Orwell");
    session.save(book);
    Pen pen = new Pen(2, "my pen", "blue");
    session.save(pen);

    assertThat(session.createQuery("from MyProduct")
      .getResultList()).hasSize(2);
}
```

在这个例子中，我们已经创建了两个`Book`和`Pen`对象，然后查询它们的超类`MyProduct`来验证我们将检索两个对象。

Hibernate 还可以查询不是实体而是由实体类扩展或实现的接口或基类。

让我们用我们的`@MappedSuperclass`例子来看一个 JUnit 测试:

```
@Test
public void givenSubclasses_whenQueryMappedSuperclass_thenOk() {
    MyEmployee emp = new MyEmployee(1, "john", "baeldung");
    session.save(emp);

    assertThat(session.createQuery(
      "from com.baeldung.hibernate.pojo.inheritance.Person")
      .getResultList())
      .hasSize(1);
}
```

注意，这也适用于任何超类或接口，不管它是不是`@MappedSuperclass`。与通常的 HQL 查询不同的是，我们必须使用完全限定名，因为它们不是 Hibernate 管理的实体。

如果我们不想让这种类型的查询返回一个子类，我们只需要将 Hibernate `@Polymorphism`注释添加到它的定义中，类型为`EXPLICIT`:

```
@Entity
@Polymorphism(type = PolymorphismType.EXPLICIT)
public class Bag implements Item { ...}
```

这种情况下，查询`Items`时，不会返回`Bag`条记录。

## 7。结论

在本文中，我们展示了 Hibernate 中映射继承的不同策略。

这些例子的完整源代码可以在 GitHub 上找到[。](https://web.archive.org/web/20221101231605/https://github.com/eugenp/tutorials/tree/master/persistence-modules/hibernate-mapping)