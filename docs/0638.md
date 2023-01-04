# 在 MongoDB 中生成唯一的 ObjectId

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/mongo-generate-unique-objectid>

## 1.介绍

在本文中，我们将讨论什么是`ObjectId`,我们如何生成它，以及确保其唯一性的可能方法。

## 2.`ObjectId`一般信息

让我们从解释什么是`ObjectId`开始。 **An `ObjectId`是一个 12 字节的十六进制值**，是 [BSON 规范](https://web.archive.org/web/20221108214334/https://bsonspec.org/)中可能的数据类型之一。BSON 是一个 JSON 文档的二进制序列化。此外，MongoDB 使用`ObjectId`作为文档中`_id`字段的默认标识符。在创建集合时，`_id`字段上还有一个默认的唯一索引。

这可以防止用户插入两个具有相同`_id. `的文档。此外，不能从集合中删除`_id`索引。但是，可以将具有相同 _id 的单个文档插入到两个集合中。

### 2.1.`ObjectId`结构

`ObjectId`可分为三个不同的部分。考虑到`6359388c80616b1fc6d7ec71,`的`ObjectId`，第一部分将由 4 个字节组成–`6359388c. `这 4 个字节代表自 Unix 纪元以来以秒为单位的时间。第二部分由接下来的 5 个字节组成，它们是`80616b1fc6`T5。这些字节代表每个进程产生一次的随机值。随机值对于机器和过程是唯一的。最后一部分是 3 个字节`d7ec71, `，它代表一个从随机值开始的递增计数器。

另外值得一提的是**以上结构对 MongoDB 4.0 及以上版本**有效。在此之前，`ObjectId`号由四部分组成。前 4 个字节表示自 Unix 纪元以来的秒数，接下来的 3 个字节表示机器标识符。

接下来的 2 个字节用于进程 id，最后 3 个字节用于计数器，从一个随机值开始。

### 2.2.`ObjectId`独特性

最重要的是，在 MongoDB 文档中也提到了，当生成时，`ObjectId`是**极有可能被认为是唯一的** **。也就是说，生成重复的`ObjectId`的可能性非常小。查看`ObjectId,`的结构，我们可以看到`ObjectId`在一秒钟内产生的可能性超过 1,8×10^19。**

即使所有的 id 都是在同一秒钟内在同一台机器的同一进程中生成的，仅计数器本身就有超过 1700 万种可能性。

## 3.`ObjectId`创作

Java 中有多种创建`ObjectId`的方法。它可以用非参数或参数化的构造函数来完成。

### 3.1.`ObjectId`用非参数化的构造函数创建

第一种也是最简单的一种方法是通过一个带有非参数化构造函数的新关键字:

```
ObjectId objectId = new ObjectId();
```

第二种方法是简单地调用一个`ObjectId`类上的静态方法`get()`。不直接调用非参数化的构造函数。然而，`get()`方法的实现包括创建与第一个例子相同的`ObjectId`——通过新的关键字:

```
ObjectId objectId = ObjectId.get();
```

### 3.2.`ObjectId`用参数化构造函数创建

其余的例子使用参数化的构造函数。我们可以通过将`Date`类作为参数传递或者将`Date`类和`int`计数器都作为参数传递来创建一个`ObjectId`。如果我们试图在两种方法中用相同的`Date`创建`ObjectId`，我们将得到不同的`ObjectId``new ObjectId(date)`和`new ObjectId(date, counter)`。

然而，如果我们在同一秒钟内创建两个`ObjectId`到`new ObjectId(date, counter)`，我们将得到一个重复的`ObjectId`，因为它是在同一秒钟内，在同一台机器上，用同一计数器生成的。让我们看一个例子:

```
@Test
public void givenSameDateAndCounter_whenComparingObjectIds_thenTheyAreNotEqual() {
    Date date = new Date();
    ObjectId objectIdDate = new ObjectId(date); // 635981f6e40f61599e839ddb
    ObjectId objectIdDateCounter1 = new ObjectId(date, 100); // 635981f6e40f61599e000064
    ObjectId objectIdDateCounter2 = new ObjectId(date, 100); // 635981f6e40f61599e000064

    assertThat(objectIdDate).isNotEqualTo(objectIdDateCounter1);
    assertThat(objectIdDate).isNotEqualTo(objectIdDateCounter2);

    assertThat(objectIdDateCounter1).isEqualTo(objectIdDateCounter2);
}
```

此外，可以通过直接提供一个十六进制值作为参数来创建`ObjectId`:

```
ObjectId objectIdHex = new ObjectId("635981f6e40f61599e000064");
```

创建一个`ObjectId`还有几种可能性。我们可以通过`byte[]`或`ByteBuffer`班。如果我们通过向构造函数传递一个字节数组来创建一个`ObjectId`，我们应该通过使用相同的字节数组通过`ByteBuffer`类来创建它，从而获得相同的`ObjectId`。

让我们看一个例子:

```
@Test
public void givenSameArrayOfBytes_whenComparingObjectIdsCreatedViaDifferentMethods_thenTheObjectIdsAreEqual(){
    byte[] bytes = "123456789012".getBytes();
    ObjectId objectIdBytes = new ObjectId(bytes);

    ByteBuffer buffer = ByteBuffer.wrap(bytes);
    ObjectId objectIdByteBuffer = new ObjectId(buffer);

    assertThat(objectIdBytes).isEqualTo(objectIdByteBuffer);
}
```

最后一种可能的方法是通过向构造函数传递时间戳和计数器来创建一个`ObjectId`。

## 4.`ObjectId`的利弊

正如所有的事情一样，有值得了解的优点和缺点。

### 4.1.`ObjectId`的好处

因为`ObjectId`是 12 字节长，所以它比 16 字节的 UUID 小。也就是说，如果我们在数据库中有很多使用`ObjectId`而不是 UUID 的文档，我们将节省一些空间。大约 26500 次使用`ObjectId`将比 UUID 节省 1MB。这似乎是最低限度的金额。

尽管如此，如果数据库足够大，单个文档也有可能出现不止一次的`ObjectId,`，那么磁盘空间和 RAM 的**增益可能是显著的，因为文档最终会变得更小。其次，正如我们之前所了解的，时间戳被嵌入到 ObjectId 中，这在某些情况下可能是有用的。**

例如，为了确定哪个`ObjectId`是首先创建的，假设它们都是自动生成的，而不是像我们之前看到的那样通过将`Date`类操作到参数化的构造函数中来创建的。

### 4.2.`ObjectId`的弊端

另一方面，有些标识符甚至比 12 字节的`ObjectId,`还要小，这将再次节省更多的磁盘空间和 RAM。此外，由于`ObjectId`只是一个生成的十六进制值，这意味着有一个重复 id 的**可能性。很渺茫，但还是有可能的。**

## 5.确保`ObjectId`的唯一性

如果我们必须确保生成的`ObjectId`是独一无二的，我们可以尝试围绕它编写一点程序，以 100%确保它不是重复的。

### 5.1.试抓`DuplicateKeyException`

假设我们在数据库中插入一个已经有 field _id 的文档。在这种情况下，我们可以捕获 **a `DuplicateKeyException`并重试插入操作，直到它成功**。这种方法只对创建了唯一索引的**字段有效。**

让我们来看一个例子。考虑一个`User`类:

```
public class User {
    public static final String NAME_FIELD = "name";

    private final ObjectId id;
    private final String name;

    // constructor
    // getters
}
```

我们将向数据库中插入一个`User`，然后尝试插入另一个具有相同`ObjectId`的。这将导致`DuplicateKeyException`被抛出。我们可以捕捉它并重试`User.` 的插入操作，但是，这一次，我们将生成另一个`ObjectId`。出于这个测试的目的，我们将使用一个[嵌入式 MongoDB](/web/20221108214334/https://www.baeldung.com/spring-boot-embedded-mongodb) 库和带有 MongoDB 的 [Spring 数据。](/web/20221108214334/https://www.baeldung.com/spring-data-mongodb-tutorial)

让我们看一个例子:

```
@Test
public void givenUserInDatabase_whenInsertingAnotherUserWithTheSameObjectId_DKEThrownAndInsertRetried() {
    // given
    String userName = "Kevin";
    User firstUser = new User(ObjectId.get(), userName);
    User secondUser = new User(ObjectId.get(), userName);

    mongoTemplate.insert(firstUser);

    // when
    try {
        mongoTemplate.insert(firstUser);
    } catch (DuplicateKeyException dke) {
        mongoTemplate.insert(secondUser);
    }

    // then
    Query query = new Query();
    query.addCriteria(Criteria.where(User.NAME_FIELD)
      .is(userName));
    List<User> users = mongoTemplate.find(query, User.class);
    assertThat(users).usingRecursiveComparison()
      .isEqualTo(Lists.newArrayList(firstUser, secondUser));
}
```

### 5.2.查找并插入

另一种方法(可能不推荐)是找到一个带有给定`ObjectId`的文档，看看它是否存在。如果它不存在，我们可以插入它。否则，抛出一个错误或生成另一个`ObjectId` 并重试。这种方法也不可靠，因为在 MongoDB 中没有**原子查找和插入** **选项，这可能导致不一致**。

**自动生成`ObjectId`并尝试插入文档而不确保其唯一性**是一种常见的方法。在每个 insert 上尝试 catch `DuplicateKeyException`并重试操作似乎是多余的。边缘案例的数量非常有限，如果不首先在`ObjectId`中植入`Date`、计数器或时间戳，就很难重现这样的案例。

然而，如果由于某种原因，我们不能承受由于那些边缘情况而产生的重复`ObjectId`，那么我们会考虑使用上述方法来确保全局唯一性。

## 5.结论

在本文中，我们了解了什么是`ObjectId` ,它是如何构建的，我们如何生成它，以及确保其唯一性的可能方法。最后，信任`ObjectIds`的自动生成似乎是最好的办法。

所有代码示例都可以在 GitHub 上找到[。](https://web.archive.org/web/20221108214334/https://github.com/eugenp/tutorials/tree/master/persistence-modules/spring-data-mongodb-2)