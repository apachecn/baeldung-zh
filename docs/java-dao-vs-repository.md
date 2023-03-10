# DAO 与存储库模式

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-dao-vs-repository>

## 1.概观

通常，repository 和 DAO 的实现被认为是可以互换的，尤其是在以数据为中心的应用程序中。这就造成了对他们差异的混淆。

在本文中，我们将讨论 DAO 和存储库模式之间的区别。

## 2.道模式

数据访问对象模式，又名 [**DAO 模式**](/web/20221129010103/https://www.baeldung.com/java-dao-pattern) 、**，是数据持久性的抽象，被认为更接近底层存储，通常以表为中心**。

因此，在许多情况下，我们的 Dao 匹配数据库表，允许以更直接的方式从存储中发送/检索数据，隐藏难看的查询。

让我们研究一下 DAO 模式的一个简单实现。

### 2.1.`User`

首先，让我们创建一个基本的`User`域类:

```java
public class User {
    private Long id;
    private String userName;
    private String firstName;
    private String email;

    // getters and setters
}
```

### 2.2.`UserDao`

然后，我们将创建为`User`域提供简单 CRUD 操作的`UserDao`接口:

```java
public interface UserDao {
    void create(User user);
    User read(Long id);
    void update(User user);
    void delete(String userName);
}
```

### 2.3.`UserDaoImpl`

最后，我们将创建实现`UserDao`接口的`UserDaoImpl`类:

```java
public class UserDaoImpl implements UserDao {
    private final EntityManager entityManager;

    @Override
    public void create(User user) {
        entityManager.persist(user);
    }

    @Override
    public User read(long id) {
        return entityManager.find(User.class, id);
    }

    // ...
}
```

这里，为了简单起见，我们使用了 [JPA `EntityManager`接口](/web/20221129010103/https://www.baeldung.com/hibernate-entitymanager)来与底层存储交互，并为`User`域提供数据访问机制。

## 3.知识库模式

根据 [Eric Evans 的书`Domain-Driven Design`](https://web.archive.org/web/20221129010103/https://www.pearson.com/store/p/domain-driven-design-tackling-complexity-in-the-heart-of-software/P100000775942/9780321125217) ，**“存储库是一种封装存储、检索和搜索行为的机制，它模拟了一组对象。”**

同样，根据 [`Patterns of Enterprise Application Architecture`](https://web.archive.org/web/20221129010103/https://www.pearson.com/store/p/patterns-of-enterprise-application-architecture/P100001391761/9780321127426) ，它**“使用类似集合的接口访问域对象，在域和数据映射层之间进行协调。”**

换句话说，存储库也处理数据并隐藏类似于 DAO 的查询。然而，它位于更高的层次，更接近于应用程序的业务逻辑。

因此，存储库可以使用 DAO 从数据库获取数据并填充域对象。或者，它可以准备来自域对象的数据，并使用 DAO 将其发送到存储系统进行持久化。

让我们检查一下`User`域的存储库模式的一个简单实现。

### 3.1.`UserRepository`

首先，让我们创建`UserRepository`接口:

```java
public interface UserRepository {
    User get(Long id);
    void add(User user);
    void update(User user);
    void remove(User user);
}
```

这里，我们添加了一些常用方法，如`get`、`add`、`update`和`remove`来处理对象集合。

### 3.2.`UserRepositoryImpl`

然后，我们将创建`UserRepositoryImpl` 类，提供`UserRepository`接口的实现:

```java
public class UserRepositoryImpl implements UserRepository {
    private UserDaoImpl userDaoImpl;

    @Override
    public User get(Long id) {
        User user = userDaoImpl.read(id);
        return user;
    }

    @Override
    public void add(User user) {
        userDaoImpl.create(user);
    }

    // ...
}
```

这里，我们使用了`UserDaoImpl`来发送/检索数据库中的数据。

到目前为止，我们可以说 DAO 和 repository 的实现看起来非常相似，因为`User`类是一个贫血的域。而且，存储库只是数据访问层(DAO)之上的另一层。

然而，DAO 似乎是访问数据的完美候选，而存储库是实现业务用例的理想方式。

## 4.具有多个 Dao 的存储库模式

为了清楚地理解最后一个陈述，让我们增强我们的`User`域来处理一个业务用例。

想象一下，我们想要通过聚合用户的 Twitter tweets、脸书帖子等来准备用户的社交媒体资料。

### 4.1.`Tweet`

首先，我们将创建带有几个保存 tweet 信息的属性的`Tweet`类:

```java
public class Tweet {
    private String email;
    private String tweetText;    
    private Date dateCreated;

    // getters and setters
}
```

### 4.2.`TweetDao` 和 `TweetDaoImpl`

然后，类似于`UserDao`，我们将创建允许获取 tweets 的`TweetDao`接口:

```java
public interface TweetDao {
    List<Tweet> fetchTweets(String email);    
}
```

同样，我们将创建提供`fetchTweets`方法实现的`TweetDaoImpl`类:

```java
public class TweetDaoImpl implements TweetDao {
    @Override
    public List<Tweet> fetchTweets(String email) {
        List<Tweet> tweets = new ArrayList<Tweet>();

        //call Twitter API and prepare Tweet object

        return tweets;
    }
}
```

这里，我们将调用 Twitter APIs 来获取用户使用其电子邮件发送的所有 tweets。

因此，在这种情况下，DAO 使用第三方 API 提供数据访问机制。

### 4.3.增强`User`域

最后，让我们创建`User`类的`UserSocialMedia`子类来保存`Tweet`对象的列表:

```java
public class UserSocialMedia extends User {
    private List<Tweet> tweets;

    // getters and setters
}
```

这里，我们的`UserSocialMedia`类是一个复杂的域，也包含了`User`域的属性。

### 4.4.`UserRepositoryImpl`

现在，我们将升级我们的`UserRepositoryImpl` 类来提供一个`User`域对象以及一列推文:

```java
public class UserRepositoryImpl implements UserRepository {
    private UserDaoImpl userDaoImpl;
    private TweetDaoImpl tweetDaoImpl;

    @Override
    public User get(Long id) {
        UserSocialMedia user = (UserSocialMedia) userDaoImpl.read(id);

        List<Tweet> tweets = tweetDaoImpl.fetchTweets(user.getEmail());
        user.setTweets(tweets);

        return user;
    }
}
```

这里，`UserRepositoryImpl`使用`UserDaoImpl`提取用户数据，使用`TweetDaoImpl.`提取用户的推文

然后，它聚集两组信息，并提供一个`UserSocialMedia`类的域对象，这对我们的业务用例来说很方便。因此，**存储库依赖 DAOs 来访问来自不同来源的数据**。

类似地，我们可以增强我们的`User`域来保存脸书帖子的列表。

## 5.比较这两种模式

现在我们已经看到了 DAO 和存储库模式的细微差别，让我们总结一下它们的区别:

*   DAO 是数据持久性的抽象。然而，存储库是对象集合的抽象
*   DAO 是一个较低级的概念，更接近于存储系统。然而，存储库是一个更高层次的概念，更接近于领域对象
*   DAO 作为数据映射/访问层工作，隐藏难看的查询。然而，存储库是域和数据访问层之间的一层，隐藏了整理数据和准备域对象的复杂性
*   DAO 不能使用存储库来实现。然而，存储库可以使用 DAO 来访问底层存储

此外，如果我们有一个贫血的域，那么存储库将只是一个 DAO。

此外，**存储库模式鼓励领域驱动的设计，为非技术团队成员提供了对数据结构的简单理解**。

## 6.结论

在本文中，我们探讨了 DAO 和存储库模式之间的差异。

首先，我们研究了 DAO 模式的一个基本实现。然后，我们看到了一个使用存储库模式的类似实现。

最后，我们看了一个利用多个 Dao 的存储库，增强了一个领域解决一个业务用例的能力。

因此，我们可以得出结论，当应用程序从以数据为中心转向面向业务时，存储库模式证明是一种更好的方法。

像往常一样，所有的代码实现都可以在 GitHub 上获得[。](https://web.archive.org/web/20221129010103/https://github.com/eugenp/tutorials/tree/master/patterns-modules/design-patterns-architectural)