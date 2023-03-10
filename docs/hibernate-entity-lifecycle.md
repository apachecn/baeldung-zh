# Hibernate 实体生命周期

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/hibernate-entity-lifecycle>

## 1。概述

每个 Hibernate 实体在框架内自然都有一个生命周期——它或者处于暂时的、受管理的、分离的或者被删除的状态。

要正确使用 Hibernate，在概念和技术层面上理解这些状态是必不可少的。

要了解处理实体的各种 Hibernate 方法，请看一下我们之前的教程之一[。](/web/20221129021404/https://www.baeldung.com/hibernate-save-persist-update-merge-saveorupdate)

## 2。助手方法

在本教程中，我们将始终使用几个助手方法:

*   `HibernateLifecycleUtil.getManagedEntities(session) –` 我们将使用它从`Session's`内部存储中获取所有托管实体

*   `DirtyDataInspector.getDirtyEntities() –` 我们将使用这个方法获得所有被标记为“脏”的实体的列表

*   一种对嵌入式数据库进行查询的便捷方法

为了更好的可读性，上面所有的助手方法都是静态导入的。您可以在本文末尾链接的 GitHub 项目中找到它们的实现。

## 3。这都是关于持久性上下文

在进入实体生命周期这个话题之前，首先，我们需要了解`the persistence context`。

**简单地说，`persistence context`位于客户端代码和数据存储库**之间。它是一个暂存区，在这里持久数据被转换成实体，准备好被客户端代码读取和修改。

从理论上讲，`persistence context `是[工作单元](https://web.archive.org/web/20221129021404/https://martinfowler.com/eaaCatalog/unitOfWork.html)模式的一个实现。它跟踪所有加载的数据，跟踪该数据的更改，并负责在业务事务结束时最终将任何更改同步回数据库。

JPA `EntityManager`和 Hibernate 的`Session `是`persistence context `概念的实现。在本文中，我们将使用 Hibernate `Session`来表示`persistence context.`

Hibernate 实体生命周期状态解释了实体是如何与`persistence context`相关联的，我们接下来会看到。

## 4。受管实体

**托管实体是数据库表行**的表示(尽管该行不一定存在于数据库中)。

这是由当前运行的`Session`管理的，而**对它所做的每一个更改都会被自动跟踪并传播到数据库**。

`Session `要么从数据库加载实体，要么重新附加一个分离的实体。我们将在第 5 节讨论分离的实体。

让我们观察一些代码来得到澄清。

我们的示例应用程序定义了一个实体，即`FootballPlayer`类。在启动时，我们将用一些示例数据初始化数据存储:

```java
+-------------------+-------+
| Name              |  ID   |
+-------------------+-------+
| Cristiano Ronaldo | 1     |
| Lionel Messi      | 2     |
| Gianluigi Buffon  | 3     |
+-------------------+-------+
```

假设我们想改变布冯的名字，我们想输入他的全名`Gianluigi Buffon `而不是吉吉·布冯。

首先，我们需要通过获得一个`Session:`来开始我们的工作单元

```java
Session session = sessionFactory.openSession();
```

在服务器环境中，我们可以通过上下文感知代理向代码中注入一个`Session`。原则是一样的:我们需要一个`Session `来封装我们工作单元的业务事务。

接下来，我们将指示我们的`Session`从持久存储中加载数据:

```java
assertThat(getManagedEntities(session)).isEmpty();

List<FootballPlayer> players = s.createQuery("from FootballPlayer").getResultList();

assertThat(getManagedEntities(session)).size().isEqualTo(3); 
```

当我们第一次获得一个`Session`时，它的持久上下文存储是空的，如我们的第一个`assert`语句所示。

接下来，我们将执行一个查询，从数据库中检索数据，创建数据的实体表示，并最终返回实体供我们使用。

在内部，`Session `跟踪它加载到持久上下文存储中的所有实体。在我们的例子中，`Session's `内部存储将在查询后包含 3 个实体。

现在让我们改变琪琪的名字:

```java
Transaction transaction = session.getTransaction();
transaction.begin();

FootballPlayer gigiBuffon = players.stream()
  .filter(p -> p.getId() == 3)
  .findFirst()
  .get();

gigiBuffon.setName("Gianluigi Buffon");
transaction.commit();

assertThat(getDirtyEntities()).size().isEqualTo(1);
assertThat(getDirtyEntities().get(0).getName()).isEqualTo("Gianluigi Buffon");
```

### 4.1。它是如何工作的？

在调用事务`commit()`或`flush()`时，`Session`将从其跟踪列表中找到任何`dirty`实体，并将状态同步到数据库。

**注意，我们不需要调用任何方法来通知`Session`我们更改了实体中的某些内容——因为它是一个托管实体，所以所有的更改都会自动传播到数据库。**

受管实体始终是一个持久实体，它必须有一个数据库标识符，即使数据库行表示尚未创建，即插入语句正在等待工作单元结束。

请参阅下面关于瞬态实体的章节。

## 5。分离的实体

**A `detached entity `只是一个普通的实体 POJO** ，它的标识值对应于一个数据库行。与受管实体的区别在于，它不再被任何`persistence context` 跟踪**。**

当用来加载一个实体的`Session`被关闭时，或者当我们调用`Session.evict(entity)`或`Session.clear()`时，这个实体可以被分离。

让我们在代码中看到它:

```java
FootballPlayer cr7 = session.get(FootballPlayer.class, 1L);

assertThat(getManagedEntities(session)).size().isEqualTo(1);
assertThat(getManagedEntities(session).get(0).getId()).isEqualTo(cr7.getId());

session.evict(cr7);

assertThat(getManagedEntities(session)).size().isEqualTo(0);
```

我们的持久性上下文不会跟踪分离实体中的变化:

```java
cr7.setName("CR7");
transaction.commit();

assertThat(getDirtyEntities()).isEmpty();
```

`Session.merge(entity)/Session.update(entity) can `(重新)附加一个会话`:`

```java
FootballPlayer messi = session.get(FootballPlayer.class, 2L);

session.evict(messi);
messi.setName("Leo Messi");
transaction.commit();

assertThat(getDirtyEntities()).isEmpty();

transaction = startTransaction(session);
session.update(messi);
transaction.commit();

assertThat(getDirtyEntities()).size().isEqualTo(1);
assertThat(getDirtyEntities().get(0).getName()).isEqualTo("Leo Messi");
```

关于`Session.merge()`和`Session.update()`的参考，请参见此处的。

### 5.1。身份字段是最重要的

我们来看看下面的逻辑:

```java
FootballPlayer gigi = new FootballPlayer();
gigi.setId(3);
gigi.setName("Gigi the Legend");
session.update(gigi);
```

在上面的例子中，我们通过构造函数以通常的方式实例化了一个实体。我们已经用值填充了字段，并将标识设置为 3，这对应于属于 Gigi Buffon 的持久数据的标识。调用`update()`的效果与我们从另一个`persistence context`加载实体的效果完全相同。

事实上，*会话*并不区分重新附加的实体来自何处。

在 web 应用程序中，从 HTML 表单值构造分离的实体是很常见的情况。

就`Session`而言，一个分离的实体只是一个普通的实体，它的标识值对应于持久数据。

请注意，上面的例子只是一个演示目的。我们需要知道我们到底在做什么。否则，如果我们只在想要更新的字段上设置值，而不改变其他字段，那么我们可能会在整个实体中得到 null 值(因此，实际上是 null)。

## 6。瞬态实体

**瞬态实体只是一个`entity`对象，它在持久存储**中没有表示，并且不受任何`Session`管理。

瞬态实体的一个典型例子是通过构造函数实例化一个新的实体。

要制作一个瞬态实体`persistent`，我们需要调用`Session.save(entity) `或`Session.saveOrUpdate(entity):`

```java
FootballPlayer neymar = new FootballPlayer();
neymar.setName("Neymar");
session.save(neymar);

assertThat(getManagedEntities(session)).size().isEqualTo(1);
assertThat(neymar.getId()).isNotNull();

int count = queryCount("select count(*) from Football_Player where name='Neymar'");

assertThat(count).isEqualTo(0);

transaction.commit();
count = queryCount("select count(*) from Football_Player where name='Neymar'");

assertThat(count).isEqualTo(1);
```

我们一执行`Session.save(entity)`，实体就被赋予一个身份值，并由`Session`管理。但是，它可能还不在数据库中，因为插入操作可能会延迟到工作单元结束时。

## 7。删除的实体

**如果`Session.delete(entity)`** 已被调用，且`Session `已将实体标记为删除，则该实体处于删除状态。删除命令本身可以在工作单元结束时发出。

让我们看看下面的代码:

```java
session.delete(neymar);

assertThat(getManagedEntities(session).get(0).getStatus()).isEqualTo(Status.DELETED);
```

但是，请注意，该实体会保留在持久上下文存储中，直到工作单元结束。

## 8。结论

`persistence context`的概念是理解 Hibernate 实体生命周期的核心。我们已经通过查看演示每种状态的代码示例阐明了生命周期。

像往常一样，本文中使用的代码可以在 GitHub 上找到[。](https://web.archive.org/web/20221129021404/https://github.com/eugenp/tutorials/tree/master/persistence-modules/hibernate5)