# 使用 Hibernate 删除对象

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/delete-with-hibernate>

## 1。概述

作为一个全功能的 ORM 框架，Hibernate 负责持久化对象(实体)的生命周期管理，包括`read`、`save`、`update`、`delete`等 CRUD 操作。

在本文中，我们将探索使用 Hibernate 从数据库中删除对象的各种方式，并解释可能出现的常见问题和陷阱。

我们使用 JPA，对于那些在 JPA 中没有标准化的特性，我们只退一步使用 Hibernate native API。

## 2。删除对象的不同方式

在下列情况下，可能会删除对象:

*   通过使用`EntityManager.remove`
*   当从其他实体实例级联删除时
*   当应用`orphanRemoval`时
*   通过执行一个`delete` JPQL 语句
*   通过执行本地查询
*   通过应用软删除技术(通过`@Where`子句中的条件过滤软删除的实体)

在本文的剩余部分，我们将详细讨论这些要点。

## 3。使用实体管理器删除

用`EntityManager`删除是删除实体实例最直接的方法:

```java
Foo foo = new Foo("foo");
entityManager.persist(foo);
flushAndClear();

foo = entityManager.find(Foo.class, foo.getId());
assertThat(foo, notNullValue());
entityManager.remove(foo);
flushAndClear();

assertThat(entityManager.find(Foo.class, foo.getId()), nullValue()); 
```

在本文的示例中，我们使用一个 helper 方法在需要时刷新和清除持久性上下文:

```java
void flushAndClear() {
    entityManager.flush();
    entityManager.clear();
}
```

在调用了`EntityManager.remove`方法之后，所提供的实例转换到`removed`状态，并且在下一次刷新时从数据库中进行相关的删除。

