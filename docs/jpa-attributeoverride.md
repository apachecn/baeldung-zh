# 用@AttributeOverride 覆盖列定义

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/jpa-attributeoverride>

## 1.概观

在本教程中，我们将展示如何使用`[@AttributeOverride](https://web.archive.org/web/20220628143724/https://docs.oracle.com/javaee/6/api/javax/persistence/AttributeOverride.html)` 来改变列映射。我们将解释在扩展或嵌入一个实体时如何使用它，我们将讨论单个和集合嵌入。

## 2.`@AttributeOverride`的属性

该注释包含两个强制属性:

*   `name – `被包含实体的字段名称
*   `column`–覆盖原始对象中定义的列定义

## 3.与`@MappedSuperclass`一起使用

让我们定义一个`Vehicle`类`:`

```
@MappedSuperclass
public class Vehicle {
    @Id
    @GeneratedValue
    private Integer id;
    private String identifier;
    private Integer numberOfWheels;

    // standard getters and setters
}
```

`[@MappedSuperclass](https://web.archive.org/web/20220628143724/https://docs.oracle.com/javaee/5/api/javax/persistence/MappedSuperclass.html)`注释表明它是其他实体的基类。

接下来让我们定义类`Car`，它扩展了`Vehicle.` **它演示了如何扩展一个实体并在一个表中存储一辆汽车的信息。请注意，注释出现在类上:**

```
@Entity
@AttributeOverride(name = "identifier", column = @Column(name = "VIN"))
public class Car extends Vehicle {
    private String model;
    private String name;

    // standard getters and setters
}
```

因此，我们有一个包含所有汽车详细信息和车辆详细信息的表。事情是这样的，对于一辆汽车，我们想在列`VIN.` 中存储一个标识符，我们用`@AttributeOverride.` **实现它，注释定义字段`identifier` 存储在`VIN`列中。**

## 4.与嵌入类一起使用

现在让我们用两个可嵌入的类为我们的车辆添加更多的细节。

让我们首先定义基本地址信息:

```
@Embeddable
public class Address {
    private String name;
    private String city;

    // standard getters and setters
}
```

让我们也创建一个包含汽车制造商信息的类:

```
@Embeddable
public class Brand {
    private String name;
    private LocalDate foundationDate;
    @Embedded
    private Address address;

    // standard getters and setters
}
```

`Brand` 类包含一个带有地址细节的嵌入式类。**我们将使用它来演示如何使用多层嵌入的`@AttributeOverride` 。**

让我们用`Brand `细节来扩展我们的`Car `:

```
@Entity
@AttributeOverride(name = "identifier", column = @Column(name = "VIN"))
public class Car extends Vehicle {
    // existing fields

    @Embedded
    @AttributeOverrides({
      @AttributeOverride(name = "name", column = @Column(name = "BRAND_NAME", length = 5)),
      @AttributeOverride(name = "address.name", column = @Column(name = "ADDRESS_NAME"))
    })
    private Brand brand;

    // standard getters and setters
}
```

首先，**`@[AttributeOverrides](https://web.archive.org/web/20220628143724/https://docs.oracle.com/javaee/5/api/javax/persistence/AttributeOverrides.html)`注释允许我们修改多个属性**。我们已经覆盖了来自`Brand` 类的`name `列定义，因为相同的列存在于`Car `类中。因此，品牌名称存储在列`BRAND_NAME`中。

此外，**我们已经定义了列长度，以表明不仅列名可以被覆盖**。注意**`column`属性覆盖了被覆盖类**中的所有值。要保持原始值，必须在`column`属性`.` 中全部设置

除此之外，`Address` 类中的`name`列已经被映射到了`ADDRESS_NAME`。**为了在多层嵌入中覆盖映射，我们使用了点“.”指定覆盖字段**的路径。

## 5.嵌入式收藏

让我们稍微试验一下这个注释，看看它是如何处理集合的。

让我们添加汽车所有者的详细信息:

```
@Embeddable
public class Owner {
    private String name;
    private String surname;

    // standard getters and setters
}
```

我们希望拥有一个拥有者和一个地址，所以让我们添加一个拥有者和他们地址的地图`:`

```
@Entity
@AttributeOverride(name = "identifier", column = @Column(name = "VIN"))
public class Car extends Vehicle {
    // existing fields

    @ElementCollection
    @AttributeOverrides({
      @AttributeOverride(name = "key.name", column = @Column(name = "OWNER_NAME")),
      @AttributeOverride(name = "key.surname", column = @Column(name = "OWNER_SURNAME")),
      @AttributeOverride(name = "value.name", column = @Column(name = "ADDRESS_NAME")),
    })
    Map<Owner, Address> owners;

    // standard getters and setters
}
```

**多亏了注释，我们可以重用`Address`类**。前缀`key`表示对来自`Owner` 类的字段的覆盖。此外，一个`value`前缀指向来自`Address`类的字段。对于列表，不需要添加前缀。

## 6.结论

这篇关于`@AttibuteOverride`注释的短文到此结束。我们已经看到了在扩展或嵌入实体时如何使用这个注释。之后，我们学习了如何在一个系列中使用它。

和往常一样，这个例子的源代码可以在 GitHub 的[上找到。](https://web.archive.org/web/20220628143724/https://github.com/eugenp/tutorials/tree/master/persistence-modules/spring-data-jpa-annotations)