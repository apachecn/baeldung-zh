# 用 Hibernate 映射 PostgreSQL 数组

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-hibernate-map-postgresql-array>

## 1.概观

**PostgreSQL 支持将任何类型的数组(内置的或用户自定义的)定义为表的列类型**。在本教程中，我们将探索用 [Hibernate](/web/20221129010145/https://www.baeldung.com/learn-jpa-hibernate) 映射 PostgreSQL 数组的几种方法。

## 2.基本设置

作为连接 PostgreSQL 数据库的先决条件，我们应该将最新的 [`postgresql`](https://web.archive.org/web/20221129010145/https://search.maven.org/search?q=g:org.postgresql%20a:postgresql) Maven 依赖项以及 Hibernate 配置添加到我们的`pom.xml`中。同样，让我们用`String`数组`roles`创建一个名为`User` 的实体类[:](/web/20221129010145/https://www.baeldung.com/jpa-entities)

```java
@Entity
public class User {
    @Id
    private Long id;
    private String name;

    private String[] roles;

    //getters and setters 
} 
```

## 3.自定义休眠类型

[Hibernate 支持自定义类型](/web/20221129010145/https://www.baeldung.com/hibernate-custom-types)将用户定义的类型映射到 SQL 查询中。因此，**我们可以创建自定义类型来映射 PostgreSQL 数组和 Hibernate** 来存储/获取数据。首先，让我们创建实现 Hibernate 的`UserType`类的`CustomStringArrayType`类[，以提供一个自定义类型来映射`String`数组:](/web/20221129010145/https://www.baeldung.com/hibernate-custom-types#2-implementingusertype)

```java
public class CustomStringArrayType implements UserType {
    @Override
    public int[] sqlTypes() {
        return new int[]{Types.ARRAY};
    }

    @Override
    public Class returnedClass() {
        return String[].class;
    }

    @Override
    public Object nullSafeGet(ResultSet rs, String[] names, SharedSessionContractImplementor session, Object owner)
      throws HibernateException, SQLException {
        Array array = rs.getArray(names[0]);
        return array != null ? array.getArray() : null;
    }

    @Override
    public void nullSafeSet(PreparedStatement st, Object value, int index, SharedSessionContractImplementor session)
      throws HibernateException, SQLException {
        if (value != null && st != null) {
            Array array = session.connection().createArrayOf("text", (String[])value);
            st.setArray(index, array);
        } else {
            st.setNull(index, sqlTypes()[0]);
        }
    }
    //implement equals, hashCode, and other methods 
} 
```

这里要注意的是，`returnedClass`方法的**返回类型是`String`数组**。另外，**`nullSafeSet`方法创建一个 PostgreSQL 类型的数组`text`** 。

## 4.使用自定义 Hibernate 类型映射数组

### 4.1.`User`实体

然后，我们将使用`CustomStringArrayType`类将`String`数组`roles` 映射到 PostgreSQL `text`数组:

```java
@Entity
public class User {
    //...

    @Column(columnDefinition = "text[]")
    @Type(type = "com.baeldung.hibernate.arraymapping.CustomStringArrayType")
    private String[] roles;

   //getters and setters 
} 
```

就是这样！我们已经准备好自定义类型实现和数组映射来对 `User` 实体执行 CRUD 操作。

### 4.2.单元测试

为了测试我们的定制类型，让我们首先插入一个`User`对象和`String`数组`roles`:

```java
@Test
public void givenArrayMapping_whenArraysAreInserted_thenPersistInDB() 
  throws HibernateException, IOException {
    transaction = session.beginTransaction();

    User user = new User();
    user.setId(2L);
    user.setName("smith");

    String[] roles = {"admin", "employee"};
    user.setRoles(roles);

    session.persist(user);
    session.flush();
    session.clear();

    transaction.commit();

    User userDBObj = session.find(User.class, 2L);

    assertEquals("smith", userDBObj.getName());
}
```

此外，我们可以获取包含 PostgreSQL `text`数组形式的`roles`的`User`记录:

```java
@Test
public void givenArrayMapping_whenQueried_thenReturnArraysFromDB() 
  throws HibernateException, IOException {
    User user = session.find(User.class, 2L);

    assertEquals("smith", user.getName());
    assertEquals("admin", user.getRoles()[0]);
    assertEquals("employee", user.getRoles()[1]);
}
```

### 4.3.`CustomIntegerArrayType`

类似地，我们可以为 PostgreSQL 支持的各种数组类型创建一个自定义类型。例如，让我们创建`CustomIntegerArrayType`来映射 PostgreSQL `int`数组:

```java
public class CustomIntegerArrayType implements UserType {
    @Override
    public int[] sqlTypes() {
        return new int[]{Types.ARRAY};
    }

    @Override
    public Class returnedClass() {
        return Integer[].class;
    }

    @Override
    public Object nullSafeGet(ResultSet rs, String[] names, SharedSessionContractImplementor session, Object owner)
      throws HibernateException, SQLException {
        Array array = rs.getArray(names[0]);
        return array != null ? array.getArray() : null;
    }

    @Override
    public void nullSafeSet(PreparedStatement st, Object value, int index, SharedSessionContractImplementor session)
      throws HibernateException, SQLException {
        if (value != null && st != null) {
            Array array = session.connection().createArrayOf("int", (Integer[])value);
            st.setArray(index, array);
        } else {
            st.setNull(index, sqlTypes()[0]);
        }
    }

    //implement equals, hashCode, and other methods 
} 
```

类似于我们在`CustomStringArrayType`类中注意到的，**方法的返回类型是`Integer`数组**。另外，**方法的实现创建了一个 PostgreSQL 类型的数组`int`** 。最后，我们可以使用`CustomIntegerArrayType`类将`Integer`数组 `locations` 映射到 PostgreSQL `int`数组:

```java
@Entity
public class User {
    //...

    @Column(columnDefinition = "int[]")
    @Type(type = "com.baeldung.hibernate.arraymapping.CustomIntegerArrayType")
    private Integer[] locations;

    //getters and setters
} 
```

## 5.用`hibernate-types`映射数组

另一方面，我们可以使用由著名的 Hibernate 专家 Vlad Mihalcea 开发的 [`hibernate-types`库](/web/20221129010145/https://www.baeldung.com/hibernate-types-library)，而不是为每个类型实现一个定制类型，如`String`、`Integer`和`Long`。

### 5.1.设置

首先，我们将最新的 [`hibernate-types-52`](https://web.archive.org/web/20221129010145/https://search.maven.org/search?q=g:com.vladmihalcea%20a:hibernate-types-52) Maven 依赖项添加到我们的`pom.xml`:

```java
<dependency>
    <groupId>com.vladmihalcea</groupId>
    <artifactId>hibernate-types-52</artifactId>
    <version>2.10.4</version>
</dependency>
```

### 5.2.`User` 实体

接下来，我们将在`User`实体中添加集成代码，以映射`String`数组 `phoneNumbers`:

```java
@TypeDefs({
    @TypeDef(
        name = "string-array",
        typeClass = StringArrayType.class
    )
})
@Entity
public class User {
    //...
    @Type(type = "string-array")
    @Column(
        name = "phone_numbers",
        columnDefinition = "text[]"
    )
    private String[] phoneNumbers;

    //getters and setters
}
```

这里，类似于自定义类型`CustomStringArrayType`，我们使用了由`hibernate-types`库提供的`StringArrayType`类作为`String`数组的映射器。同样，我们可以在库中找到**其他一些方便的地图绘制者，如`DateArrayType`、`EnumArrayType`和`DoubleArrayType`。**

### 5.3.单元测试

就是这样！我们已经准备好使用`hibernate-types`库进行数组映射。让我们更新已经讨论过的单元测试来验证插入操作:

```java
@Test
public void givenArrayMapping_whenArraysAreInserted_thenPersistInDB() 
  throws HibernateException, IOException {
    transaction = session.beginTransaction();

    User user = new User();
    user.setId(2L);
    user.setName("smith");

    String[] roles = {"admin", "employee"};
    user.setRoles(roles);

    String[] phoneNumbers = {"7000000000", "8000000000"};
    user.setPhoneNumbers(phoneNumbers);

    session.persist(user);
    session.flush();
    session.clear();

    transaction.commit();
}
```

类似地，我们可以验证读取操作:

```java
@Test
public void givenArrayMapping_whenQueried_thenReturnArraysFromDB() 
  throws HibernateException, IOException {
    User user = session.find(User.class, 2L);

    assertEquals("smith", user.getName());
    assertEquals("admin", user.getRoles()[0]);
    assertEquals("employee", user.getRoles()[1]);
    assertEquals("7000000000", user.getPhoneNumbers()[0]);
    assertEquals("8000000000", user.getPhoneNumbers()[1]);
}
```

## 6.结论

在本文中，我们探索了用 Hibernate 映射 PostgreSQL 数组。首先，我们使用 Hibernate 的`UserType`类创建了一个自定义类型来映射`String`数组。然后，我们使用自定义类型将 PostgreSQL `text`数组映射到 Hibernate。最后，我们使用`hibernate-types`库来映射 PostgreSQL 数组。和往常一样，源代码可以在 GitHub 的[上找到。](https://web.archive.org/web/20221129010145/https://github.com/eugenp/tutorials/tree/master/persistence-modules/hibernate-mapping)