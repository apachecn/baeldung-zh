# 使用 Spring Boot 为 MongoDB 自动生成字段

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-boot-mongodb-auto-generated-field>

## 1。概述

在本教程中，我们将学习如何为 Spring Boot 的 MongoDB 实现一个连续的、自动生成的字段。

**当我们使用 MongoDB 作为 Spring Boot 应用程序的数据库时，我们不能在模型中使用`@GeneratedValue`注释，因为它不可用。**因此，我们需要一种方法来产生与使用 JPA 和 SQL 数据库相同的效果。

这个问题的一般解决方案很简单。我们将创建一个集合(表),存储为其他集合生成的序列。在创建新记录的过程中，我们将使用它来获取下一个值。

## 2。依赖性

让我们将以下弹簧靴起动器添加到我们的`pom.xml`:

```java
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
        <versionId>2.2.2.RELEASE</versionId>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-data-mongodb</artifactId>
        <versionId>2.2.2.RELEASE</versionId>
    </dependency>
</dependencies>
```

依赖关系的最新版本由 [`spring-boot-starter-parent`](https://web.archive.org/web/20220526050715/https://search.maven.org/classic/#search%7Cga%7C1%7Cg%3Aorg.springframework.boot%20a%3Aspring-boot-starter-parent) 管理。

## 3。收藏

正如概述中所讨论的，我们将创建一个集合，为其他集合存储自动递增的序列。我们将这个集合称为`database_sequences. `，它可以使用`mongo` shell 或 MongoDB Compass 来创建。让我们创建一个相应的模型类:

```java
@Document(collection = "database_sequences")
public class DatabaseSequence {

    @Id
    private String id;

    private long seq;

    //getters and setters omitted
}
```

然后让我们创建一个`users`集合，以及一个相应的模型对象，它将存储使用我们系统的人的详细信息:

```java
@Document(collection = "users")
public class User {

    @Transient
    public static final String SEQUENCE_NAME = "users_sequence";

    @Id
    private long id;

    private String email;

    //getters and setters omitted
}
```

在上面创建的`User`模型中，我们添加了一个静态字段`SEQUENCE_NAME,`，它是对`users`集合的自动递增序列的唯一引用。

我们还用`@Transient`对它进行了注释，以防止它与模型的其他属性一起被持久化。

## 4。创建新记录

到目前为止，我们已经创建了所需的集合和模型。现在，我们将创建一个服务，该服务将生成自动递增的值，该值可用作实体的`id`。

让我们创建一个拥有`generateSequence()`的`SequenceGeneratorService`:

```java
public long generateSequence(String seqName) {
    DatabaseSequence counter = mongoOperations.findAndModify(query(where("_id").is(seqName)),
      new Update().inc("seq",1), options().returnNew(true).upsert(true),
      DatabaseSequence.class);
    return !Objects.isNull(counter) ? counter.getSeq() : 1;
}
```

现在，我们可以在创建新记录时使用`generateSequence()`:

```java
User user = new User();
user.setId(sequenceGenerator.generateSequence(User.SEQUENCE_NAME));
user.setEmail("[[email protected]](/web/20220526050715/https://www.baeldung.com/cdn-cgi/l/email-protection)");
userRepository.save(user);
```

为了列出所有用户，我们将使用`UserRepository`:

```java
List<User> storedUsers = userRepository.findAll();
storedUsers.forEach(System.out::println);
```

像现在这样，每次我们创建模型的新实例时，我们都必须设置 id 字段。我们可以通过为 Spring Data MongoDB 生命周期事件创建一个监听器来规避这个过程。

**为此，我们将创建一个扩展`AbstractMongoEventListener<User>`的`UserModelListener`，然后我们将覆盖`onBeforeConvert()` :**

```java
@Override
public void onBeforeConvert(BeforeConvertEvent<User> event) {
    if (event.getSource().getId() < 1) {
        event.getSource().setId(sequenceGenerator.generateSequence(User.SEQUENCE_NAME));
    }
}
```

现在，每当我们保存一个新的`User,`时，`id`将被自动设置。

## 5。结论

总之，我们已经看到了如何为 id 字段生成连续的、自动递增的值，并模拟了与 SQL 数据库中相同的行为。

默认情况下，Hibernate 使用类似的方法生成自动递增的值。

和往常一样，完整的源代码可以在 Github 的[上找到。](https://web.archive.org/web/20220526050715/https://github.com/eugenp/tutorials/tree/master/persistence-modules/spring-boot-persistence-mongodb)