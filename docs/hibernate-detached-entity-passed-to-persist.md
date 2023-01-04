# Hibernate 的“分离的实体被传递给 Persist”错误

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/hibernate-detached-entity-passed-to-persist>

## 1.概观

在本文中，我们将了解 Hibernate 的`PersistentObjectException`，它在试图保存一个分离的实体时发生。

我们将从理解`detached` 状态的含义以及 Hibernate 的`persist`和`merge`方法之间的区别开始。之后，我们将在各种用例中重现错误，并看看如何修复它。

## 2.分离的实体

让我们先简要回顾一下什么是`detached`状态，以及它与[实体生命周期](/web/20220810173509/https://www.baeldung.com/hibernate-entity-lifecycle)的关系。

一个`detached`实体是一个不再被`persistence context`跟踪的 Java 对象。**如果我们关闭或清除会话，实体可以达到这种状态。类似地，我们可以通过手动将实体从持久性上下文中移除来分离它。**

对于本文中的代码示例，我们将使用`Post`和`Comment`实体。为了分离一个特定的`Post`实体，我们可以使用`session.evict(post)`。此外，我们可以通过使用`session.clear()`清除会话，将所有实体从上下文中分离出来。

例如，一些测试将需要一个分离的`Post`。那么，让我们看看如何实现这一点:

```
@Before
public void beforeEach() {
    session = HibernateUtil.getSessionFactory().openSession();
    session.beginTransaction();

    this.detachedPost = new Post("Hibernate Tutorial");
    session.persist(detachedPost);
    session.evict(detachedPost);
}
```

首先，我们持久化了`Post`实体，然后用`session.evict(post)`将其分离。

## 3.尝试保持分离的实体

如果我们试图持久化一个分离的实体，Hibernate 将抛出一个`PersistenceException`错误消息“分离的实体被传递到持久化”。

让我们尝试持久化一个分离的`Post`实体，并期待这个异常:

```
@Test
public void givenDetachedPost_whenTryingToPersist_thenThrowException() {
    detachedPost.setTitle("Hibernate Tutorial for Absolute Beginners");

    assertThatThrownBy(() -> session.persist(detachedPost))
      .isInstanceOf(PersistenceException.class)
      .hasMessageContaining("org.hibernate.PersistentObjectException: detached entity passed to persist");
}
```

为了避免这种情况，我们应该了解实体状态，并使用适当的方法来保存它。

**如果我们使用`merge`方法，Hibernate 会根据@** `**Id**` **字段**将实体重新附加到持久性上下文:

```
@Test
public void givenDetachedPost_whenTryingToMerge_thenNoExceptionIsThrown() {
    detachedPost.setTitle("Hibernate Tutorial for Beginners");

    session.merge(detachedPost);
    session.getTransaction().commit();

    List<Post> posts = session.createQuery("Select p from Post p", Post.class).list();
    assertThat(posts).hasSize(1);
    assertThat(posts.get(0).getTitle())
        .isEqualTo("Hibernate Tutorial for Beginners");
}
```

同样，我们也可以使用其他特定于 Hibernate 的方法，比如 [`update`、`save`和`saveOrUpdate`](/web/20220810173509/https://www.baeldung.com/hibernate-save-persist-update-merge-saveorupdate) 。与`persist`和`merge, `不同，这些方法不是 JPA 规范的一部分。因此，如果我们想使用 JPA 抽象，就应该避免使用它们。

## 4.试图通过关联保持分离的实体

对于这个例子，我们将介绍`Comment`实体:

```
@Entity
public class Comment {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String text;

    @ManyToOne(cascade = CascadeType.MERGE)
    private Post post;

    // constructor, getters and setters
}
```

我们可以注意到,`Comment`实体与`Post`有多对一的关系。

将[级联类型](/web/20220810173509/https://www.baeldung.com/jpa-cascade-types)设置为`CascadeType.MERGE`。因此，我们将只把`merge`操作传播给相关的`Post`。

换句话说，如果我们`merge`一个`Comment`实体，Hibernate 将把操作传播给相关的`Post`，两个实体都将在数据库中被更新。然而，如果我们想用这个设置`persist`一个`Comment`，我们必须首先`merge`相关联的`Post`:

```
@Test
public void givenDetachedPost_whenMergeAndPersistComment_thenNoExceptionIsThrown() {
    Comment comment = new Comment("nice article!");
    Post mergedPost = (Post) session.merge(detachedPost);
    comment.setPost(mergedPost);

    session.persist(comment);
    session.getTransaction().commit();

    List<Comment> comments = session.createQuery("Select c from Comment c", Comment.class).list();
    Comment savedComment = comments.get(0);
    assertThat(savedComment.getText()).isEqualTo("nice article!");
    assertThat(savedComment.getPost().getTitle())
        .isEqualTo("Hibernate Tutorial");
}
```

**另一方面，如果[级联类型](/web/20220810173509/https://www.baeldung.com/jpa-cascade-types)被设置为`PERSIST`或`ALL`，Hibernate 将尝试在分离的关联字段上传播*持久化*操作。**因此，当我们`persist`一个`Post`实体具有这些级联类型之一时，Hibernate 将`persist`相关联的 detached `Comment`，这将导致另一个`PersistentObjectException`。

## 5.结论

在本文中，我们讨论了 Hbernate 的`PersistentObjectException`并了解了其主要原因。

我们可以通过正确使用 [Hibernate 的`save`、`persist`、`update`、`merge`、`saveOrUpdate`、T6 方法来避免。](/web/20220810173509/https://www.baeldung.com/hibernate-save-persist-update-merge-saveorupdate)

此外，对 [JPA 级联类型](/web/20220810173509/https://www.baeldung.com/jpa-cascade-types)的良好利用将防止`PersistentObjectException`在我们的实体关联中发生。

和往常一样，这篇文章的源代码可以在 GitHub 的[上找到。](https://web.archive.org/web/20220810173509/https://github.com/eugenp/tutorials/tree/master/persistence-modules/hibernate-exceptions)