# hibernate @ not null vs @ Column(nullable = false)

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/hibernate-notnull-vs-nullable>

## 1.介绍

乍看之下，**似乎`@NotNull`和`@Column(nullable = false)`注释服务于同一个目的**，并且可以互换使用。然而，正如我们很快会看到的，这并不完全正确。

即使在 JPA 实体上使用时，**这两种方法本质上都阻止了在底层数据库中存储`null`值，但是这两种方法之间还是有很大的不同。**

在这个快速教程中，我们将比较`@NotNull`和`@Column(nullable = false)`约束。

## 2.属国

对于所有给出的例子，我们将使用一个简单的 [Spring Boot](/web/20220626075530/https://www.baeldung.com/spring-boot) 应用程序。

下面是`pom.xml`文件的相关部分，显示了所需的依赖关系:

```java
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-data-jpa</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-validation</artifactId>
    </dependency>
    <dependency>
        <groupId>com.h2database</groupId>
        <artifactId>h2</artifactId>
    </dependency>
</dependencies>
```

### 2.1.样本实体

让我们也定义一个非常简单的实体，我们将在整个教程中使用:

```java
@Entity
public class Item {

    @Id
    @GeneratedValue
    private Long id;

    private BigDecimal price;
}
```

## 3.`@NotNull`注解

**注释`@NotNull`在 [Bean 验证](/web/20220626075530/https://www.baeldung.com/javax-validation)规范**中定义。这意味着它的使用不仅限于实体。相反，我们也可以在任何其他 bean 上使用`@NotNull`。

让我们继续使用我们的用例，将`@NotNull`注释添加到`Item`的`price`字段:

```java
@Entity
public class Item {

    @Id
    @GeneratedValue
    private Long id;

    @NotNull
    private BigDecimal price;
}
```

现在，让我们尝试用`null` `price`持久化一个项目:

```java
@SpringBootTest
public class ItemIntegrationTest {

    @Autowired
    private ItemRepository itemRepository;

    @Test
    public void shouldNotAllowToPersistNullItemsPrice() {
        itemRepository.save(new Item());
    }
}
```

让我们看看 Hibernate 的输出:

```java
2019-11-14 12:31:15.070 ERROR 10980 --- [ main] o.h.i.ExceptionMapperStandardImpl : 
HHH000346: Error during managed flush [Validation failed for classes 
[com.baeldung.h2db.springboot.models.Item] during persist time for groups 
[javax.validation.groups.Default,] List of constraint violations:[
ConstraintViolationImpl{interpolatedMessage='must not be null', propertyPath=price, rootBeanClass=class 
com.baeldung.h2db.springboot.models.Item, 
messageTemplate='{javax.validation.constraints.NotNull.message}'}]]

(...)

Caused by: javax.validation.ConstraintViolationException: Validation failed for classes 
[com.baeldung.h2db.springboot.models.Item] during persist time for groups 
[javax.validation.groups.Default,] List of constraint violations:[
ConstraintViolationImpl{interpolatedMessage='must not be null', propertyPath=price, rootBeanClass=class 
com.baeldung.h2db.springboot.models.Item, 
messageTemplate='{javax.validation.constraints.NotNull.message}'}]
```

我们可以看到，**在这种情况下，我们的系统抛出了** `**javax.validation.ConstraintViolationException**.`

需要注意的是，Hibernate 没有触发 SQL insert 语句。因此，无效数据不会保存到数据库中。

这是因为 pre-persist 实体生命周期事件在将查询发送到数据库之前触发了 bean 验证。

### 3.1.模式生成

在上一节中，我们已经介绍了`@NotNull`验证是如何工作的。

现在让我们看看如果让 Hibernate 为我们生成数据库模式会发生什么。

出于这个原因，我们将在我们的`application.properties`文件中设置几个属性:

```java
spring.jpa.hibernate.ddl-auto=create-drop
spring.jpa.show-sql=true
```

如果我们现在启动应用程序，我们将看到 DDL 语句:

```java
create table item (
   id bigint not null,
    price decimal(19,2) not null,
    primary key (id)
)
```

令人惊讶的是， **Hibernate 自动将`not null`约束添加到`price`列定义中。**

这怎么可能呢？

结果是，Hibernate 将应用于实体的 bean 验证注释翻译成 DDL 模式元数据。

这很方便，也很有意义。如果我们将`@NotNull`应用于实体，我们很可能也想创建相应的数据库列`not null`。

然而，如果出于某种原因，我们希望**禁用这个休眠特性，我们需要做的就是将`hibernate.validator.apply_to_ddl`属性设置为`false.`**

为了测试这一点，让我们更新我们的`application.properties`:

```java
spring.jpa.hibernate.ddl-auto=create-drop
spring.jpa.show-sql=true
spring.jpa.properties.hibernate.validator.apply_to_ddl=false
```

让我们运行应用程序并查看 DDL 语句:

```java
create table item (
   id bigint not null,
    price decimal(19,2),
    primary key (id)
)
```

不出所料，**这次 Hibernate 没有给`price`列**添加`not null`约束。

## 4.`@Column(nullable = false)`注解

**`@Column`注释被定义为 [Java 持久性 API](https://web.archive.org/web/20220626075530/https://jcp.org/en/jsr/detail?id=338) 规范**的一部分。

它主要用于 DDL 模式元数据的生成。这意味着**如果我们让 Hibernate 自动生成数据库模式，它会将`not null`约束应用到特定的数据库列**。

让我们用`@Column(nullable = false)`更新我们的`Item` 实体，看看它是如何工作的:

```java
@Entity
public class Item {

    @Id
    @GeneratedValue
    private Long id;

    @Column(nullable = false)
    private BigDecimal price;
}
```

我们现在可以尝试保存一个`null price`值:

```java
@SpringBootTest
public class ItemIntegrationTest {

    @Autowired
    private ItemRepository itemRepository;

    @Test
    public void shouldNotAllowToPersistNullItemsPrice() {
        itemRepository.save(new Item());
    }
}
```

下面是 Hibernate 的输出片段:

```java
Hibernate: 

    create table item (
       id bigint not null,
        price decimal(19,2) not null,
        primary key (id)
    )

(...)

Hibernate: 
    insert 
    into
        item
        (price, id) 
    values
        (?, ?)
2019-11-14 13:23:03.000  WARN 14580 --- [main] o.h.engine.jdbc.spi.SqlExceptionHelper   : 
SQL Error: 23502, SQLState: 23502
2019-11-14 13:23:03.000 ERROR 14580 --- [main] o.h.engine.jdbc.spi.SqlExceptionHelper   : 
NULL not allowed for column "PRICE"
```

首先，我们可以注意到 **Hibernate 按照我们预期的**生成了带有`not null`约束的价格列。

此外，它能够创建 SQL 插入查询并传递它。因此，**是底层数据库触发了错误。**

### 4.1.确认

几乎所有的资料都强调`@Column(nullable = false)`仅用于模式 DDL 生成。

然而，Hibernate 能够**针对可能的`null`值执行实体的验证，即使相应的字段只标注了`@Column(nullable = false)`** 。

为了激活这个 Hibernate 特性，我们需要显式地将`hibernate.check_nullability`属性设置为`true`:

```java
spring.jpa.show-sql=true
spring.jpa.properties.hibernate.check_nullability=true
```

现在让我们再次执行我们的测试用例，并检查输出:

```java
org.springframework.dao.DataIntegrityViolationException: 
not-null property references a null or transient value : com.baeldung.h2db.springboot.models.Item.price; 
nested exception is org.hibernate.PropertyValueException: 
not-null property references a null or transient value : com.baeldung.h2db.springboot.models.Item.price
```

这一次，我们的测试用例抛出了`org.hibernate.PropertyValueException`。

注意到这一点很重要，在这种情况下， **Hibernate 没有向数据库**发送插入 SQL 查询。

## 5.摘要

在本文中，我们已经描述了`@NotNull`和`@Column(nullable – false)`注释是如何工作的。

尽管它们都阻止我们在数据库中存储`null`值，但它们采用了不同的方法。

**根据经验法则`,`，我们应该更喜欢`@NotNull`注释，而不是`@Column(nullable = false)`注释**。这样，我们可以确保在 Hibernate 向数据库发送任何插入或更新 SQL 查询之前进行验证。

此外，依靠 Bean 验证中定义的标准规则通常比让数据库处理验证逻辑更好。

但是，即使我们让 Hibernate 生成数据库模式，**它也会将`@NotNull`注释翻译成数据库约束。我们必须确保`hibernate.validator.apply_to_ddl`属性被设置为`true.`**

像往常一样，GitHub 上的所有代码示例[都可用。](https://web.archive.org/web/20220626075530/https://github.com/eugenp/tutorials/tree/master/persistence-modules/spring-boot-persistence-h2)