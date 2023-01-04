# @在 Hibernate 中不可变

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/hibernate-immutable>

## 1。概述

在本文中，我们将讨论如何在 Hibernate 中使实体、集合或属性[成为不可变的](https://web.archive.org/web/20220524113602/https://docs.jboss.org/hibernate/orm/5.2/javadocs/org/hibernate/annotations/Immutable.html)。

默认情况下，字段是可变的，这意味着我们能够对它们执行改变其状态的操作。

## 2。肚子

为了启动并运行我们的项目，我们首先需要将必要的依赖项添加到我们的`pom.xml`中。当我们使用 Hibernate 时，我们将添加相应的[依赖关系](https://web.archive.org/web/20220524113602/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22org.hibernate%22%20AND%20a%3A%22hibernate-core%22):

```java
<dependency>
    <groupId>org.hibernate</groupId>
    <artifactId>hibernate-core</artifactId>
    <version>5.6.7.Final</version>
</dependency>
```

而且，因为我们使用的是 [HSQLDB](https://web.archive.org/web/20220524113602/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22org.hsqldb%22%20AND%20a%3A%22hsqldb%22) ，我们还需要:

```java
<dependency>
    <groupId>org.hsqldb</groupId>
    <artifactId>hsqldb</artifactId>
    <version>2.3.4</version>
</dependency>
```

## 3。实体上的注释

首先，让我们定义一个简单的实体类:

```java
@Entity
@Immutable
@Table(name = "events_generated")
public class EventGeneratedId {

    @Id
    @Column(name = "event_generated_id")
    @GeneratedValue(generator = "increment")
    @GenericGenerator(name = "increment", strategy = "increment")
    private Long id;

    @Column(name = "name")
    private String name;
    @Column(name = "description")
    private String description;

    // standard setters and getters
}
```

正如你已经注意到的，我们已经向我们的实体添加了`@Immutable`注释，所以如果我们试图保存一个`Event`:

```java
@Test
public void addEvent() {
    Event event = new Event();
    event.setId(2L);
    event.setTitle("Public Event");
    session.save(event);
    session.getTransaction().commit();
    session.close();
}
```

那么我们应该得到输出:

```java
Hibernate: insert into events (title, event_id) values (?, ?)
```

即使我们删除了注释，输出也应该是相同的，这意味着当我们试图添加一个实体而不考虑注释时，没有任何效果。

同样重要的是要注意，在我们的`EventGeneratedId` 实体中，我们添加了`GeneratedValue`注释，但是这只会在我们创建实体时有所不同。这是因为它指定了 id 的生成策略——由于`Immutable`注释，任何其他操作都不会影响`Id`字段。

### 3.1。更新实体

现在，保存实体没有问题，让我们尝试更新它:

```java
@Test
public void updateEvent() {
    Event event = (Event) session.createQuery(
      "FROM Event WHERE title='My Event'").list().get(0);
    event.setTitle("Public Event");
    session.saveOrUpdate(event);
    session.getTransaction().commit();
}
```

Hibernate 将简单地忽略`update` 操作，而不会抛出异常。然而，如果我们移除`@Immutable`注释，我们会得到不同的结果:

```java
Hibernate: select ... from events where title='My Event'
Hibernate: update events set title=? where event_id=?
```

这告诉我们，我们的对象现在是可变的(`mutable` 是缺省值，如果我们不包括注释的话)，并且将允许更新完成它的工作。

### 3.2。删除实体

在删除实体时:

```java
@Test
public void deleteEvent() {
    Event event = (Event) session.createQuery(
      "FROM Event WHERE title='My Event'").list().get(0);
    session.delete(event);
    session.getTransaction().commit();
}
```

我们将能够执行删除，不管它是否可变:

```java
Hibernate: select ... from events where title='My Event'
Hibernate: delete from events where event_id=?
```

## 4。关于集合的注释

到目前为止，我们已经看到了注释对实体的作用，但是正如我们在开始时提到的，它也可以应用于集合。

首先，让我们向我们的`Event`类添加一个集合:

```java
@Immutable
public Set<String> getGuestList() {
    return guestList;
}
```

和以前一样，我们已经预先添加了注释，所以如果我们继续尝试向我们的集合添加一个元素:

```java
org.hibernate.HibernateException: 
  changed an immutable collection instance: [com.baeldung.entities.Event.guestList#1]
```

这一次我们得到了一个异常，因为对于集合，我们不允许添加或删除它们。

### 4.1。删除收藏

另一种情况是当我们试图删除并且已经设置了`@Cascade` 注释时，`Collection`不可变会抛出异常。

因此，每当`@Immutable`出现并且我们试图删除:

```java
@Test
public void deleteCascade() {
    Event event = (Event) session.createQuery(
      "FROM Event WHERE title='Public Event'").list().get(0);
    String guest = event.getGuestList().iterator().next();
    event.getGuestList().remove(guest);
    session.saveOrUpdate(event);
    session.getTransaction().commit();
}
```

输出:

```java
org.hibernate.HibernateException: 
  changed an immutable collection instance:
  [com.baeldung.entities.Event.guestList#1]
```

## 5。XML 注释

最后，还可以使用 XML 通过`mutable=false`属性来完成配置:

```java
<hibernate-mapping>
    <class name="com.baeldung.entities.Event" mutable="false">
        <id name="id" column="event_id">
            <generator class="increment"/>
        </id>
        <property name="title"/>
    </class>
</hibernate-mapping>
```

然而，由于我们基本上是使用注释方法实现示例的，所以我们不会使用 XML 来讨论细节。

## 6。结论

在这篇简短的文章中，我们探索了 Hibernate 中有用的`@Immutable`注释，以及它如何帮助我们在数据上定义更好的语义和约束。

和往常一样，所有这些例子和片段的实现都可以在 GitHub 项目中找到。这是一个基于 Maven 的项目，因此应该很容易导入和运行。