注意，如果对应用了`PERSIST`操作，那么**被删除的实例将被重新持久化。一个常见的错误是忽略了一个`PERSIST`操作已经被应用到一个被移除的实例(通常，因为它是在刷新时从另一个实例级联而来的)，因为 [JPA 规范](https://web.archive.org/web/20220628070435/https://download.oracle.com/otndocs/jcp/persistence-2_1-fr-eval-spec/index.html)的章节`3.2.2`要求在这种情况下要再次持久化这样的实例。**

我们通过定义从`Foo`到`Bar`的`@ManyToOne`关联来说明这一点:

```java
@Entity
public class Foo {
    @ManyToOne(fetch = FetchType.LAZY, cascade = CascadeType.ALL)
    private Bar bar;

    // other mappings, getters and setters
}
```

当我们删除一个被`Foo`实例引用的`Bar`实例时，这个`Bar`实例不会从数据库中删除:

```java
Bar bar = new Bar("bar");
Foo foo = new Foo("foo");
foo.setBar(bar);
entityManager.persist(foo);
flushAndClear();

foo = entityManager.find(Foo.class, foo.getId());
bar = entityManager.find(Bar.class, bar.getId());
entityManager.remove(bar);
flushAndClear();

bar = entityManager.find(Bar.class, bar.getId());
assertThat(bar, notNullValue());

foo = entityManager.find(Foo.class, foo.getId());
foo.setBar(null);
entityManager.remove(bar);
flushAndClear();

assertThat(entityManager.find(Bar.class, bar.getId()), nullValue());
```

如果被移除的`Bar`被`Foo`引用，则`PERSIST`操作从`Foo`级联到`Bar`，因为关联被标记为`cascade = CascadeType.ALL`并且删除是未计划的。为了验证这一点，我们可以为`org.hibernate`包启用跟踪日志级别，并搜索像`un-scheduling entity deletion`这样的条目。

## 4。级联删除

删除父实体时，删除可以级联到子实体:

```java
Bar bar = new Bar("bar");
Foo foo = new Foo("foo");
foo.setBar(bar);
entityManager.persist(foo);
flushAndClear();

foo = entityManager.find(Foo.class, foo.getId());
entityManager.remove(foo);
flushAndClear();

assertThat(entityManager.find(Foo.class, foo.getId()), nullValue());
assertThat(entityManager.find(Bar.class, bar.getId()), nullValue());
```

此处`bar`被移除，因为移除是从`foo`级联的，因为关联被声明为级联从`Foo`到`Bar`的所有生命周期操作。

注意**在`@ManyToMany`关联**中级联`REMOVE`操作几乎总是一个错误，因为这将触发删除可能与其他父实例关联的子实例。这也适用于`CascadeType.ALL`，因为这意味着所有的操作都是级联的，包括`REMOVE`操作。

## 5。孤儿的去除

`orphanRemoval`指令声明，当关联的实体实例与父实体分离时，或者等效地，当父实体被移除时，关联的实体实例将被移除。

我们通过定义从`Bar`到`Baz:`的关联来说明这一点

```java
@Entity
public class Bar {
    @OneToMany(cascade = CascadeType.ALL, orphanRemoval = true)
    private List<Baz> bazList = new ArrayList<>();

    // other mappings, getters and setters
}
```

然后，当从父`Bar`实例的列表中删除一个`Baz`实例时，该实例被自动删除:

```java
Bar bar = new Bar("bar");
Baz baz = new Baz("baz");
bar.getBazList().add(baz);
entityManager.persist(bar);
flushAndClear();

bar = entityManager.find(Bar.class, bar.getId());
baz = bar.getBazList().get(0);
bar.getBazList().remove(baz);
flushAndClear();

assertThat(entityManager.find(Baz.class, baz.getId()), nullValue());
```

**`orphanRemoval`操作的语义完全类似于直接应用于受影响的子实例**的`REMOVE`操作，这意味着`REMOVE`操作进一步级联到嵌套的子实例。因此，您必须确保没有其他实例引用被移除的实例(否则它们会被重新持久化)。

## 6。使用 JPQL 语句删除

Hibernate 支持 DML 风格的删除操作:

```java
Foo foo = new Foo("foo");
entityManager.persist(foo);
flushAndClear();

entityManager.createQuery("delete from Foo where id = :id")
  .setParameter("id", foo.getId())
  .executeUpdate();

assertThat(entityManager.find(Foo.class, foo.getId()), nullValue());
```

值得注意的是， **DML 风格的 JPQL 语句既不影响已经加载到持久性上下文**中的实体实例的状态，也不影响其生命周期，因此建议在加载受影响的实体之前执行这些语句。

## 7 .**。使用本地查询删除**

有时，我们需要依靠原生查询来实现 Hibernate 不支持的或者特定于数据库供应商的功能。我们也可以用本地查询删除数据库中的数据:

```java
Foo foo = new Foo("foo");
entityManager.persist(foo);
flushAndClear();

entityManager.createNativeQuery("delete from FOO where ID = :id")
  .setParameter("id", foo.getId())
  .executeUpdate();

assertThat(entityManager.find(Foo.class, foo.getId()), nullValue());
```

对于 JPA DML 风格的语句，同样的建议也适用于本地查询，即**本地查询既不影响实体实例的状态也不影响其生命周期，实体实例在执行查询**之前被加载到持久性上下文中。

## 8。软删除

出于审计和保存历史的目的，通常不希望从数据库中删除数据。在这种情况下，我们可以应用一种叫做软删除的技术。基本上，我们只是将一行标记为已删除，并在检索数据时将其过滤掉。

为了避免读取可软删除实体的所有查询中的`where`子句中的大量冗余条件，Hibernate 提供了`@Where`注释，该注释可以放在实体上，并且包含一个 SQL 片段，该片段会自动添加到为该实体生成的 SQL 查询中。

为了演示这一点，我们向`Foo`实体添加了`@Where`注释和一个名为`DELETED`的列:

```java
@Entity
@Where(clause = "DELETED = 0")
public class Foo {
    // other mappings

    @Column(name = "DELETED")
    private Integer deleted = 0;

    // getters and setters

    public void setDeleted() {
        this.deleted = 1;
    }
}
```

以下测试确认一切正常:

```java
Foo foo = new Foo("foo");
entityManager.persist(foo);
flushAndClear();

foo = entityManager.find(Foo.class, foo.getId());
foo.setDeleted();
flushAndClear();

assertThat(entityManager.find(Foo.class, foo.getId()), nullValue());
```

## 9。结论

在本文中，我们研究了用 Hibernate 删除数据的不同方法。我们解释了基本概念和一些最佳实践。我们还演示了如何使用 Hibernate 轻松实现软删除。

Github 上的[提供了使用 Hibernate 删除对象教程的实现。这是一个基于 Maven 的项目，所以应该很容易导入和运行。](https://web.archive.org/web/20220628070435/https://github.com/eugenp/tutorials/tree/master/persistence-modules/spring-hibernate-5)