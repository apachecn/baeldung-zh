# Hibernate/JPA 中的标识符概述

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/hibernate-identifiers>

## 1。概述

Hibernate 中的标识符表示实体的主键。这意味着值是唯一的，以便它们可以标识特定的实体，它们不为空，并且不会被修改。

Hibernate 提供了几种不同的方法来定义标识符。在本文中，我们将回顾使用库映射实体 id 的每种方法。

## 2。简单标识符

定义标识符最直接的方法是使用`@Id`注释。

简单 id 使用`@Id`映射到这些类型之一的单个属性:Java 原语和原语包装器类型、`String`、`Date`、`BigDecimal`和`BigInteger`。

让我们看一个用主键类型`long`定义实体的简单例子:

```java
@Entity
public class Student {

    @Id
    private long studentId;

    // standard constructor, getters, setters
}
```

## 3。生成的标识符

如果我们想自动生成主键值，**我们可以添加`@GeneratedValue`注释。**

这可以使用四种生成类型:自动、标识、序列和表。

如果我们没有明确指定一个值，生成类型默认为 AUTO。

### 3.1。`AUTO`一代人

如果我们使用默认的生成类型，持久性提供者将根据主键属性的类型来确定值。这个类型可以是数值型或者`UUID`。

对于数值，生成基于序列或表格生成器，而`UUID`值将使用`UUIDGenerator`。

让我们首先使用自动生成策略映射一个实体主键:

```java
@Entity
public class Student {

    @Id
    @GeneratedValue
    private long studentId;

    // ...
}
```

在这种情况下，主键值在数据库级别是唯一的。

**现在我们来看看`UUIDGenerator`，它是在 Hibernate 5 中引入的。**

为了使用这个特性，我们只需要用`@GeneratedValue`注释声明一个类型为`UUID`的 id:

```java
@Entity
public class Course {

    @Id
    @GeneratedValue
    private UUID courseId;

    // ...
}
```

Hibernate 将生成一个“8d D5 f 315-9788-4d 00-87 bb-10 eed 9 eff 566”形式的 id。

### 3.2。`IDENTITY`一代人

这种类型的生成依赖于`IdentityGenerator`，它期望数据库中的`identity`列生成值。这意味着它们会自动递增。

要使用这种生成类型，我们只需设置`strategy`参数:

```java
@Entity
public class Student {

    @Id
    @GeneratedValue (strategy = GenerationType.IDENTITY)
    private long studentId;

    // ...
}
```

需要注意的一点是，身份生成禁止批量更新。

### 3.3。`SEQUENCE`一代人

为了使用基于序列的 id，Hibernate 提供了`SequenceStyleGenerator`类。

如果我们的数据库支持，这个生成器使用序列。如果不支持，它会切换到表生成。

为了定制序列名称，我们可以使用带有`SequenceStyleGenerator strategy`的`@GenericGenerator`注释:

```java
@Entity
public class User {
    @Id
    @GeneratedValue(generator = "sequence-generator")
    @GenericGenerator(
      name = "sequence-generator",
      strategy = "org.hibernate.id.enhanced.SequenceStyleGenerator",
      parameters = {
        @Parameter(name = "sequence_name", value = "user_sequence"),
        @Parameter(name = "initial_value", value = "4"),
        @Parameter(name = "increment_size", value = "1")
        }
    )
    private long userId;

    // ...
}
```

在本例中，我们还为序列设置了一个初始值，这意味着主键生成将从 4 开始。

`SEQUENCE`是 Hibernate 文档推荐的生成类型。

**每个序列生成的值都是唯一的。**如果我们不指定序列名，Hibernate 将对不同的类型重用相同的`hibernate_sequence` 。

### 3.4。表格生成

`TableGenerator`使用一个底层数据库表，该表保存标识符生成值的片段。

让我们使用`@TableGenerator`注释定制表名:

```java
@Entity
public class Department {
    @Id
    @GeneratedValue(strategy = GenerationType.TABLE, 
      generator = "table-generator")
    @TableGenerator(name = "table-generator", 
      table = "dep_ids", 
      pkColumnName = "seq_id", 
      valueColumnName = "seq_value")
    private long depId;

    // ...
}
```

在这个例子中，我们可以看到我们还可以定制其他属性，比如`pkColumnName`和`valueColumnName`。

然而，这种方法的缺点是它不能很好地扩展，并且会对性能产生负面影响。

**总而言之，这四种生成类型将导致生成相似的值，但使用不同的数据库机制。**

### 3.5。定制生成器

假设我们不想使用任何现成的策略。为了做到这一点，**我们可以通过实现`IdentifierGenerator` 接口来定义我们的定制生成器。**

我们将创建一个生成器来构建包含一个前缀和一个数字的标识符:

```java
public class MyGenerator 
  implements IdentifierGenerator, Configurable {

    private String prefix;

    @Override
    public Serializable generate(
      SharedSessionContractImplementor session, Object obj) 
      throws HibernateException {

        String query = String.format("select %s from %s", 
            session.getEntityPersister(obj.getClass().getName(), obj)
              .getIdentifierPropertyName(),
            obj.getClass().getSimpleName());

        Stream ids = session.createQuery(query).stream();

        Long max = ids.map(o -> o.replace(prefix + "-", ""))
          .mapToLong(Long::parseLong)
          .max()
          .orElse(0L);

        return prefix + "-" + (max + 1);
    }

    @Override
    public void configure(Type type, Properties properties, 
      ServiceRegistry serviceRegistry) throws MappingException {
        prefix = properties.getProperty("prefix");
    }
}
```

