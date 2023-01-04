# Hibernate 聚合函数

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/hibernate-aggregate-functions>

## 1.概观

Hibernate 聚合函数**使用满足给定查询条件的所有对象的属性值来计算最终结果。**

Hibernate 查询语言(HQL)支持各种聚合函数——`SELECT`语句中的`min(), max(), sum(), avg(), and count()`。就像任何其他 SQL 关键字一样，这些函数的使用是不区分大小写的。

在这个快速教程中，我们将探索如何使用它们。请注意，在下面的例子中，我们使用原语或包装类型来存储聚合函数的结果。HQL 两者都支持，所以问题是选择使用哪一个。

## 2.初始设置

让我们从定义一个`Student`实体开始:

```java
@Entity
public class Student {

    @Id
    @GeneratedValue(strategy = GenerationType.SEQUENCE)
    private long studentId;

    private String name;

    private int age;

    // constructor, getters and setters
}
```

用一些学生填充我们的数据库:

```java
public class AggregateFunctionsIntegrationTest {

    private static Session session;
    private static Transaction transaction;

    @BeforeClass
    public static final void setup() throws HibernateException, IOException {
        session = HibernateUtil.getSessionFactory()
          .openSession();
        transaction = session.beginTransaction();

        session.save(new Student("Jonas", 22, 12f));
        session.save(new Student("Sally", 20, 34f));
        session.save(new Student("Simon", 25, 45f));
        session.save(new Student("Raven", 21, 43f));
        session.save(new Student("Sam", 23, 33f));

    }

}
```

注意，我们的`studentId`字段已经使用`SEQUENCE`生成策略进行了填充。

我们可以在关于 [Hibernate 标识符生成策略](/web/20220526043258/https://www.baeldung.com/hibernate-identifiers)的教程中了解更多。

## 3.`min()`

现在，假设我们想找出存储在我们的`Student`表中的所有学生的最小年龄。我们可以通过使用`min()`函数轻松实现:

```java
@Test
public void whenMinAge_ThenReturnValue() {
    int minAge = (int) session.createQuery("SELECT min(age) from Student")
      .getSingleResult();
    assertThat(minAge).isEqualTo(20);
}
```

**`getSingleResult()`方法返回一个`Object` 类型。**因此，我们将输出向下转换为一个`int`。

## 4.`max()`

类似于`min()`函数，我们有一个`max()`函数:

```java
@Test
public void whenMaxAge_ThenReturnValue() {
    int maxAge = (int) session.createQuery("SELECT max(age) from Student")
      .getSingleResult();
    assertThat(maxAge).isEqualTo(25);
}
```

同样，结果被向下转换为一个`int`类型。

**`min()`和`max()`函数的返回类型取决于上下文**中的字段。对我们来说，它返回一个整数，因为`Student's age`是一个`int`类型属性。

## 5.`sum()`

我们可以使用`sum()`函数来计算所有年龄的总和:

```java
@Test
public void whenSumOfAllAges_ThenReturnValue() {
    Long sumOfAllAges = (Long) session.createQuery("SELECT sum(age) from Student")
      .getSingleResult();
    assertThat(sumOfAllAges).isEqualTo(111);
}
```

根据字段的数据类型，**`sum()`函数返回一个`Long`或一个`Double`。**

## 6.`avg()`

类似地，我们可以使用`avg()`函数找到平均年龄:

```java
@Test
public void whenAverageAge_ThenReturnValue() {
    Double avgAge = (Double) session.createQuery("SELECT avg(age) from Student")
      .getSingleResult();
    assertThat(avgAge).isEqualTo(22.2);
}
```

**`avg()`函数总是返回一个`Double`值。**

## 7.`count()`

与原生 SQL 一样，HQL 也提供了一个`count()`函数。让我们在`Student`表中找到记录的数量:

```java
@Test
public void whenCountAll_ThenReturnValue() {
    Long totalStudents = (Long) session.createQuery("SELECT count(*) from Student")
      .getSingleResult();
    assertThat(totalStudents).isEqualTo(5);
}
```

**`count()`函数返回一个`Long`类型。**

我们可以使用 `count()`函数的任何可用变体—`count(*), count(…),` `count(distinct …),` 或 `count(all …)`。**它们中的每一个都在语义上等同于其本地 SQL 对应物。**

## 8.结论

在本教程中，我们简要介绍了 Hibernate 中可用的聚合函数类型。Hibernate 聚合函数类似于普通 SQL 中可用的函数。

和往常一样，完整的源代码可以在 GitHub 的[上找到。](https://web.archive.org/web/20220526043258/https://github.com/eugenp/tutorials/tree/master/persistence-modules/hibernate-enterprise)