# 使用 Hibernate 排序

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/hibernate-sort>

## 1。概述

本文通过使用 Hibernate 查询语言(HQL)和 Criteria API，说明了如何使用 Hibernate 进行排序。

## 延伸阅读:

## [休眠:保存、持续、更新、合并、保存或更新](/web/20221208143837/https://www.baeldung.com/hibernate-save-persist-update-merge-saveorupdate)

A quick and practical guide to Hibernate write methods: save, persist, update, merge, saveOrUpdate.[Read more](/web/20221208143837/https://www.baeldung.com/hibernate-save-persist-update-merge-saveorupdate) →

## [用 Hibernate 删除对象](/web/20221208143837/https://www.baeldung.com/delete-with-hibernate)

Quick guide to deleting an entity in Hibernate.[Read more](/web/20221208143837/https://www.baeldung.com/delete-with-hibernate) →

## [冬眠拦截器](/web/20221208143837/https://www.baeldung.com/hibernate-interceptor)

A quick and practical guide to creating Hibernate interceptors.[Read more](/web/20221208143837/https://www.baeldung.com/hibernate-interceptor) →

## 2。用 HQL 分类

使用 Hibernate 的 HQL 进行排序就像在 HQL 查询字符串中添加 `Order By`子句一样简单:

```java
String hql = "FROM Foo f ORDER BY f.name";
Query query = sess.createQuery(hql);
```

执行这段代码后，Hibernate 将生成以下 SQL 查询:

```java
Hibernate: select foo0_.ID as ID1_0_, foo0_.NAME as NAME2_0_ from 
    FOO foo0_ order by foo0_.NAME
```

默认的排序方向是升序。这就是为什么订单条件`asc`没有包含在生成的 SQL 查询中。

### 2.1。使用显式排序顺序

要手动指定排序顺序，您需要在 `HQL`查询字符串中包含顺序方向:

```java
String hql = "FROM Foo f ORDER BY f.name ASC";
Query query = sess.createQuery(hql);
```

在本例中，设置 HQL 中的`asc`子句包含在生成的 SQL 查询中:

```java
Hibernate: select foo0_.ID as ID1_0_, foo0_.NAME as NAME2_0_ 
    from FOO foo0_ order by foo0_.NAME ASC
```

### 2.2。按多个属性排序

可以将多个属性以及可选的排序顺序添加到 HQL 查询字符串的`Order By`子句中:

```java
String hql = "FROM Foo f ORDER BY f.name DESC, f.id ASC";
Query query = sess.createQuery(hql);
```

生成的 SQL 查询将相应地改变:

```java
Hibernate: select foo0_.ID as ID1_0_, foo0_.NAME as NAME2_0_ 
    from FOO foo0_ order by foo0_.NAME DESC, foo0_.ID ASC
```

### 2.3。设置空值的排序优先级

默认情况下，当作为排序依据的属性具有`null`值时，由 RDMS 决定优先级。可以通过在 HQL 查询字符串中放置一个 `NULLS FIRST`或`NULLS LAST`子句来覆盖这个默认处理。

这个简单的示例将所有空值放在结果列表的末尾:

```java
String hql = "FROM Foo f ORDER BY f.name NULLS LAST";
Query query = sess.createQuery(hql);
```

让我们看看**生成的 SQL 查询**中的 `is null then 1 else 0`子句:

```java
Hibernate: select foo0_.ID as ID1_1_, foo0_.NAME as NAME2_1_, 
foo0_.BAR_ID as BAR_ID3_1_, foo0_.idx as idx4_1_ from FOO foo0_ 
order by case when foo0_.NAME is null then 1 else 0 end, foo0_.NAME
```

### 2.4。一对多关系排序

让我们分析一个复杂的排序案例:**对一对多关系中的实体进行排序**–`Bar`包含一个`Foo`实体的集合。

我们将通过用**Hibernate`@OrderBy`注释**来注释集合来做到这一点；我们将指定完成订购的字段以及方向:

```java
@OrderBy(clause = "NAME DESC")
Set<Foo> fooList = new HashSet();
```

注意注释中`clause`参数。与类似的`@OrderBy` JPA 注释相比，这是 Hibernate 的`@OrderBy`所独有的。这种方法与其 JPA 等效方法的另一个区别在于， `clause`参数表明排序是基于 `FOO`表的`NAME`列进行的，而不是基于`Foo`的 `name`属性。

现在我们来看看`Bars`和`Foos`的实际排序:

```java
String hql = "FROM Bar b ORDER BY b.id";
Query query = sess.createQuery(hql);
```

**产生的 SQL 语句**显示排序后的`Foo's`放在了 `fooList:`中

```java
Hibernate: select bar0_.ID as ID1_0_, bar0_.NAME as NAME2_0_ from BAR bar0_ 
    order by bar0_.ID Hibernate: select foolist0_.BAR_ID as BAR_ID3_0_0_, 
    foolist0_.ID as ID1_1_0_, foolist0_.ID as ID1_1_1_, foolist0_.NAME as 
    NAME2_1_1_, foolist0_.BAR_ID as BAR_ID3_1_1_, foolist0_.idx as idx4_1_1_ 
    from FOO foolist0_ where foolist0_.BAR_ID=? order by foolist0_.NAME desc
```

需要记住的一点是，不可能像 JPA 那样对列表进行排序。Hibernate 文档声明:

> `“Hibernate currently ignores @OrderBy on @ElementCollection on e.g. List<String>. The order of elements is as returned by the database, undefined.”`

顺便提一下，可以通过使用 Hibernate 的遗留 XML 配置，并用一个`<Bag…>`元素替换`<List..>`元素来解决这个限制。

## 3。根据休眠标准排序

Criteria 对象 API 提供了`Order`类作为管理排序的主要 API。

### 3.1。设置分类顺序

`Order`类有两种方法来设置排序顺序:

*   `**asc**(String attribute)` :按`attribute`对查询进行升序排序。
*   `**desc**(String attribute)` :按`attribute`降序排列查询。

让我们从一个简单的例子开始——按单个`id` 属性排序:

```java
Criteria criteria = sess.createCriteria(Foo.class, "FOO");
criteria.addOrder(Order.asc("id"));
```

请注意，`asc`方法的参数是区分大小写的，并且应该与作为排序依据的属性的`name`相匹配。

Hibernate Criteria 的对象 API 明确设置了排序顺序方向，这反映在代码生成的 SQL 语句中:

```java
Hibernate: select this_.ID as ID1_0_0_, this_.NAME as NAME2_0_0_ 
    from FOO this_ order by this_.ID sac
```

### 3.2。按多个属性排序

按多个属性排序只需要向`Criteria` 实例添加一个`Order`对象，如下例所示:

```java
Criteria criteria = sess.createCriteria(Foo.class, "FOO");
criteria.addOrder(Order.asc("name"));
criteria.addOrder(Order.asc("id"));
```

SQL 中生成的查询是:

```java
Hibernate: select this_.ID as ID1_0_0_, this_.NAME as NAME2_0_0_ from 
    FOO this_ order by this_.NAME asc, this_.ID sac
```

### 3.3。设置空值的排序优先级

默认情况下，当作为排序依据的属性具有`null`值时，由 RDMS 决定优先级。Hibernate Criteria Object API 使得更改默认设置变得很简单，而且**将空值放在升序排序列表的末尾**:

```java
Criteria criteria = sess.createCriteria(Foo.class, "FOO");
criteria.addOrder(Order.asc("name").nulls(NullPrecedence.LAST));
```

下面是底层的 `SQL`查询–带有 **`is null then 1 else 0`** 子句:

```java
Hibernate: select this_.ID as ID1_1_1_, this_.NAME as NAME2_1_1_, 
    this_.BAR_ID as BAR_ID3_1_1_, this_.idx as idx4_1_1_, bar2_.ID as
    ID1_0_0_, bar2_.NAME as NAME2_0_0_ from FOO order by case when 
    this_.NAME is null then 1 else 0 end, this_.NAME asc
```

或者，我们也可以**将空值放在降序列表的开头**:

```java
Criteria criteria = sess.createCriteria(Foo.class, "FOO");
criteria.addOrder(Order.desc("name").nulls(NullPrecedence.FIRST));
```

相应的 SQL 查询如下——带有 **`is null then 0 else 1`** 子句:

```java
Hibernate: select this_.ID as ID1_1_1_, this_.NAME as NAME2_1_1_, 
    this_.BAR_ID as BAR_ID3_1_1_, this_.idx as idx4_1_1_, bar2_.ID as 
    ID1_0_0_, bar2_.NAME as NAME2_0_0_ from FOO order by case when 
    this_.NAME is null then 0 else 1 end, this_.NAME desc
```

注意，**如果作为排序依据的属性是像`int,` 这样的原始类型，那么`PresisitenceException`将抛出**。

例如，如果 `f.anIntVariable`的值为空，那么查询的执行:

```java
String jql = "Select f from Foo as f order by f.anIntVariable desc NULLS FIRST";
Query sortQuery = entityManager.createQuery(jql);
```

将抛出:

```java
javax.persistence.PersistenceException: org.hibernate.PropertyAccessException:
Null value was assigned to a property of primitive type setter of 
com.cc.jpa.example.Foo.anIntVariable
```

## 4。结论

本文探讨了 Hibernate 的排序——使用简单实体和一对多关系中的实体的可用 API。

这个 Hibernate 排序教程的实现可以在 github 项目中找到——这是一个基于 Eclipse 的项目，所以它应该很容易导入和运行。