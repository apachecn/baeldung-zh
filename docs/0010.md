# Hibernate 的 addScalar()方法

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/hibernate-addscalar>

## 1.概观

在这个快速教程中，我们将借助一个例子来讨论 Hibernate 中使用的`addScalar()`方法。我们将学习如何使用这种方法以及使用它的好处。

## 2.`addScalar()`解决什么问题？

通常，当使用原生 SQL 查询在 Hibernate 中获取结果时，我们使用`createNativeQuery()` 方法，然后是 `list()` 方法:

```
session.createNativeQuery("SELECT * FROM Student student")
  .list();
```

在这种情况下，Hibernate 使用`ResultSetMetadata` 来查找列细节，并返回`Object`数组`.` 的列表

但是，**过度使用 `ResultSetMetadata`可能会导致性能下降，**这就是`addScalar()`方法有用的地方。

通过**使用`addScalar()`的方法，我们可以防止 Hibernate 使用`ResultSetMetadata`T3。**

## 3.如何使用`addScalar()`？

让我们创建一个新方法，使用 `addScalar()`方法获取学生列表:

```
public List<Object[]> fetchColumnWithScalar() {
    return session.createNativeQuery("SELECT * FROM Student student")
      .addScalar("studentId", StandardBasicTypes.LONG)
      .addScalar("name", StandardBasicTypes.STRING)
      .addScalar("age", StandardBasicTypes.INTEGER)
      .list();
}
```

这里，我们需要指定列名及其数据类型作为`addScalar()`方法的参数。

现在，Hibernate 不会使用`ResultSetMetadata`来获取列细节，因为我们在`addScalar().` 中预先定义了它，因此，与之前的方法相比，我们将获得更好的性能。

## 4.其他优势

让我们再看几个可以使用 `addScalar()`方法的用例。

### 4.1.限制列数

我们还可以**使用 `addScalar()`方法来限制我们的查询**返回的列数。

让我们编写另一个方法`fetchLimitedColumnWithScalar()` 来只获取学生姓名列:

```
public List<String> fetchLimitedColumnWithScalar() {
    return session.createNativeQuery("SELECT * FROM Student student")
      .addScalar("name", StandardBasicTypes.STRING)
      .list();
}
```

这里，我们在查询中使用了星号来获取学生的`List`:

```
SELECT * FROM Student student
```

但是，它不会获取所有的列，只会在`List` 中返回一个列`name`，因为我们在`addScalar()`方法中只指定了一个列。

让我们创建一个 JUnit 方法来验证由`fetchLimitedColumnWithScalar()`方法返回的列:

```
List<String> list = scalarExample.fetchLimitedColumnWithScalar();
for (String colValue : list) {
    assertTrue(colValue.startsWith("John"));
}
```

正如我们所见，这将返回字符串的`List` 而不是`Object`数组。此外，在我们的样本数据中，我们保留了所有以“John”开头的学生姓名，这就是为什么我们在上面的单元测试中针对它断言列值。

这使得我们的代码在返回什么方面更加明确。

### 4.2.返回单个标量值

我们还可以使用`addScalar()`方法直接返回一个标量值，而不是列表。

让我们创建一个返回所有学生平均年龄的方法:

```
public Integer fetchAvgAgeWithScalar() {
    return (Integer) session.createNativeQuery("SELECT AVG(age) as avgAge FROM Student student")
      .addScalar("avgAge")
      .uniqueResult();
}
```

现在，让我们用一个单元测试方法来验证这一点:

```
Integer avgAge = scalarExample.fetchAvgAgeWithScalar();
assertEquals(true, (avgAge >= 5 && avgAge <= 24));
```

正如我们所看到的，`fetchAvgAgeScalar()`方法直接返回了`Integer`值，我们正在断言它。

在我们的样本数据中，我们提供了 5 到 24 岁之间的学生的随机年龄。因此，在断言期间，我们期望平均值在 5 到 24 之间。

类似地，我们可以**使用 SQL 中的任何其他聚合函数直接获得`count`、`max`、`min`、`sum`，或者直接使用`addScalar()`方法**获得任何其他单个标量值。

## 5.重载的`addScalar()` 方法

我们还有一个重载的方法**，它只接受一个列名作为它的单个参数**。

让我们创建一个新方法并使用重载的`addScalar()`方法，该方法获取`“`年龄`“`列而不指定其类型:

```
public List<Object[]> fetchWithOverloadedScalar() {
    return session.createNativeQuery("SELECT * FROM Student student")
      .addScalar("name", StandardBasicTypes.STRING)
      .addScalar("age")
      .list();
}
```

现在，让我们编写另一个 JUnit 方法来验证我们的方法是否返回两列或更多列:

```
List<Object[]> list = scalarExample.fetchColumnWithOverloadedScalar();
for (Object[] colArray : list) {
    assertEquals(2, colArray.length);
}
```

正如我们所看到的，这返回了一个`Object`数组的`List`，数组的大小为 2，代表列表中的姓名和年龄列。

## 6.结论

在本文中，我们已经看到了在 Hibernate 中使用`addScalar()`方法，如何使用它，何时使用它，以及一个例子。

和往常一样，这些例子的代码可以在 GitHub 的[上找到。](https://web.archive.org/web/20220524055631/https://github.com/eugenp/tutorials/tree/master/persistence-modules/hibernate-queries)