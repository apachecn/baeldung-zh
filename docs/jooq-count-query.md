# jOOQ 中的计数查询

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/jooq-count-query>

## 1.概观

在本教程中，我们将演示如何使用 **jOOQ 面向对象查询**执行计数查询，也称为 [jOOQ](/web/20221225104252/https://www.baeldung.com/jooq-with-spring) 。jOOQ 是一个流行的 Java 数据库库，它帮助你用 Java 编写类型安全的 SQL 查询。

## 2.jOOQ

jOOQ 是 ORM 的替代品。与大多数其他 ORM 不同， **jOOQ 是以关系模型为中心的，而不是以领域模型为中心的**。[例如，Hibernate](/web/20221225104252/https://www.baeldung.com/learn-jpa-hibernate) 帮助我们编写 Java 代码，然后自动翻译成 SQL。然而，jOOQ 允许我们使用 SQL 在数据库中创建关系对象，然后它生成 Java 代码来映射这些对象。

## 3.Maven 依赖性

我们将需要本教程中的`[jooq](https://web.archive.org/web/20221225104252/https://search.maven.org/classic/#search%7Cga%7C1%7Cg%3A%22org.jooq%22)` 模块:

```java
<dependency> 
    <groupId>org.jooq</groupId> 
    <artifactId>jooq</artifactId> 
    <version>3.14.8</version> 
</dependency>
```

## 4.计数查询

假设我们的数据库中有一个`author`表。`author`表包含一个`id, first_name, and last_name.`

运行计数查询可以通过几种不同的方式来完成。

### 4.1.`fetchCount`****

[`DSL.fetchCount`](https://web.archive.org/web/20221225104252/https://www.jooq.org/javadoc/latest/org.jooq/org/jooq/DSLContext.html#fetchCount(org.jooq.Select)) 有多种方式来统计表中的记录数。

首先，我们来看一下 **`fetchCount​(Table<?> table)`** 统计记录数量的方法:

```java
int count = dsl.fetchCount(DSL.selectFrom(AUTHOR));
Assert.assertEquals(3, count); 
```

接下来，我们用`selectFrom` 方法和`where`子句尝试一下 **`fetchCount​(Table<?> table)`** 方法来统计记录数:

```java
int count = dsl.fetchCount(DSL.selectFrom(AUTHOR)
  .where(AUTHOR.FIRST_NAME.equalIgnoreCase("Bryan")));
Assert.assertEquals(1, count);
```

现在，让我们试试 `**fetchCount​(Table<?> table, Condition condition)**`的方法来统计记录的数量:

```java
int count = dsl.fetchCount(AUTHOR, AUTHOR.FIRST_NAME.equalIgnoreCase("Bryan"));
Assert.assertEquals(1, count); 
```

我们也可以对多种情况使用`**fetchCount​(Table<?> table, Collection<? extends Condition> conditions)**`方法:

```java
Condition firstCond = AUTHOR.FIRST_NAME.equalIgnoreCase("Bryan");
Condition secondCond = AUTHOR.ID.notEqual(1);
List<Condition> conditions = new ArrayList<>();
conditions.add(firstCond);
conditions.add(secondCond);
int count = dsl.fetchCount(AUTHOR, conditions);
Assert.assertEquals(1, count); 
```

在本例中，我们将过滤条件添加到一个列表中，并将其提供给`fetchCount`方法。

`fetchCount` 方法还允许多个条件的变量:

```java
Condition firstCond = AUTHOR.FIRST_NAME.equalIgnoreCase("Bryan");
Condition secondCond = AUTHOR.ID.notEqual(1);
int count = dsl.fetchCount(AUTHOR, firstCond, secondCond);
Assert.assertEquals(1, count); 
```

或者，我们可以使用`[and(Condition condition)](https://web.archive.org/web/20221225104252/https://www.jooq.org/javadoc/latest/org.jooq/org/jooq/Condition.html#and(org.jooq.Condition))`来组合内联条件:

```java
int count = dsl.fetchCount(AUTHOR, AUTHOR.FIRST_NAME.equalIgnoreCase("Bryan").and(AUTHOR.ID.notEqual(1)));
Assert.assertEquals(1, count);
```

### 4.2.`count`

让我们试试 [`count`](https://web.archive.org/web/20221225104252/https://www.jooq.org/javadoc/latest/org.jooq/org/jooq/impl/DSL.html#count()) 方法来获得可用记录的数量:

```java
int count = dsl.select(DSL.count()).from(AUTHOR)
  .fetchOne(0, int.class);
Assert.assertEquals(3, count); 
```

### 4.3.`selectCount`

现在，让我们尝试使用 [`selectCount`](https://web.archive.org/web/20221225104252/https://www.jooq.org/javadoc/latest/org.jooq/org/jooq/DSLContext.html#selectCount()) 方法来获得可用记录的计数:

```java
int count = dsl.selectCount().from(AUTHOR)
  .where(AUTHOR.FIRST_NAME.equalIgnoreCase("Bryan"))
  .fetchOne(0, int.class);
Assert.assertEquals(1, count);
```

### 4.4.简单`select`

我们还可以用一个简单的 [`select`](https://web.archive.org/web/20221225104252/https://www.jooq.org/javadoc/latest/org.jooq/org/jooq/DSLContext.html#select(java.util.Collection)) 方法来得到可用记录的计数:

```java
int count = dsl.select().from(AUTHOR).execute();
Assert.assertEquals(3, count);
```

### 4.5.用`groupBy`计数

让我们尝试使用*选择*和`count`方法来查找按字段分组的记录数:

```java
Result<Record2<String, Integer>> result = dsl.select(AUTHOR.FIRST_NAME, DSL.count())
  .from(AUTHOR).groupBy(AUTHOR.FIRST_NAME).fetch();
Assert.assertEquals(3, result.size());
Assert.assertEquals(result.get(0).get(0), "Bert");
Assert.assertEquals(result.get(0).get(1), 1);
```

## 5.结论

在本文中，我们已经了解了如何在 jOOQ 中执行计数查询。

我们已经看到了使用`selectCount, count, fetchCount,` `select,` 和`count`以及 `groupBy`方法来计算记录的数量。

像往常一样，本教程中使用的所有代码示例都可以在 GitHub 上获得。