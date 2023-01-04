# Hibernate 中的自定义类型和@Type 注释

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/hibernate-custom-types>

## 1.概观

Hibernate 通过将 Java 中的面向对象模型与数据库中的关系模型进行映射，简化了 SQL 和 JDBC 之间的数据处理。虽然基本 Java 类的映射是 Hibernate 内置的，但是定制类型的映射通常很复杂。

在本教程中，我们将了解 Hibernate 如何允许我们将基本类型映射扩展到自定义 Java 类。除此之外，我们还将看到一些自定义类型的常见示例，并使用 Hibernate 的类型映射机制来实现它们。

## 2.Hibernate 映射类型

Hibernate 使用映射类型将 Java 对象转换成存储数据的 SQL 查询。类似地，在检索数据时，它使用映射类型将 SQL ResultSet 转换成 Java 对象。

通常，Hibernate 将类型分为实体类型和值类型**。**具体来说，实体类型用于映射特定领域的 Java 实体，因此独立于应用程序中的其他类型而存在。相反，值类型用于映射数据对象，并且几乎总是由实体拥有。

在本教程中，我们将重点关注值类型的映射，值类型进一步分为:

*   基本类型–基本 Java 类型的映射
*   可嵌入——复合 java 类型/POJO 的映射
*   集合–基本和复合 java 类型集合的映射

## 3.Maven 依赖性

