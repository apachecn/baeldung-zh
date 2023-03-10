# Spring Data Java 8 支持

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-data-java-8>

## 1。概述

Spring Data 现在支持核心的 Java 8 特性——比如`Optional`、`Stream` API 和`CompletableFuture`。

在这篇简短的文章中，我们将通过一些例子来说明如何在框架中使用它们。

## 2。`Optional`

让我们从 CRUD 存储库方法开始——它现在将结果包装在一个`Optional`中:

```java
public interface CrudRepository<T, ID> extends Repository<T, ID> {

    Optional<T> findById(ID id);

}
```

当返回一个`Optional`实例时，这是一个有用的提示，表明该值可能不存在。更多可选信息可在[这里](/web/20220626202127/https://www.baeldung.com/java-optional)找到。

我们现在要做的就是将返回类型指定为一个`Optional`:

```java
public interface UserRepository extends JpaRepository<User, Integer> {

    Optional<User> findOneByName(String name);

} 
```

## 3。`Stream` API

Spring Data 还支持 Java 8 最重要的特性之一——`Stream`API。

过去，每当我们需要返回多个结果时，我们需要返回一个集合:

```java
public interface UserRepository extends JpaRepository<User, Integer> {
    // ...
    List<User> findAll();
    // ...
}
```

这种实现的一个问题是内存消耗。

我们必须急切地加载并保存所有检索到的对象。

我们可以通过利用分页来改进:

```java
public interface UserRepository extends JpaRepository<User, Integer> {
    // ...
    Page<User> findAll(Pageable pageable);
    // ...
} 
```

在某些情况下，这就足够了，但在其他情况下，分页真的不是办法，因为检索数据需要大量的请求。

感谢 Java 8 `Stream` API 和 JPA 提供者——我们现在可以**定义我们的存储库方法只返回一个`Stream`对象**:

```java
public interface UserRepository extends JpaRepository<User, Integer> {
    // ...
    Stream<User> findAllByName(String name);
    // ...
}
```

Spring Data 使用特定于提供者的实现来传输结果(Hibernate 使用`ScrollableResultSet`，EclipseLink 使用`ScrollableCursor`)。它减少了内存消耗和对数据库的查询调用。因此，它也比前面提到的两种解决方案快得多。

**用`Stream`处理数据要求我们在完成**时关闭`Stream`。

这可以通过调用`Stream`上的`close()`方法或使用`try-with-resources`来完成:

```java
try (Stream<User> foundUsersStream 
  = userRepository.findAllByName(USER_NAME_ADAM)) {

assertThat(foundUsersStream.count(), equalTo(3l)); 
```

我们还必须记住在事务中调用一个存储库方法。否则，我们会得到一个异常:

> `org.springframework.dao.InvalidDataAccessApiUsageException`:您正在尝试执行一个流查询方法，而没有一个保持连接打开的周围事务，以便能够实际使用`Stream`。确保使用流的代码使用了`@Transactional`或任何其他声明(只读)事务的方式。

## 4。`CompletableFuture`

**在 Java 8 的`CompletableFuture`** 和 Spring 异步方法执行机制的支持下，Spring 数据仓库可以异步运行:

```java
@Async
CompletableFuture<User> findOneByStatus(Integer status); 
```

调用此方法的客户端将立即返回一个 future，但方法将在不同的线程中继续执行。

更多关于`CompletableFuture`处理的信息可以在[这里](/web/20220626202127/https://www.baeldung.com/java-completablefuture)找到。

## 5。结论

在本教程中，我们展示了 Java 8 特性如何与 Spring 数据协同工作。

Github 上的[提供了示例的完整实现。](https://web.archive.org/web/20220626202127/https://github.com/eugenp/tutorials/tree/master/persistence-modules/spring-data-jpa-enterprise)