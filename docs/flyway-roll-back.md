# 使用 Flyway 回滚迁移

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/flyway-roll-back>

## 1.介绍

在这个简短的教程中，我们将探索几种使用 [Flyway 回滚迁移的方法。](/web/20220524121925/https://www.baeldung.com/database-migrations-with-flyway)

## 2.通过迁移模拟回滚

在本节中，我们将使用标准迁移文件回滚数据库。

在我们的例子中，我们将使用 Flyway 的命令行版本。然而，核心原则同样适用于其他格式，如核心 API、Maven 插件等。

### 2.1.创建迁移

首先，让我们向数据库添加一个新的`book`表。为此，我们将创建一个名为`V1_0__create_book_table.sql`的迁移文件:

```java
create table book (
  id numeric,
  title varchar(128),
  author varchar(256),
  constraint pk_book primary key (id)
);
```

其次，让我们应用迁移:

```java
./flyway migrate
```

### 2.2.模拟回滚

然后，在某个时候，假设我们需要逆转上一次迁移。

为了将数据库恢复到创建`book`表之前，让我们创建名为`V2_0__drop_table_book.sql`的迁移:

```java
drop table book;
```

接下来，让我们应用迁移:

```java
./flyway migrate
```

最后，我们可以使用以下命令检查所有迁移的历史记录:

```java
./flyway info
```

这为我们提供了以下输出:

```java
+-----------+---------+-------------------+------+---------------------+---------+
| Category  | Version | Description       | Type | Installed On        | State   |
+-----------+---------+-------------------+------+---------------------+---------+
| Versioned | 1.0     | create book table | SQL  | 2020-08-29 16:07:43 | Success |
| Versioned | 2.0     | drop table book   | SQL  | 2020-08-29 16:08:15 | Success |
+-----------+---------+-------------------+------+---------------------+---------+
```

请注意，我们的第二次迁移运行成功。

就 Flyway 而言，第二个迁移文件只是另一个标准迁移。数据库到先前版本的实际恢复完全是通过 SQL 完成的。例如，在我们的例子中，删除表的 SQL 与创建表的第一次迁移相反。

使用这种方法，**审计跟踪不会向我们显示第二次迁移与第一次**相关，因为它们有不同的版本号。为了获得这样的审计线索，我们需要使用 Flyway Undo。

## 3.使用飞行路线撤消

首先，需要注意的是 **Flyway 撤销是 Flyway 的一项商业功能，在社区版中不可用。**因此，我们需要专业版或企业版才能使用该功能。

### 3.1.创建迁移文件

首先，让我们创建一个名为`V1_0__create_book_table.sql`的迁移文件:

```java
create table book (
  id numeric,
  title varchar(128),
  author varchar(256),
  constraint pk_book primary key (id)
);
```

其次，让我们创建相应的撤销迁移文件`U1_0__create_book_table.sql`:

```java
drop table book;
```

在我们的撤销迁移中，请注意文件名前缀“U”与正常迁移前缀“V”的比较。此外，在我们的撤销迁移文件中，**我们编写了撤销相应迁移文件**更改的 SQL。在我们的例子中，我们删除了由正常迁移创建的表。

### 3.2.应用迁移

接下来，让我们检查迁移的当前状态:

```java
./flyway -pro info
```

这为我们提供了以下输出:

```java
+-----------+---------+-------------------+------+--------------+---------+----------+
| Category  | Version | Description       | Type | Installed On | State   | Undoable |
+-----------+---------+-------------------+------+--------------+---------+----------+
| Versioned | 1.0     | create book table | SQL  |              | Pending | Yes      |
+-----------+---------+-------------------+------+--------------+---------+----------+ 
```

请注意最后一列`Undoable`，它表示 Flyway 已经检测到一个撤销迁移文件，该文件伴随着我们的正常迁移文件。

接下来，让我们应用我们的迁移:

```java
./flyway migrate
```

当它完成时，我们的迁移也完成了，我们的模式有了一个新的 book 表:

```java
 List of relations
 Schema |         Name          | Type  |  Owner   
--------+-----------------------+-------+----------
 public | book                  | table | baeldung
 public | flyway_schema_history | table | baeldung
(2 rows) 
```

### 3.3.回滚上次迁移

最后，让我们使用命令行撤销最后一次迁移:

```java
./flyway -pro undo
```

命令成功运行后，我们可以再次检查迁移的状态:

```java
./flyway -pro info
```

这为我们提供了以下输出:

```java
+-----------+---------+-------------------+----------+---------------------+---------+----------+
| Category  | Version | Description       | Type     | Installed On        | State   | Undoable |
+-----------+---------+-------------------+----------+---------------------+---------+----------+
| Versioned | 1.0     | create book table | SQL      | 2020-08-22 15:48:00 | Undone  |          |
| Undo      | 1.0     | create book table | UNDO_SQL | 2020-08-22 15:49:47 | Success |          |
| Versioned | 1.0     | create book table | SQL      |                     | Pending | Yes      |
+-----------+---------+-------------------+----------+---------------------+---------+----------+
```

请注意撤销是如何成功的，第一次迁移又回到挂起状态。此外，与第一种方法相比，**审计跟踪清楚地显示了被回滚的迁移。**

虽然 Flyway Undo 很有用，但是它假设整个迁移已经成功。例如，如果迁移中途失败，它可能无法按预期工作。

## 4.结论

在这个简短的教程中，我们研究了如何使用标准迁移来恢复数据库。我们还研究了使用 Flyway Undo 回滚迁移的官方方法。像往常一样，我们所有与本教程相关的代码都可以在 GitHub 上找到。