# 在 JPA 中插入语句

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/jpa-insert>

## 1.概观

在这个快速教程中，我们将学习如何**在 JPA 对象**上执行 INSERT 语句。

关于 Hibernate 的更多信息，请查看我们全面的关于 JPA with Spring 的指南和关于 JPA 的 Spring 数据介绍来深入了解这个主题。

## 2.在 JPA 中持久化对象

在 JPA 中，由 [`EntityManager`](/web/20221207051614/https://www.baeldung.com/hibernate-entitymanager) 自动处理每一个从瞬态进入托管状态的实体。

`EntityManager`检查给定的实体是否已经存在，然后决定是否应该插入或更新它。由于这种自动管理，**t**他只允许 JPA 的语句有 SELECT、UPDATE 和 DELETE。

在下面的例子中，我们将看看管理和绕过这个限制的不同方法。

## 3.定义通用模型

现在，让我们从定义一个我们将在本教程中使用的简单实体开始:

```
@Entity
public class Person {

    @Id
    private Long id;
    private String firstName;
    private String lastName;

    // standard getters and setters, default and all-args constructors
}
```

此外，让我们定义一个将用于我们的实现的存储库类:

```
@Repository
public class PersonInsertRepository {

    @PersistenceContext
    private EntityManager entityManager;

}
```

此外，我们将通过 Spring 应用 [`@Transactional`](/web/20221207051614/https://www.baeldung.com/transaction-configuration-with-jpa-and-spring) 注释来自动处理事务。这样，我们就不必担心用我们的`EntityManager,`提交我们的更改来创建事务，或者在出现异常的情况下手动执行回滚。

## 4.`createNativeQuery`

对于手动创建的查询，我们可以使用`EntityManager#createNativeQuery`方法。它允许我们创建任何类型的 SQL 查询，而不仅仅是 JPA 支持的查询。让我们向我们的存储库类添加一个新方法:

```
@Transactional
public void insertWithQuery(Person person) {
    entityManager.createNativeQuery("INSERT INTO person (id, first_name, last_name) VALUES (?,?,?)")
      .setParameter(1, person.getId())
      .setParameter(2, person.getFirstName())
      .setParameter(3, person.getLastName())
      .executeUpdate();
}
```

使用这种方法，我们需要定义一个包含列名的文字查询，并设置它们相应的值。

我们现在可以测试我们的存储库了:

```
@Test
public void givenPersonEntity_whenInsertedTwiceWithNativeQuery_thenPersistenceExceptionExceptionIsThrown() {
    Person person = new Person(1L, "firstname", "lastname");

    assertThatExceptionOfType(PersistenceException.class).isThrownBy(() -> {
        personInsertRepository.insertWithQuery(PERSON);
        personInsertRepository.insertWithQuery(PERSON);
    });
}
```

在我们的测试中，每个操作都试图向数据库中插入一个新条目。由于我们试图插入两个具有相同`id`的实体，第二个插入操作因抛出一个`PersistenceException`而失败。

如果我们使用 Spring Data 的`[@Query](/web/20221207051614/https://www.baeldung.com/spring-data-jpa-query).`，这里的原理是相同的

## 5.`persist`

在前面的例子中，我们创建了插入查询，但是我们必须为每个实体创建文字查询。这种方法效率不高，并且会产生大量样板代码。

相反，我们可以利用来自`EntityManager`的`persist`方法。

和前面的例子一样，让我们用一个定制的方法来扩展我们的存储库类:

```
@Transactional
public void insertWithEntityManager(Person person) {
    this.entityManager.persist(person);
}
```

现在，我们可以再次测试我们的方法

```
@Test
public void givenPersonEntity_whenInsertedTwiceWithEntityManager_thenEntityExistsExceptionIsThrown() {
    assertThatExceptionOfType(EntityExistsException.class).isThrownBy(() -> {
        personInsertRepository.insertWithEntityManager(new Person(1L, "firstname", "lastname"));
        personInsertRepository.insertWithEntityManager(new Person(1L, "firstname", "lastname"));
    });
}
```

与使用本地查询相比，**我们不必指定列名和相应的值**。相反，`EntityManager`为我们处理这些。

在上面的测试中，我们还期望抛出的是`EntityExistsException`，而不是它的超类`PersistenceException`，后者更加专门化，由`persist`抛出。

另一方面，在这个例子中，我们必须确保**每次都用新的`Person.`** 实例调用我们的插入方法，否则，它将已经被`EntityManager,`管理，导致更新操作。

## 6。结论

在本文中，我们展示了在 JPA 对象上执行插入操作的方法。我们看了使用原生查询的例子，以及使用`EntityManager#persist`创建定制插入语句的例子。

和往常一样，本文中使用的完整代码可以从 GitHub 上的[处获得。](https://web.archive.org/web/20221207051614/https://github.com/eugenp/tutorials/tree/master/persistence-modules/spring-data-jpa-crud)