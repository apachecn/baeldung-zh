# Hibernate @WhereJoinTable 注释

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/hibernate-wherejointable>

## 1.概观

使用像 Hibernate 这样的对象关系映射工具，可以很容易地将数据读入对象，但是对于复杂的数据模型，却很难形成查询。

多对多的关系总是具有挑战性，但是当我们希望**基于关系本身的一些属性来获取相关的实体时，这可能更具挑战性。**

在本教程中，我们将看看如何使用 Hibernate 的`@WhereJoinTable`注释来解决这个问题。

## 2.基本`@ManyToMany`关系

让我们从一个简单的 [`@ManyToMany`关系](/web/20221128035425/https://www.baeldung.com/jpa-many-to-many)开始。我们需要领域模型实体、关系实体和一些样本测试数据。

### 2.1.领域模型

假设我们有两个简单的实体，`User`和`Group`，它们被关联为`@ManyToMany:`

```java
@Entity(name = "users")
public class User {

    @Id
    @GeneratedValue
    private Long id;
    private String name;

    @ManyToMany
    private List<Group> groups = new ArrayList<>();

    // standard getters and setters

} 
```

```java
@Entity
public class Group {

    @Id
    @GeneratedValue
    private Long id;
    private String name;

    @ManyToMany(mappedBy = "groups")
    private List<User> users = new ArrayList<>();

    // standard getters and setters

} 
```

正如我们所看到的，我们的`User`实体可以是多个`Group`实体的成员。类似地，一个`Group`实体可以包含不止一个`User`实体。

### 2.2.关系实体

**对于`@ManyToMany`关联，我们需要一个单独的数据库表，称为关系表。**关系表需要包含至少两列:相关的`User`和`Group`实体的主键。

只有两个主键列[，我们的 Hibernate 映射可以表示这个关系表](/web/20221128035425/https://www.baeldung.com/hibernate-many-to-many)。

但是，如果我们需要在关系表中放入额外的数据，**我们还应该为多对多关系本身定义一个关系实体。**

让我们创建`UserGroupRelation`类来做这件事:

```java
@Entity(name = "r_user_group")
public class UserGroupRelation implements Serializable {

    @Id
    @Column(name = "user_id", insertable = false, updatable = false)
    private Long userId;

    @Id
    @Column(name = "group_id", insertable = false, updatable = false)
    private Long groupId;

} 
```

这里我们将实体命名为`r_user_group`,这样我们可以在以后引用它。

对于我们的额外数据，假设我们想要为每个`Group`存储每个`User`的角色。所以，我们将创建`UserGroupRole`枚举:

```java
public enum UserGroupRole {
    MEMBER, MODERATOR
} 
```

接下来，我们将向`UserGroupRelation:`添加一个`role`属性

```java
@Enumerated(EnumType.STRING)
private UserGroupRole role; 
```

最后，为了正确配置它，我们需要在`User`的`groups`集合上添加`@JoinTable`注释。这里我们将使用`UserGroupRelation:`的实体名`r_user_group,` 来指定连接表名

```java
@ManyToMany
@JoinTable(
    name = "r_user_group",
    joinColumns = @JoinColumn(name = "user_id"),
    inverseJoinColumns = @JoinColumn(name = "group_id")
)
private List<Group> groups = new ArrayList<>(); 
```

### 2.3.抽样资料

对于我们的集成测试，让我们定义一些样本数据:

```java
public void setUp() {
    session = sessionFactory.openSession();
    session.beginTransaction();

    user1 = new User("user1");
    user2 = new User("user2");
    user3 = new User("user3");

    group1 = new Group("group1");
    group2 = new Group("group2");

    session.save(group1);
    session.save(group2);

    session.save(user1);
    session.save(user2);
    session.save(user3);

    saveRelation(user1, group1, UserGroupRole.MODERATOR);
    saveRelation(user2, group1, UserGroupRole.MODERATOR);
    saveRelation(user3, group1, UserGroupRole.MEMBER);

    saveRelation(user1, group2, UserGroupRole.MEMBER);
    saveRelation(user2, group2, UserGroupRole.MODERATOR);
}

private void saveRelation(User user, Group group, UserGroupRole role) {

    UserGroupRelation relation = new UserGroupRelation(user.getId(), group.getId(), role);

    session.save(relation);
    session.flush();
    session.refresh(user);
    session.refresh(group);
}
```

我们可以看到，`user1`和`user2`分两组。另外，我们应该注意到，`user1`在`group1`上是`MODERATOR`，同时它在`group2`上有一个`MEMBER`角色。

## 3.获取`@ManyToMany`关系

现在我们已经正确地配置了我们的实体，让我们获取`User`实体的组。

### 3.1.简单获取

为了获取组，我们可以简单地调用活动 Hibernate 会话中的`User`的`getGroups()`方法:

```java
List<Group> groups = user1.getGroups(); 
```

我们对`groups`的输出将是:

```java
[Group [name=group1], Group [name=group2]] 
```

但是，我们如何获得组角色仅为`MODERATOR?`的用户的组呢

### 3.2.关系实体上的自定义筛选器

我们可以**使用`@WhereJoinTable`注释直接只获取过滤后的组。**

让我们定义一个新的属性为`moderatorGroups`，并在上面加上`@WhereJoinTable`注释。当我们通过该属性访问相关实体时，它将只包含我们的用户是`MODERATOR.`的组

我们需要添加一个 SQL where 子句来根据`MODERATOR` 角色过滤组:

```java
@WhereJoinTable(clause = "role='MODERATOR'")
@ManyToMany
@JoinTable(
    name = "r_user_group",
    joinColumns = @JoinColumn(name = "user_id"),
    inverseJoinColumns = @JoinColumn(name = "group_id")
)
private List<Group> moderatorGroups = new ArrayList<>(); 
```

因此，我们可以很容易地获得应用了指定 SQL where 子句的组:

```java
List<Group> groups = user1.getModeratorGroups(); 
```

我们的输出将是用户仅拥有`MODERATOR:`角色的组

```java
[Group [name=group1]] 
```

## 4.结论

在本教程中，我们学习了如何使用 Hibernate 的`@WhereJoinTable`注释基于关系表的属性**过滤`@ManyToMany`集合。**

和往常一样，本教程中给出的所有代码示例都可以从 GitHub 上的[处获得。](https://web.archive.org/web/20221128035425/https://github.com/eugenp/tutorials/tree/master/persistence-modules/hibernate-annotations)