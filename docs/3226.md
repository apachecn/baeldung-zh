# 如何在 JPA 查询中返回多个实体

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/jpa-return-multiple-entities>

## 1.概观

在这个简短的教程中，我们将看到如何在 JPA 查询中返回多个不同的实体。

首先，我们将创建一个包含几个不同实体的简单代码示例。然后，我们将解释如何创建返回多个不同实体的 JPA 查询。最后，我们将展示 Hibernate 的 JPA 实现中的一个工作示例。

## 2.示例配置

在我们解释如何在单个查询中返回多个实体之前，让我们构建一个我们将要处理的示例。

我们将创建一个应用程序，允许其用户购买特定电视频道的订阅。它由三个表格组成:`Channel`、`Subscription`、`User`。

首先，让我们看一下`Channel`实体:

```
@Entity
public class Channel {
    @Id
    @GeneratedValue(strategy = GenerationType.AUTO)
    private Long id;

    private String code;

    private Long subscriptionId;

   // getters, setters, etc.
} 
```

它由映射到相应列的 3 个字段组成。第一个也是最重要的一个是`id, `，它也是主键。在`code`字段中，我们将存储`Channel`的代码。

最后，还有一个`subscriptionId`栏目。它将用于创建频道和它所属的订阅之间的关系。一个频道可以属于不同的订阅。

现在，让我们看看`Subscription`实体:

```
@Entity
public class Subscription {
    @Id
    @GeneratedValue(strategy = GenerationType.AUTO)
    private Long id;

    private String code;

   // getters, setters, etc.
}
```

比第一个还要简单。它由主键字段`id`和订阅字段`code`组成。

让我们再来看看`User`实体:

```
@Entity
public class User {
    @Id
    @GeneratedValue(strategy = GenerationType.AUTO)
    private Long id;

    private String email;

    private Long subscriptionId;

    // getters, setters, etc.
}
```

除主键`id`字段外，还包括`email`和`subscriptionId`字段。后者用于创建用户和他们选择的订阅之间的关系。

## 3.在查询中返回多个实体

### 3.1.创建查询

为了创建一个返回多个不同实体的查询，我们需要做两件事。

首先，我们需要列出希望在 SQL 查询的`SELECT `部分返回的实体，用逗号分隔。

其次，我们需要通过它们的主键和对应的外键将它们相互连接起来。

让我们看看我们的例子。假设我们想获取用户用给定的电子邮件`.`购买的分配给`Subscriptions `的所有【The JPA 查询看起来是这样的:

```
SELECT c, s, u
  FROM Channel c, Subscription s, User u
  WHERE c.subscriptionId = s.id AND s.id = u.subscriptionId AND u.email=:email
```

### 3.2.提取结果

选择多个不同实体的 JPA 查询在一个数组`Objects`中返回它们。值得指出的是，数组保持了实体的顺序。这是至关重要的信息，因为我们需要手动将返回的对象转换为特定的实体类。

让我们看看实际情况。我们创建了一个专用的存储库类，用于创建查询和获取结果:

```
public class ReportRepository {
    private final EntityManagerFactory emf;

    public ReportRepository() {
        // create an instance of entity manager factory
    }

    public List<Object[]> find(String email) {
        EntityManager entityManager = emf.createEntityManager();
        Query query = entityManager
          .createQuery("SELECT c, s, u FROM  Channel c, Subscription s, User u" 
          + " WHERE c.subscriptionId = s.id AND s.id = u.subscriptionId AND u.email=:email");
        query.setParameter("email", eamil);

        return query.getResultList();
    }
} 
```

我们使用的是上一节中的精确查询。然后，我们设置一个电子邮件参数来缩小结果范围。最后，我们获取结果列表。

让我们看看如何从获取的列表中提取单个实体:

```
List<Object[]> reportDetails = reportRepository.find("[[email protected]](/web/20220926191644/https://www.baeldung.com/cdn-cgi/l/email-protection)");

for (Object[] reportDetail : reportDetails) {
    Channel channel = (Channel) reportDetail[0];
    Subscription subscription = (Subscription) reportDetail[1];
    User user = (User) reportDetail[2];

    // do something with entities
}
```

我们迭代获取的列表，并从给定的对象数组中提取实体。记住我们的 JPA 查询和它的`SELECT`部分中实体的顺序，我们得到一个`Channel`实体作为第一个元素，一个`Subscription`实体作为第二个元素，一个`User`实体作为数组的最后一个元素。

## 4.结论

在本文中，w 讨论了如何在 JPA 查询中返回多个实体 。首先，我们创建了一个示例，我们将在本文的后面进行处理。然后，我们解释了如何编写一个 JPA 查询来返回多个不同的实体。最后，我们展示了如何从结果列表中提取它们。

和往常一样，GitHub 上的[提供了这篇文章的完整源代码。](https://web.archive.org/web/20220926191644/https://github.com/eugenp/tutorials/tree/master/persistence-modules/java-jpa-3)