为了创建我们的自定义 Hibernate 类型，我们需要 [hibernate-core](https://web.archive.org/web/20221130182151/https://search.maven.org/search?q=g:org.hibernate%20a:hibernate-core) 依赖关系:

```
<dependency>
    <groupId>org.hibernate</groupId>
    <artifactId>hibernate-core</artifactId>
    <version>5.6.7.Final</version>
</dependency>
```

## 4.Hibernate 中的自定义类型

我们可以对大多数用户域使用 Hibernate 基本映射类型。然而，在许多用例中，我们需要实现自定义类型。

Hibernate 使得实现定制类型变得相对容易。在 Hibernate 中实现自定义类型有三种方法。我们来详细讨论一下每一个。

### 4.1.实施`BasicType`

我们可以通过实现 Hibernate 的`BasicType `或者它的一个特定实现`AbstractSingleColumnStandardBasicType.`来创建一个定制的基本类型

在我们实现第一个自定义类型之前，让我们看一个实现基本类型的常见用例。假设我们必须使用一个遗留数据库，它将日期存储为 VARCHAR。通常， **Hibernate 会将其映射到`String` Java 类型。因此，对于应用程序开发人员来说，日期验证变得更加困难。**

所以让我们实现我们的`LocalDateString `类型，它将`LocalDate ` Java 类型存储为 VARCHAR:

```
public class LocalDateStringType 
  extends AbstractSingleColumnStandardBasicType<LocalDate> {

    public static final LocalDateStringType INSTANCE = new LocalDateStringType();

    public LocalDateStringType() {
        super(VarcharTypeDescriptor.INSTANCE, LocalDateStringJavaDescriptor.INSTANCE);
    }

    @Override
    public String getName() {
        return "LocalDateString";
    }
}
```

这段代码中最重要的是构造函数参数。首先是`SqlTypeDescriptor`的实例，它是 Hibernate 的 SQL 类型表示，在我们的例子中是 VARCHAR。第二个参数是代表 Java 类型的`JavaTypeDescriptor`的实例。

现在，我们可以用 VARCHAR: 来实现一个用于存储和检索`LocalDate` 的`LocalDateStringJavaDescriptor `

```
public class LocalDateStringJavaDescriptor extends AbstractTypeDescriptor<LocalDate> {

    public static final LocalDateStringJavaDescriptor INSTANCE = 
      new LocalDateStringJavaDescriptor();

    public LocalDateStringJavaDescriptor() {
        super(LocalDate.class, ImmutableMutabilityPlan.INSTANCE);
    }

    // other methods
}
```

接下来，我们需要覆盖将 Java 类型转换成 SQL 的`wrap `和`unwrap `方法。让我们从`unwrap:`开始

```
@Override
public <X> X unwrap(LocalDate value, Class<X> type, WrapperOptions options) {

    if (value == null)
        return null;

    if (String.class.isAssignableFrom(type))
        return (X) LocalDateType.FORMATTER.format(value);

    throw unknownUnwrap(type);
}
```

接下来，`wrap `方法:

```
@Override
public <X> LocalDate wrap(X value, WrapperOptions options) {
    if (value == null)
        return null;

    if(String.class.isInstance(value))
        return LocalDate.from(LocalDateType.FORMATTER.parse((CharSequence) value));

    throw unknownWrap(value.getClass());
}
```

在`PreparedStatement `绑定期间调用`unwrap() `，将`LocalDate `转换为字符串类型，该类型映射到 VARCHAR。同样，在`ResultSet `检索过程中调用`wrap() `将`String `转换成 Java `LocalDate`。

最后，我们可以在实体类中使用自定义类型:

```
@Entity
@Table(name = "OfficeEmployee")
public class OfficeEmployee {

    @Column
    @Type(type = "com.baeldung.hibernate.customtypes.LocalDateStringType")
    private LocalDate dateOfJoining;

    // other fields and methods
}
```

稍后，我们将看到如何在 Hibernate 中注册这种类型。因此，**使用注册码而不是完全限定的类名来引用这个类型。**

### 4.2.实施`UserType`

由于 Hibernate 中基本类型的多样性，我们很少需要实现自定义的基本类型。相比之下，更典型的用例是将复杂的 Java 域对象映射到数据库。这种域对象通常存储在多个数据库列中。

所以让我们通过实现`UserType:`来实现一个复杂的`PhoneNumber`对象

```
public class PhoneNumberType implements UserType {
    @Override
    public int[] sqlTypes() {
        return new int[]{Types.INTEGER, Types.INTEGER, Types.INTEGER};
    }

    @Override
    public Class returnedClass() {
        return PhoneNumber.class;
    }

    // other methods
} 
```

这里，被覆盖的`sqlTypes `方法返回字段的 SQL 类型，其顺序与它们在我们的`PhoneNumber `类中声明的顺序相同。类似地，`returnedClass `方法返回我们的`PhoneNumber ` Java 类型。

剩下唯一要做的就是实现 Java 类型和 SQL 类型之间的转换方法，就像我们对`BasicType`所做的那样。

一、`nullSafeGet `法:

```
@Override
public Object nullSafeGet(ResultSet rs, String[] names, 
  SharedSessionContractImplementor session, Object owner) 
  throws HibernateException, SQLException {
    int countryCode = rs.getInt(names[0]);

    if (rs.wasNull())
        return null;

    int cityCode = rs.getInt(names[1]);
    int number = rs.getInt(names[2]);
    PhoneNumber employeeNumber = new PhoneNumber(countryCode, cityCode, number);

    return employeeNumber;
}
```

接下来，`nullSafeSet `方法:

```
@Override
public void nullSafeSet(PreparedStatement st, Object value, 
  int index, SharedSessionContractImplementor session) 
  throws HibernateException, SQLException {

    if (Objects.isNull(value)) {
        st.setNull(index, Types.INTEGER);
        st.setNull(index + 1, Types.INTEGER);
        st.setNull(index + 2, Types.INTEGER);
    } else {
        PhoneNumber employeeNumber = (PhoneNumber) value;
        st.setInt(index,employeeNumber.getCountryCode());
        st.setInt(index+1,employeeNumber.getCityCode());
        st.setInt(index+2,employeeNumber.getNumber());
    }
}
```

最后，我们可以在我们的`OfficeEmployee `实体类中声明我们的自定义`PhoneNumberType `:

```
@Entity
@Table(name = "OfficeEmployee")
public class OfficeEmployee {

    @Columns(columns = { @Column(name = "country_code"), 
      @Column(name = "city_code"), @Column(name = "number") })
    @Type(type = "com.baeldung.hibernate.customtypes.PhoneNumberType")
    private PhoneNumber employeeNumber;

    // other fields and methods
}
```

### 4.3.实施`CompositeUserType`

对于简单的类型，实现`UserType `效果很好。然而，映射复杂的 Java 类型(带有集合和级联复合类型)需要更多的技巧。 **Hibernate 允许我们通过实现`CompositeUserType `接口来映射这些类型。**

因此，让我们通过为我们之前使用的`OfficeEmployee `实体实现一个`AddressType `来看看这一点:

```
public class AddressType implements CompositeUserType {

    @Override
    public String[] getPropertyNames() {
        return new String[] { "addressLine1", "addressLine2", 
          "city", "country", "zipcode" };
    }

    @Override
    public Type[] getPropertyTypes() {
        return new Type[] { StringType.INSTANCE, 
          StringType.INSTANCE, 
          StringType.INSTANCE, 
          StringType.INSTANCE, 
          IntegerType.INSTANCE };
    }

    // other methods
}
```

与映射类型属性索引的`UserTypes`相反，`CompositeType `映射我们的`Address `类的属性名。更重要的是， `getPropertyType`方法返回每个属性的映射类型。

此外，我们还需要实现将`PreparedStatement `和`ResultSet `索引映射到类型属性的`getPropertyValue `和`setPropertyValue `方法。作为一个例子，考虑我们的`AddressType:`的`getPropertyValue `

```
@Override
public Object getPropertyValue(Object component, int property) throws HibernateException {

    Address empAdd = (Address) component;

    switch (property) {
    case 0:
        return empAdd.getAddressLine1();
    case 1:
        return empAdd.getAddressLine2();
    case 2:
        return empAdd.getCity();
    case 3:
        return empAdd.getCountry();
    case 4:
        return Integer.valueOf(empAdd.getZipCode());
    }

    throw new IllegalArgumentException(property + " is an invalid property index for class type "
      + component.getClass().getName());
}
```

最后，我们需要实现用于 Java 和 SQL 类型之间转换的`nullSafeGet `和`nullSafeSet `方法。这类似于我们之前在`PhoneNumberType.`中所做的

请注意， **`CompositeType`通常被实现为`Embeddable `类型的替代映射机制。**

### 4.4.类型参数化

除了创建自定义类型， **Hibernate 还允许我们根据参数改变类型的行为。**

例如，假设我们需要存储我们的`OfficeEmployee. `的`Salary `，更重要的是，应用程序必须将工资金额转换成当地货币金额。

因此，让我们实现我们的参数化的`SalaryType `，它接受`currency `作为参数:

```
public class SalaryType implements CompositeUserType, DynamicParameterizedType {

    private String localCurrency;

    @Override
    public void setParameterValues(Properties parameters) {
        this.localCurrency = parameters.getProperty("currency");
    }

    // other method implementations from CompositeUserType
}
```

请注意，我们已经跳过了示例中的`CompositeUserType` 方法，专注于参数化。这里，我们简单地实现了 Hibernate 的`DynamicParameterizedType`，并覆盖了`setParameterValues() `方法。现在，`SalaryType `接受一个`currency `参数，并在存储之前转换任何数量。

我们将在声明`Salary:`时将`currency`作为参数传递

```
@Entity
@Table(name = "OfficeEmployee")
public class OfficeEmployee {

    @Type(type = "com.baeldung.hibernate.customtypes.SalaryType", 
      parameters = { @Parameter(name = "currency", value = "USD") })
    @Columns(columns = { @Column(name = "amount"), @Column(name = "currency") })
    private Salary salary;

    // other fields and methods
}
```

## 5.基本类型注册表

Hibernate 维护所有内置基本类型在`BasicTypeRegistry`中的映射。因此，消除了对这种类型的映射信息进行注释的需要。

此外，Hibernate 允许我们在`BasicTypeRegistry`中注册定制类型，就像基本类型一样。通常，应用程序会在引导`SessionFactory. `时注册自定义类型，让我们通过注册我们之前实现的`LocalDateString `类型来理解这一点:

```
private static SessionFactory makeSessionFactory() {
    ServiceRegistry serviceRegistry = StandardServiceRegistryBuilder()
      .applySettings(getProperties()).build();

    MetadataSources metadataSources = new MetadataSources(serviceRegistry);
    Metadata metadata = metadataSources
      .addAnnotatedClass(OfficeEmployee.class)
      .getMetadataBuilder()
      .applyBasicType(LocalDateStringType.INSTANCE)
      .build();

    return metadata.buildSessionFactory()
}

private static Properties getProperties() {
    // return hibernate properties
}
```

因此，**消除了在类型映射中使用完全限定类名的限制:**

```
@Entity
@Table(name = "OfficeEmployee")
public class OfficeEmployee {

    @Column
    @Type(type = "LocalDateString")
    private LocalDate dateOfJoining;

    // other methods
}
```

这里， `LocalDateString`是`LocalDateStringType `映射到的键。

或者，我们可以通过定义`TypeDefs:`跳过类型注册

```
@TypeDef(name = "PhoneNumber", typeClass = PhoneNumberType.class, 
  defaultForType = PhoneNumber.class)
@Entity
@Table(name = "OfficeEmployee")
public class OfficeEmployee {

    @Columns(columns = {@Column(name = "country_code"),
    @Column(name = "city_code"),
    @Column(name = "number")})
    private PhoneNumber employeeNumber;

    // other methods
}
```

## 6.结论

在本教程中，我们讨论了在 Hibernate 中定义自定义类型的多种方法。此外，**我们基于一些新的定制类型可以派上用场的常见用例，为我们的实体类实现了一些定制类型。**

像往常一样，代码样本可以在 GitHub 上的[处获得。](https://web.archive.org/web/20221130182151/https://github.com/eugenp/tutorials/tree/master/persistence-modules/hibernate-annotations)