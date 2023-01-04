# Hibernate 会话中的对象状态

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/hibernate-session-object-states>

## 1.介绍

Hibernate 是一个管理持久数据的便利框架，但是理解它的内部工作方式有时会很棘手。

在本教程中，我们将学习对象状态以及如何在它们之间移动。我们还将研究分离实体可能遇到的问题以及如何解决这些问题。

## 2.Hibernate 会话

`[Session](https://web.archive.org/web/20220626103912/https://docs.jboss.org/hibernate/orm/3.5/javadocs/org/hibernate/Session.html)`接口是用来与 Hibernate 通信的主要工具。它提供了一个 API，使我们能够创建、读取、更新和删除持久对象。`session`拥有简单的生命周期。我们打开它，执行一些操作，然后关闭它。

当我们在`session`期间对对象进行操作时，它们会附着到那个`session`上。我们所做的更改会在关闭时被检测并保存。关闭后，Hibernate 会断开对象和会话之间的连接。

## 3.对象状态

在 Hibernate 的`Session`上下文中，对象可以处于三种可能的状态之一:瞬态、持久或分离。

### 3.1.短暂的

**一个我们没有附着在任何`session`上的物体处于瞬态。**因为它从未被持久化，所以它在数据库中没有任何表示。因为没有`session`知道，所以不会自动保存。

让我们用构造函数创建一个用户对象，并确认它不是由会话管理的:

```java
Session session = openSession();
UserEntity userEntity = new UserEntity("John");
assertThat(session.contains(userEntity)).isFalse();
```

### 3.2.坚持的

**我们与`session`关联的对象处于持久状态。**我们要么保存它，要么从持久上下文中读取它，所以它代表数据库中的某一行。

让我们创建一个对象，然后使用`persist`方法使其持久化:

```java
Session session = openSession();
UserEntity userEntity = new UserEntity("John");
session.persist(userEntity);
assertThat(session.contains(userEntity)).isTrue();
```

或者，我们可以使用`save`方法。不同之处在于，`persist`方法将只保存一个对象，如果需要的话，`save`方法将另外生成它的标识符。

### 3.3.分离的

当我们关闭`session`时，里面的所有物体都分离了。**尽管它们仍然代表数据库中的行，但它们不再由任何`session` :** 管理

```java
session.persist(userEntity);
session.close();
assertThat(session.isOpen()).isFalse();
assertThatThrownBy(() -> session.contains(userEntity));
```

接下来，我们将学习如何保存瞬态和分离的实体。

## 4.保存和重新附加实体

### 4.1.保存临时实体

让我们创建一个新实体，并将其保存到数据库中。当我们第一次构造对象时，它会处于瞬态。

为了`persist`我们的新实体，我们将使用`persist`方法:

```java
UserEntity userEntity = new UserEntity("John");
session.persist(userEntity);
```

现在，我们将创建与第一个对象具有相同标识符的另一个对象。第二个对象是暂时的，因为它还没有被任何`session`管理，但是我们不能使用`persist`方法使它持久化。它已经在数据库中表示出来了，所以在持久层的环境中它并不是什么新东西。

相反，**我们将使用`merge`方法来更新数据库并使对象持久化**:

```java
UserEntity onceAgainJohn = new UserEntity("John");
session.merge(onceAgainJohn);
```

### 4.2.保存分离的实体

**如果我们关闭前面的`session`，我们的对象将处于分离状态。**与前面的例子类似，它们在数据库中有表示，但是它们当前不受任何`session`管理。我们可以使用`merge`方法让它们再次持久化:

```java
UserEntity userEntity = new UserEntity("John");
session.persist(userEntity);
session.close();
session.merge(userEntity);
```

## 5.嵌套实体

当我们考虑嵌套实体时，事情变得更加复杂。假设我们的用户实体也将存储关于他的经理的信息:

```java
public class UserEntity {
    @Id
    private String name;

    @ManyToOne
    private UserEntity manager;
}
```

当我们保存这个实体时，我们不仅需要考虑实体本身的状态，还需要考虑嵌套实体的状态。让我们创建一个持久用户实体，然后设置它的管理器:

```java
UserEntity userEntity = new UserEntity("John");
session.persist(userEntity);
UserEntity manager = new UserEntity("Adam");
userEntity.setManager(manager);
```

如果我们现在尝试更新它，我们会得到一个异常:

```java
assertThatThrownBy(() -> {
            session.saveOrUpdate(userEntity);
            transaction.commit();
});
```

```java
java.lang.IllegalStateException: org.hibernate.TransientPropertyValueException: object references an unsaved transient instance - save the transient instance before flushing : com.baeldung.states.UserEntity.manager -> com.baeldung.states.UserEntity 
```

这是因为 Hibernate 不知道如何处理临时嵌套实体。

### 5.1.持久化嵌套实体

解决这个问题的一个方法是显式持久化嵌套实体:

```java
UserEntity manager = new UserEntity("Adam");
session.persist(manager);
userEntity.setManager(manager);
```

然后，在提交事务后，我们将能够检索正确保存的实体:

```java
transaction.commit();
session.close();

Session otherSession = openSession();
UserEntity savedUser = otherSession.get(UserEntity.class, "John");
assertThat(savedUser.getManager().getName()).isEqualTo("Adam");
```

### 5.2.级联操作

如果我们在实体类中正确配置了关系的`cascade`属性，瞬态嵌套实体可以自动持久化:

```java
@ManyToOne(cascade = CascadeType.PERSIST)
private UserEntity manager;
```

**现在，当我们持久化对象时，该操作将级联到所有嵌套的实体:**

```java
UserEntityWithCascade userEntity = new UserEntityWithCascade("John");
session.persist(userEntity);
UserEntityWithCascade manager = new UserEntityWithCascade("Adam");

userEntity.setManager(manager); // add transient manager to persistent user
session.saveOrUpdate(userEntity);
transaction.commit();
session.close();

Session otherSession = openSession();
UserEntityWithCascade savedUser = otherSession.get(UserEntityWithCascade.class, "John");
assertThat(savedUser.getManager().getName()).isEqualTo("Adam");
```

## 6.摘要

在本教程中，我们仔细研究了 Hibernate `Session`在对象状态方面的工作方式。然后，我们研究了它可能产生的一些问题以及如何解决这些问题。

和往常一样，源代码可以在 GitHub 上的[处获得。](https://web.archive.org/web/20220626103912/https://github.com/eugenp/tutorials/tree/master/persistence-modules/spring-boot-persistence-2)