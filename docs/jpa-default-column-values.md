# JPA 中的默认列值

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/jpa-default-column-values>

## 1.概观

在本教程中，我们将研究 JPA 中的默认列值。

我们将了解如何在实体中以及直接在 SQL 表定义中将它们设置为默认属性。

## 2.创建实体时

设置默认列值的第一种方法是**将其直接设置为实体属性值**:

```java
@Entity
public class User {
    @Id
    private Long id;
    private String firstName = "John Snow";
    private Integer age = 25;
    private Boolean locked = false;
}
```

现在，每次我们使用`new`操作符创建一个实体时，它都会设置我们提供的默认值:

```java
@Test
void saveUser_shouldSaveWithDefaultFieldValues() {
    User user = new User();
    user = userRepository.save(user);

    assertEquals(user.getName(), "John Snow");
    assertEquals(user.getAge(), 25);
    assertFalse(user.getLocked());
}
```

这种解决方案有一个缺点。

当我们查看 SQL 表定义时，我们不会在其中看到任何默认值:

```java
create table user
(
    id     bigint not null constraint user_pkey primary key,
    name   varchar(255),
    age    integer,
    locked boolean
);
```

因此，**如果我们用`null`覆盖它们，实体将被保存，没有任何错误**:

```java
@Test
void saveUser_shouldSaveWithNullName() {
    User user = new User();
    user.setName(null);
    user.setAge(null);
    user.setLocked(null);
    user = userRepository.save(user);

    assertNull(user.getName());
    assertNull(user.getAge());
    assertNull(user.getLocked());
}
```

## 3.在模式定义中

要在 SQL 表定义中直接创建**默认值，我们可以使用`@Column`注释并设置其`columnDefinition`参数:**

```java
@Entity
public class User {
    @Id
    Long id;

    @Column(columnDefinition = "varchar(255) default 'John Snow'")
    private String name;

    @Column(columnDefinition = "integer default 25")
    private Integer age;

    @Column(columnDefinition = "boolean default false")
    private Boolean locked;
}
```

使用这种方法，默认值将出现在 SQL 表定义中:

```java
create table user
(
    id     bigint not null constraint user_pkey primary key,
    name   varchar(255) default 'John Snow',
    age    integer      default 35,
    locked boolean      default false
);
```

实体将使用默认值正确保存:

```java
@Test
void saveUser_shouldSaveWithDefaultSqlValues() {
    User user = new User();
    user = userRepository.save(user);

    assertEquals(user.getName(), "John Snow");
    assertEquals(user.getAge(), 25);
    assertFalse(user.getLocked());
}
```

记住**通过使用这个解决方案，我们将不能在第一次保存实体时将一个给定的列设置为`null`** 。如果我们不提供任何值，缺省值将被自动设置。

## 4.结论

在这篇短文中，我们学习了如何在 JPA 中设置默认的列值。

和往常一样，完整的源代码可以在 GitHub 上找到[。](https://web.archive.org/web/20221128043948/https://github.com/eugenp/tutorials/tree/master/persistence-modules/java-jpa-2)