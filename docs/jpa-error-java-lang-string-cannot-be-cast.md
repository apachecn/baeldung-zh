# 修复 JPA 错误“java.lang.String 无法转换为 ljava . lang . string；”

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/jpa-error-java-lang-string-cannot-be-cast>

 ![](img/7635ed396b1af926a19b2621e27cd573.png)

JPA can behave very differently depending on the exact circumstances under which it is used. Code that works in our local environment or in staging performs very poorly (or even flat out fails) when thrown against real-scale databases in production environments.

Debugging these JPA issues in production is pretty difficult - existing APMs don’t provide enough granular insights at the code level, and tracking every single place someone queried entities one by one instead of in bulk can be a grueling, time-consuming task.

**Lightrun** is a new approach to debugging in production. Using Lightrun’s Logs and Snapshots, you can now get debugger-level granularity in production without opening inbound ports, redeploying, restarting, or even stropping the running application.

In addition, instrumenting Lightrun Metrics at runtime allows you to track down persistence issues securely and in real-time. Want to see it in action? **Check out our 2-minute tutorial** for debugging JPA performance issues in production using Lightrun:

[>> Debugging Spring Persistence and JPA Issues Using Lightrun](/web/20220524032728/https://www.baeldung.com/lightrun-n-jpa)

## 1。简介

当然，我们从来没有想过可以在 Java 中将一个`String` 转换成一个`String `数组:

```java
java.lang.String cannot be cast to [Ljava.lang.String;
```

但是，这是一个常见的 JPA 错误。

在这个快速教程中，我们将展示这个问题是如何出现的，以及如何解决它。

## 2。JPA 中的常见错误案例

在 JPA 中，当我们处理本地查询并使用`EntityManager`的`createNativeQuery`方法时，得到这个错误**并不罕见。**

它的 [Javadoc](https://web.archive.org/web/20220524032728/https://docs.oracle.com/javaee/7/api/javax/persistence/EntityManager.html#createNativeQuery-java.lang.String-) 实际上警告我们**该方法将返回一个`Object[]`列表，或者如果查询只返回一列，则只返回一个`Object`。**

让我们看一个例子。首先，让我们创建一个查询执行器，我们希望重用它来执行我们所有的查询:

```java
public class QueryExecutor {
    public static List<String[]> executeNativeQueryNoCastCheck(String statement, EntityManager em) {
        Query query = em.createNativeQuery(statement);
        return query.getResultList();
    }
}
```

如上所述，我们使用的是`createNativeQuery()`方法，我们总是期望结果集包含一个`String`数组。

之后，让我们创建一个简单的实体用于我们的示例:

```java
@Entity
public class Message {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String text;

    // getters and setters

}
```

最后，让我们创建一个测试类，在运行测试之前插入一个`Message`:

```java
public class SpringCastUnitTest {

    private static EntityManager em;
    private static EntityManagerFactory emFactory;

    @BeforeClass
    public static void setup() {
        emFactory = Persistence.createEntityManagerFactory("jpa-h2");
        em = emFactory.createEntityManager();

        // insert an object into the db
        Message message = new Message();
        message.setText("text");

        EntityTransaction tr = em.getTransaction();
        tr.begin();
        em.persist(message);
        tr.commit();
    }
}
```

现在，我们可以使用我们的`QueryExecutor`来执行一个查询，检索我们实体的`text`字段:

```java
@Test(expected = ClassCastException.class)
public void givenExecutorNoCastCheck_whenQueryReturnsOneColumn_thenClassCastThrown() {
    List<String[]> results = QueryExecutor.executeNativeQueryNoCastCheck("select text from message", em);

    // fails
    for (String[] row : results) {
        // do nothing
    }
}
```

我们可以看到，**因为查询中只有一列，JPA 实际上会返回一个字符串列表，而不是字符串数组列表。**我们得到一个`ClassCastException`,因为查询返回一个列，而我们期望的是一个数组。

## 3.手动铸造修复

**修复这个错误最简单的方法是检查结果集对象的类型**以避免`ClassCastException.` 让我们在`QueryExecutor`中实现一个这样做的方法:

```java
public static List<String[]> executeNativeQueryWithCastCheck(String statement, EntityManager em) {
    Query query = em.createNativeQuery(statement);
    List results = query.getResultList();

    if (results.isEmpty()) {
        return new ArrayList<>();
    }

    if (results.get(0) instanceof String) {
        return ((List<String>) results)
          .stream()
          .map(s -> new String[] { s })
          .collect(Collectors.toList());
    } else {
        return (List<String[]>) results;
    }
}
```

然后，我们可以使用这个方法来执行我们的查询，而不会出现异常:

```java
@Test
public void givenExecutorWithCastCheck_whenQueryReturnsOneColumn_thenNoClassCastThrown() {
    List<String[]> results = QueryExecutor.executeNativeQueryWithCastCheck("select text from message", em);
    assertEquals("text", results.get(0)[0]);
}
```

这不是一个理想的解决方案，因为我们必须将结果转换为一个数组，以防查询只返回一列。

## 4.JPA 实体映射修复

修复这个错误的另一种方法是将结果集映射到一个实体。这样，**我们可以预先决定如何映射我们的查询结果**并避免不必要的转换。

让我们为我们的执行器添加另一个方法来支持自定义实体映射的使用:

```java
public static <T> List<T> executeNativeQueryGeneric(String statement, String mapping, EntityManager em) {
    Query query = em.createNativeQuery(statement, mapping);
    return query.getResultList();
}
```

之后，让我们创建一个自定义的`SqlResultSetMapping `来将我们之前查询的结果集映射到一个`Message`实体:

```java
@SqlResultSetMapping(
  name="textQueryMapping",
  classes={
    @ConstructorResult(
      targetClass=Message.class,
      columns={
        @ColumnResult(name="text")
      }
    )
  }
)
@Entity
public class Message {
    // ...
}
```

在这种情况下，我们还必须添加一个与我们新创建的`SqlResultSetMapping`相匹配的构造函数:

```java
public class Message {

    // ... fields and default constructor

    public Message(String text) {
        this.text = text;
    }

    // ... getters and setters

}
```

最后，我们可以使用新的 executor 方法来运行我们的测试查询，并获得一个`Message`列表:

```java
@Test
public void givenExecutorGeneric_whenQueryReturnsOneColumn_thenNoClassCastThrown() {
    List<Message> results = QueryExecutor.executeNativeQueryGeneric(
      "select text from message", "textQueryMapping", em);
    assertEquals("text", results.get(0).getText());
}
```

这个解决方案更加简洁，因为我们将结果集映射委托给 JPA。

## 5。结论

在本文中，我们已经展示了本地查询是获得这个`ClassCastException`的常见地方。我们还研究了如何自己进行类型检查，以及如何通过将查询结果映射到传输对象来解决这个问题。

与往常一样，GitHub 上的[提供了示例的完整源代码。](https://web.archive.org/web/20220524032728/https://github.com/eugenp/tutorials/tree/master/persistence-modules/java-jpa)