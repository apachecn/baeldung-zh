# Hibernate 中的 MultipleBagFetchException 指南

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-hibernate-multiplebagfetchexception>

## 1.概观

在本教程中，我们将讨论*[MultipleBagFetchException](https://web.archive.org/web/20220628113359/https://docs.jboss.org/hibernate/orm/5.2/javadocs/org/hibernate/loader/MultipleBagFetchException.html)。*我们将从需要理解的必要术语开始，然后探索一些变通方法，直到找到理想的解决方案。

我们将创建一个简单的音乐应用程序的域来演示每个解决方案。

## 2。冬眠中的包是什么？

**一个包，类似于*列表*，是一个可以包含重复元素的集合。然而，这是不妥当的。**此外，包是一个 [Hibernate](/web/20220628113359/https://www.baeldung.com/jpa-hibernate-difference) 术语，不是 Java 集合框架的一部分。

根据前面的定义，值得强调的是 *List* 和 Bag 都使用了 *java.util.List* 。虽然在 Hibernate 中，两者被区别对待。为了将一个包与一个*列表*区分开来，让我们看看实际代码。

一个包:

```java
// @ any collection mapping annotation
private List<T> collection;
```

一个`List`:

```java
// @ any collection mapping annotation
@OrderColumn(name = "position")
private List<T> collection;
```

## 3.`MultipleBagFetchException`的起因

在一个*实体*上同时提取两个或更多的袋子可以形成一个笛卡尔乘积。因为包没有顺序，Hibernate 不能将正确的列映射到正确的实体。因此，在这种情况下，它抛出一个*MultipleBagFetchException*。

下面举一些导致*multiplebagfettechexception 的具体例子。*

对于第一个例子，让我们试着创建一个简单的实体，它有两个袋子，并且都带有eager fetch 类型。一位*艺术家*可能是一个很好的例子。它可以收集*歌曲*和*优惠*。

鉴于此，让我们创建*艺术家*实体:

```java
@Entity
class Artist {

    @Id
    @GeneratedValue(strategy = GenerationType.AUTO)
    private Long id;

    private String name;

    @OneToMany(mappedBy = "artist", fetch = FetchType.EAGER)
    private List<Song> songs;

    @OneToMany(mappedBy = "artist", fetch = FetchType.EAGER)
    private List<Offer> offers;

    // constructor, equals, hashCode
}
```

**如果我们试图运行一个测试，我们将立即遇到一个*multiplebagfettechexception*，它将无法构建 Hibernate `SessionFactory`。**话虽如此，我们还是不要这样做了。

相反，让我们将一个或两个集合的获取类型转换为 lazy:

```java
@OneToMany(mappedBy = "artist")
private List<Song> songs;

@OneToMany(mappedBy = "artist")
private List<Offer> offers;
```

现在，我们将能够创建并运行一个测试。**虽然，如果我们试图同时获取这两个包集合，仍然会导致*multiplebagfettechexception*。**

## 4.模拟一个`MultipleBagFetchException`

在上一节中，我们已经看到了 *MultipleBagFetchException 的原因。*这里，让我们通过创建一个集成测试来验证这些声明。

为了简单起见，我们使用之前创建的 `Artist` 实体。

现在，让我们创建集成测试，让我们尝试使用 JPQL: 同时获取 `songs` 和 `offers`

```java
@Test
public void whenFetchingMoreThanOneBag_thenThrowAnException() {
    IllegalArgumentException exception =
      assertThrows(IllegalArgumentException.class, () -> {
        String jpql = "SELECT artist FROM Artist artist "
          + "JOIN FETCH artist.songs "
          + "JOIN FETCH artist.offers ";

        entityManager.createQuery(jpql);
    });

    final String expectedMessagePart = "MultipleBagFetchException";
    final String actualMessage = exception.getMessage();

    assertTrue(actualMessage.contains(expectedMessagePart));
}
```

从断言中，**我们遇到了一个*IllegalArgumentException**，*，其根本原因是*MultipleBagFetchException*。**

## 5.领域模型

在继续讨论可能的解决方案之前，让我们看看必要的领域模型，稍后我们将使用它们作为参考。

假设我们正在处理一个音乐应用的领域。考虑到这一点，让我们把注意力集中在某些实体上:*专辑、艺术家、*和*用户。*

我们已经看到了 *Artist* 实体，所以让我们继续处理另外两个实体。

首先，让我们看一下`Album`实体:

```java
@Entity
class Album {

    @Id
    @GeneratedValue(strategy = GenerationType.AUTO)
    private Long id;

    private String name;

    @OneToMany(mappedBy = "album")
    private List<Song> songs;

    @ManyToMany(mappedBy = "followingAlbums")
    private Set<Follower> followers;

    // constructor, equals, hashCode

}
```

一个*专辑*有一个`songs`的集合，同时可以有一个`followers`的集合。

接下来，这里是*用户*实体:

```java
@Entity
class User {

    @Id
    @GeneratedValue(strategy = GenerationType.AUTO)
    private Long id;

    private String name;

    @OneToMany(mappedBy = "createdBy", cascade = CascadeType.PERSIST)
    private List<Playlist> playlists;

    @OneToMany(mappedBy = "user", cascade = CascadeType.PERSIST)
    @OrderColumn(name = "arrangement_index")
    private List<FavoriteSong> favoriteSongs;

    // constructor, equals, hashCode
}
```

一个*用户*可以创建多个`playlists`。此外，*用户*具有用于`favoriteSongs`的单独的`List`，其中其顺序基于排列索引。

## 6.解决方法:在单个 JPQL 查询中使用`Set`

在做任何事情之前，让我们强调一下 **这种方法会生成一个笛卡尔积，这只是一种变通方法。** **这是因为我们将在一个 JPQL 查询中同时获取两个集合。** 相比之下，用一个 [`Set`](/web/20220628113359/https://www.baeldung.com/java-set-operations) 没什么不好。如果我们不需要我们的集合有顺序或任何重复的元素，这是合适的选择。

为了演示这种方法，让我们引用领域模型中的*相册*实体。

一个 `Album ` 实体有两个集合:`songs`和`followers`。 `songs` 的集合类型为 bag。**然而，对于** **`followers, `我们使用的是** `**Set.** ` 也就是说，**我们不会遇到** **`MultipleBagFetchException `的情况，即使我们试图同时获取两个集合。**

使用集成测试，让我们尝试通过 id 检索一个`Album`,同时在一个 JPQL 查询中获取它的两个集合:

```java
@Test
public void whenFetchingOneBagAndSet_thenRetrieveSuccess() {
    String jpql = "SELECT DISTINCT album FROM Album album "
      + "LEFT JOIN FETCH album.songs "
      + "LEFT JOIN FETCH album.followers "
      + "WHERE album.id = 1";

    Query query = entityManager.createQuery(jpql)
      .setHint(QueryHints.HINT_PASS_DISTINCT_THROUGH, false);

    assertEquals(1, query.getResultList().size());
}
```

正如我们所看到的，**号我们已经成功取回了一艘`Album`。是因为只有`songs`的单子是个包**。另一方面，**`followers`的集合是一个`Set`** 。

顺便提一下，值得强调的是我们正在使用 `QueryHints.HINT_PASS_DISTINCT_THROUGH. ` 因为我们正在使用实体 JPQL 查询，它防止了 `DISTINCT` 关键字被包含在实际的 SQL 查询中。因此，我们也将这个查询提示用于其余的方法。

## 7.解决方法:在单个 JPQL 查询中使用`List`

与上一节类似， **这也会生成笛卡尔积，从而可能导致性能问题** 。同样，使用 `[List](/web/20220628113359/https://www.baeldung.com/java-arraylist), Set,` 或 Bag 作为数据类型也没什么问题。本节的目的是进一步演示 Hibernate 可以同时获取集合，前提是 Bag 类型不超过一个。

对于这种方法，让我们使用来自我们领域模型的 `User` 实体。

前面说过，一个 `User` 有两个集合:`playlists`和`favoriteSongs`。****`playlists`没有定义顺序，使其成为一个包集合。但是对于`favoriteSongs`** **的`List`，其顺序取决于`User`如何排列**。如果我们仔细观察`FavoriteSong`实体，`arrangementIndex`属性使得这样做成为可能。**

 **同样，使用单个 JPQL 查询，让我们尝试验证我们是否能够在同时获取 `playlists` 和 `favoriteSongs` 的集合时检索所有用户。

为了演示，让我们创建一个集成测试:

```java
@Test
public void whenFetchingOneBagAndOneList_thenRetrieveSuccess() {
    String jpql = "SELECT DISTINCT user FROM User user "
      + "LEFT JOIN FETCH user.playlists "
      + "LEFT JOIN FETCH user.favoriteSongs ";

    List<User> users = entityManager.createQuery(jpql, User.class)
      .setHint(QueryHints.HINT_PASS_DISTINCT_THROUGH, false)
      .getResultList();

    assertEquals(3, users.size());
}
```

从断言中，我们可以看到我们已经成功地检索到了所有用户。而且， **我们没遇到过 `MultipleBagFetchException` 。这是因为尽管我们正在同时获取两个系列，但只有 `playlists` 是一个包系列。**

## 8.理想的解决方案:使用多个查询

从前面的解决方案中，我们已经看到了使用单个 JPQL 查询来同时检索集合。不幸的是，它生成了一个笛卡尔积。我们知道这并不理想。所以在这里，让我们在不牺牲性能的情况下解决 `MultipleBagFetchException` 。

假设我们正在处理一个拥有多个箱包系列的实体。在我们的例子中，**就是** **`Artist`实体。它有两个包系列:`songs`和`offers`。**

在这种情况下，**我们甚至不能使用单个 JPQL 查询同时获取两个集合。这样做会导致`MultipleBagFetchException`。相反，让我们把它分成两个 JPQL 查询。**

通过这种方法，我们希望能成功地同时获得两个包系列，一次一个。

再一次，也是最后一次，让我们快速创建一个检索所有艺术家的集成测试:

```java
@Test
public void whenUsingMultipleQueries_thenRetrieveSuccess() {
    String jpql = "SELECT DISTINCT artist FROM Artist artist "
      + "LEFT JOIN FETCH artist.songs ";

    List<Artist> artists = entityManager.createQuery(jpql, Artist.class)
      .setHint(QueryHints.HINT_PASS_DISTINCT_THROUGH, false)
      .getResultList();

    jpql = "SELECT DISTINCT artist FROM Artist artist "
      + "LEFT JOIN FETCH artist.offers "
      + "WHERE artist IN :artists ";

    artists = entityManager.createQuery(jpql, Artist.class)
      .setParameter("artists", artists)
      .setHint(QueryHints.HINT_PASS_DISTINCT_THROUGH, false)
      .getResultList();

    assertEquals(2, artists.size());
}
```

在测试中，我们首先检索所有艺术家，同时获取其集合`songs`。

然后，我们创建了另一个查询来获取艺术家的`offers`。

**使用这种方法，我们避免了`MultipleBagFetchException`以及笛卡尔积的形成。**

## 9.结论

在本文中，我们已经详细探讨了 `MultipleBagFetchException` 。我们讨论了必要的词汇和这个异常的原因。然后我们进行了模拟。之后，我们讨论了一个简单的音乐应用程序的领域，它为我们的每一个解决方案和理想的解决方案提供了不同的场景。最后，我们设置了几个集成测试来验证每种方法。

和往常一样，GitHub 上的[提供了这篇文章的完整源代码。](https://web.archive.org/web/20220628113359/https://github.com/eugenp/tutorials/tree/master/persistence-modules/java-jpa-3)**