# 用 Spring 数据迭代大型结果集的模式

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-data-jpa-iterate-large-result-sets>

## 1.概观

在本教程中，我们将**探索用[Spring Data JPA](/web/20221007132457/https://www.baeldung.com/the-persistence-layer-with-spring-data-jpa)检索大型数据集的各种方法。**

首先，我们将使用分页查询，我们将看到`Slice`和`Page`之间的区别。之后，我们将学习如何流式传输和处理来自数据库的数据，而不是收集数据。

## 2.分页查询

这种情况的一种常见方法是使用[分页查询](/web/20221007132457/https://www.baeldung.com/spring-data-jpa-pagination-sorting)。为此，**我们需要定义一个批量大小并执行多个查询**。因此，我们将能够以较小的批量处理所有实体，并避免在内存中加载大量数据。

### 2.1 使用切片分页

对于本文中的代码示例，我们将使用`Student` 实体作为数据模型:

```
@Entity
public class Student {

    @Id
    @GeneratedValue
    private Long id;

    private String firstName;
    private String lastName;

    // consturctor, getters and setters

}
```

让我们添加一个通过`firstName`查询所有学生的方法。对于 Spring Data JPA，我们只需向`JpaRepository`添加一个方法，该方法接收一个`Pableable`作为参数并返回一个`Slice`:

```
@Repository
public interface StudentRepository extends JpaRepository<Student, Long> {
    Slice<Student> findAllByFirstName(String firstName, Pageable page);
}
```

**我们可以注意到返回类型是`Slice<Students>`。*切片*对象允许我们处理第一批`Student`实体。**`slice`对象公开了一个`hasNext()` 方法，允许我们检查正在处理的批处理是否是结果集中的最后一个。

此外，在方法`nextPageable().` 的帮助下，我们可以从一个切片移动到下一个切片。该方法返回请求下一个切片所需的`Pageable`对象。因此，我们可以在一个`while`循环中结合使用这两种方法，一片一片地检索所有数据:

```
void processStudentsByFirstName(String firstName) {
    Slice<Student> slice = repository.findAllByFirstName(firstName, PageRequest.of(0, BATCH_SIZE));
    List<Student> studentsInBatch = slice.getContent();
    studentsInBatch.forEach(emailService::sendEmailToStudent);

    while(slice.hasNext()) {
        slice = repository.findAllByFirstName(firstName, slice.nextPageable());
        slice.get().forEach(emailService::sendEmailToStudent);
    }
}
```

让我们使用小批量运行一个简短的测试，并遵循 SQL 语句。我们希望执行多个查询:

```
[main] DEBUG org.hibernate.SQL - select student0_.id as id1_0_, student0_.first_name as first_na2_0_, student0_.last_name as last_nam3_0_ from student student0_ where student0_.first_name=? limit ?
[main] DEBUG org.hibernate.SQL - select student0_.id as id1_0_, student0_.first_name as first_na2_0_, student0_.last_name as last_nam3_0_ from student student0_ where student0_.first_name=? limit ? offset ?
[main] DEBUG org.hibernate.SQL - select student0_.id as id1_0_, student0_.first_name as first_na2_0_, student0_.last_name as last_nam3_0_ from student student0_ where student0_.first_name=? limit ? offset ?
```

### 2.2.使用页面分页

作为`Slice` < >的替代，我们也可以使用`Page<>` 作为查询的返回类型:

```
@Repository
public interface StudentRepository extends JpaRepository<Student, Long> {
    Slice<Student> findAllByFirstName(String firstName, Pageable page);
    Page<Student> findAllByLastName(String lastName, Pageable page);
}
```

**`Page`接口扩展了`Slice,` ，向其添加了另外两个方法:`getTotalPages()`和`getTotalElements()`。**

当通过网络请求分页数据时，通常使用。这样，调用者将确切地知道还剩多少行以及还需要多少额外的请求。

另一方面，使用`Page` s 会导致对满足条件的行进行计数的额外查询:

```
[main] DEBUG org.hibernate.SQL - select student0_.id as id1_0_, student0_.first_name as first_na2_0_, student0_.last_name as last_nam3_0_ from student student0_ where student0_.last_name=? limit ?
[main] DEBUG org.hibernate.SQL - select count(student0_.id) as col_0_0_ from student student0_ where student0_.last_name=?
[main] DEBUG org.hibernate.SQL - select student0_.id as id1_0_, student0_.first_name as first_na2_0_, student0_.last_name as last_nam3_0_ from student student0_ where student0_.last_name=? limit ? offset ?
[main] DEBUG org.hibernate.SQL - select count(student0_.id) as col_0_0_ from student student0_ where student0_.last_name=?
[main] DEBUG org.hibernate.SQL - select student0_.id as id1_0_, student0_.first_name as first_na2_0_, student0_.last_name as last_nam3_0_ from student student0_ where student0_.last_name=? limit ? offset ? 
```

**因此，如果我们需要知道实体的总数，我们应该只使用页面<T3 作为返回类型。**

## 3.从数据库流式传输

Spring Data JPA 还允许我们从结果集中流式传输数据:

```
Stream<Student> findAllByFirstName(String firstName);
```

因此，我们将一个接一个地处理实体，而不是同时将它们加载到内存中。然而，我们需要用一个 [try-with-resource](/web/20221007132457/https://www.baeldung.com/java-try-with-resources) 块手动关闭 Spring Data JPA 创建的流。此外，我们必须将查询包装在一个只读事务中。

最后，即使我们一行一行地处理，我们也必须确保持久化上下文没有保存对所有实体的引用。我们可以通过在使用流之前手动分离实体来实现这一点:

```
private final EntityManager entityManager;

@Transactional(readOnly = true)
public void processStudentsByFirstNameUsingStreams(String firstName) {
    try (Stream<Student> students = repository.findAllByFirstName(firstName)) {
        students.peek(entityManager::detach)
            .forEach(emailService::sendEmailToStudent);
    }
}
```

## 4.结论

在本文中，我们探索了处理大型数据集的各种方法。最初，我们通过多个分页的查询来实现这一点。我们了解到，当调用者需要知道元素的总数时，我们应该使用`Page<>`作为返回类型，否则使用`Slice<>`。之后，我们学习了如何从数据库中流式传输行并分别处理它们。

和往常一样，代码样本可以在 GitHub 上找到[。](https://web.archive.org/web/20221007132457/https://github.com/eugenp/tutorials/tree/master/persistence-modules/spring-boot-persistence-3)