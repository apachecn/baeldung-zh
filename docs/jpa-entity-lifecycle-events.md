# JPA 实体生命周期事件

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/jpa-entity-lifecycle-events>

## 1.介绍

当使用 JPA 时，在一个实体的生命周期中，有几个事件可以通知我们。在本教程中，我们将讨论 JPA 实体生命周期事件，以及当这些事件发生时，我们如何使用注释来处理回调和执行代码。

我们将从注释实体本身的方法开始，然后继续使用实体侦听器。

## 2.JPA 实体生命周期事件

JPA 指定了七个可选的生命周期事件，称为:

*   在为新实体调用 persist 之前-`@PrePersist`
*   在为新实体调用 persist 之后—`@PostPersist`
*   实体移除前-`@PreRemove`
*   删除实体后-`@PostRemove`
*   更新操作前-`@PreUpdate`
*   实体更新后-`@PostUpdate`
*   实体加载后-`@PostLoad`

**有两种使用生命周期事件注释的方法:**在实体中注释方法和创建一个带有注释回调方法的`EntityListener`。我们也可以两者同时使用。不管在哪里，**回调方法都需要有一个`void`返回类型。**

因此，如果我们创建一个新的实体，并调用我们的存储库的`save`方法，我们用`@PrePersist`注释的方法被调用，然后记录被插入数据库，最后，我们的`@PostPersist`方法被调用。**如果我们使用`@GeneratedValue`来自动生成我们的主键，我们可以期望这个键在`@PostPersist`方法中可用。**

对于`@PostPersist`、`@PostRemove`和`@PostUpdate`操作，文档提到这些事件可以在操作发生后、刷新后或事务结束时立即发生。

**我们应该注意到，只有当数据实际发生变化**时，才会调用`@PreUpdate`回调，也就是说，如果有一个实际的 SQL update 语句要运行。不管实际上是否发生了变化，都会调用`@PostUpdate`回调。

如果我们任何持久化或移除实体的回调抛出异常，事务将被回滚。

## 3.注释`Entity`

让我们从直接在实体中使用回调注释开始。在我们的例子中，当`User`记录被更改时，我们将留下一个日志记录，所以我们将在回调方法中添加简单的日志记录语句。

此外，我们希望确保在从数据库中加载用户后，收集用户的全名。我们将通过用`@PostLoad`注释一个方法来实现。

我们将从定义我们的`User`实体开始:

```java
@Entity
public class User {
    private static Log log = LogFactory.getLog(User.class);

    @Id
    @GeneratedValue
    private int id;

    private String userName;
    private String firstName;
    private String lastName;
    @Transient
    private String fullName;

    // Standard getters/setters
}
```

接下来，我们需要创建一个`UserRepository`接口:

```java
public interface UserRepository extends JpaRepository<User, Integer> {
    public User findByUserName(String userName);
}
```

现在，让我们返回到我们的`User`类并添加我们的回调方法:

```java
@PrePersist
public void logNewUserAttempt() {
    log.info("Attempting to add new user with username: " + userName);
}

@PostPersist
public void logNewUserAdded() {
    log.info("Added user '" + userName + "' with ID: " + id);
}

@PreRemove
public void logUserRemovalAttempt() {
    log.info("Attempting to delete user: " + userName);
}

@PostRemove
public void logUserRemoval() {
    log.info("Deleted user: " + userName);
}

@PreUpdate
public void logUserUpdateAttempt() {
    log.info("Attempting to update user: " + userName);
}

@PostUpdate
public void logUserUpdate() {
    log.info("Updated user: " + userName);
}

@PostLoad
public void logUserLoad() {
    fullName = firstName + " " + lastName;
}
```

当我们运行测试时，我们会看到一系列来自我们的带注释方法的日志语句。此外，当我们从数据库加载用户时，我们可以可靠地期望填充用户的全名。

## 4.注释一个`EntityListener`

我们现在将扩展我们的例子，使用一个单独的`EntityListener`来处理我们的更新回调。如果我们有一些操作想要应用于我们所有的实体，我们可能更喜欢这种方法，而不是将方法放在我们的实体中。

让我们创建我们的`AuditTrailListener`来记录`User`表上的所有活动:

```java
public class AuditTrailListener {
    private static Log log = LogFactory.getLog(AuditTrailListener.class);

    @PrePersist
    @PreUpdate
    @PreRemove
    private void beforeAnyUpdate(User user) {
        if (user.getId() == 0) {
            log.info("[USER AUDIT] About to add a user");
        } else {
            log.info("[USER AUDIT] About to update/delete user: " + user.getId());
        }
    }

    @PostPersist
    @PostUpdate
    @PostRemove
    private void afterAnyUpdate(User user) {
        log.info("[USER AUDIT] add/update/delete complete for user: " + user.getId());
    }

    @PostLoad
    private void afterLoad(User user) {
        log.info("[USER AUDIT] user loaded from database: " + user.getId());
    }
}
```

从示例中可以看出，**我们可以对一个方法**应用多个注释。

现在，我们需要回到我们的`User`实体并将`@EntityListener`注释添加到类中:

```java
@EntityListeners(AuditTrailListener.class)
@Entity
public class User {
    //...
}
```

而且，当我们运行测试时，我们将为每个更新操作获得两组日志消息，并在从数据库加载用户后获得一条日志消息。

## 5.结论

在本文中，我们已经了解了什么是 JPA 实体生命周期回调以及何时调用它们。我们看了注释，并讨论了使用它们的规则。我们还尝试在实体类和`EntityListener`类中使用它们。

GitHub 上的[提供了示例代码。](https://web.archive.org/web/20221126221955/https://github.com/eugenp/tutorials/tree/master/persistence-modules/spring-data-jpa-annotations)