在这个例子中，**我们从`IdentifierGenerator`接口覆盖了`generate()`方法。**

首先，我们希望从表单`prefix-XX`的现有主键中找到最大的数字。然后，我们将找到的最大数字加 1，并追加`prefix`属性以获得新生成的 id 值。

我们的类还实现了`Configurable`接口，这样我们就可以在`configure()`方法中设置`prefix`属性值。

接下来，让我们将这个定制生成器添加到一个实体中。

为此，**我们可以使用带有`strategy`参数的`@GenericGenerator`注释，该参数包含我们的生成器类**的完整类名:

```java
@Entity
public class Product {

    @Id
    @GeneratedValue(generator = "prod-generator")
    @GenericGenerator(name = "prod-generator", 
      parameters = @Parameter(name = "prefix", value = "prod"), 
      strategy = "com.baeldung.hibernate.pojo.generator.MyGenerator")
    private String prodId;

    // ...
}
```

另外，请注意，我们已经将前缀参数设置为“prod”。

让我们看一个快速的 JUnit 测试，以便更清楚地理解生成的 id 值:

```java
@Test
public void whenSaveCustomGeneratedId_thenOk() {
    Product product = new Product();
    session.save(product);
    Product product2 = new Product();
    session.save(product2);

    assertThat(product2.getProdId()).isEqualTo("prod-2");
}
```

这里使用“prod”前缀生成的第一个值是“prod-1”，后面是“prod-2”。

## 4。复合标识符

除了我们目前看到的简单标识符，Hibernate 还允许我们定义复合标识符。

复合 id 由具有一个或多个持久属性的主键类表示。

**主键类必须满足几个条件**:

*   应该使用`@EmbeddedId`或`@IdClass`注释来定义。
*   它应该是公共的、可序列化的，并且有一个公共的无参数构造函数。
*   最后，它应该实现`equals()`和`hashCode()`方法。

类的属性可以是基本的、复合的或多元素的，同时避免使用集合和`OneToOne`属性。

### 4.1。`@EmbeddedId`

现在让我们看看如何使用`@EmbeddedId`定义一个 id。

首先，我们需要一个用`@Embeddable`注释的主键类:

```java
@Embeddable
public class OrderEntryPK implements Serializable {

    private long orderId;
    private long productId;

    // standard constructor, getters, setters
    // equals() and hashCode() 
}
```

接下来，我们可以使用@ `EmbeddedId`将类型为`OrderEntryPK`的 id 添加到实体中:

```java
@Entity
public class OrderEntry {

    @EmbeddedId
    private OrderEntryPK entryId;

    // ...
}
```

让我们看看如何使用这种类型的复合 id 来设置实体的主键:

```java
@Test
public void whenSaveCompositeIdEntity_thenOk() {
    OrderEntryPK entryPK = new OrderEntryPK();
    entryPK.setOrderId(1L);
    entryPK.setProductId(30L);

    OrderEntry entry = new OrderEntry();
    entry.setEntryId(entryPK);
    session.save(entry);

    assertThat(entry.getEntryId().getOrderId()).isEqualTo(1L);
}
```

这里的`OrderEntry`对象有一个由两个属性组成的`OrderEntryPK`主 id:`orderId`和`productId`。

### 4.2。`@IdClass`

`@IdClass`注释类似于`@EmbeddedId`。与`@IdClass`的区别在于属性是在主实体类中定义的，每个属性都使用`@Id`。主键类看起来和以前一样。

让我们用一个`@IdClass`来重写`OrderEntry`的例子:

```java
@Entity
@IdClass(OrderEntryPK.class)
public class OrderEntry {
    @Id
    private long orderId;
    @Id
    private long productId;

    // ...
}
```

然后我们可以直接在`OrderEntry`对象上设置 id 值:

```java
@Test
public void whenSaveIdClassEntity_thenOk() {        
    OrderEntry entry = new OrderEntry();
    entry.setOrderId(1L);
    entry.setProductId(30L);
    session.save(entry);

    assertThat(entry.getOrderId()).isEqualTo(1L);
}
```

注意，对于这两种类型的复合 id，主键类也可以包含`@ManyToOne`属性。

Hibernate 还允许定义由`@ManyToOne`关联和`@Id`注释组成的主键。在这种情况下，实体类也应该满足主键类的条件。

然而，这种方法的缺点是实体对象和标识符之间没有分离。

## 5。派生标识符

使用`@MapsId`注释从实体的关联中获得派生标识符。

首先，让我们创建一个`UserProfile`实体，它从与`User`实体的一对一关联中获得其 id:

```java
@Entity
public class UserProfile {

    @Id
    private long profileId;

    @OneToOne
    @MapsId
    private User user;

    // ...
}
```

接下来，让我们验证一个`UserProfile`实例与其关联的`User`实例具有相同的 id:

```java
@Test
public void whenSaveDerivedIdEntity_thenOk() {        
    User user = new User();
    session.save(user);

    UserProfile profile = new UserProfile();
    profile.setUser(user);
    session.save(profile);

    assertThat(profile.getProfileId()).isEqualTo(user.getUserId());
}
```

## 6。结论

在本文中，我们看到了在 Hibernate 中定义标识符的多种方法。

这些例子的完整源代码可以在 GitHub 上找到[。](https://web.archive.org/web/20221205203204/https://github.com/eugenp/tutorials/tree/master/persistence-modules/hibernate5)