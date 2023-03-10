# MySQL 中“超过锁等待超时”错误是什么原因造成的？

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/mysql-lock-wait-timeout-error>

## 1.概观

在本教程中，我们将讨论 MySQL 中的“超过锁定等待超时”错误。我们将讨论导致这个错误的原因，以及关于 MySQL 锁的一些细微差别。

为了简单起见，我们将重点关注 MySQL 的 InnoDB 引擎，因为它是最受欢迎的引擎之一。然而，我们可以使用这里使用的相同测试来检查其他引擎的行为。

## 2.锁定 MySQL

[锁](https://web.archive.org/web/20221009095608/https://dev.mysql.com/doc/refman/8.0/en/glossary.html#glos_locking)是一个特殊的对象，它控制对资源的访问。对于 MySQL，这些资源可以是表、行或内部数据结构。

另一个需要习惯的概念是锁定模式。**锁模式“S”(共享)允许事务读取一行**。多个事务可以同时获取特定行的锁。

“X”(独占)锁允许单个事务获取它。该事务可以更新或删除该行，而另一个事务必须等到锁被释放后才能获取它。

MySQL 也有意向锁。这些与表相关，并指示事务打算在表中的行上获取的锁的种类。

锁定对于保证高度并发环境中的一致性和可靠性至关重要。然而，优化性能时，必须做出一些权衡，在这种情况下，选择正确的隔离级别至关重要。

## 3.隔离级别

MySQL InnoDB 提供了四个事务隔离级别。它们在性能、一致性、可靠性和再现性之间提供不同程度的平衡。它们分别从最不严格到最严格:

*   READ UNCOMMITTED **:** 简而言之，所有事务都可以读取其他人所做的所有更改，即使它们没有被提交
*   已提交读取:只有已提交的更改对其他事务可见
*   可重复读取:第一个查询定义了一个快照，它成为该行的基线。即使另一个事务在读取后立即更改了行，如果在第一次查询后没有更改，基线也将总是被返回
*   SERIALIZABLE:行为与前一个完全一样，只是如果禁用了自动提交，它会在任何更新或删除期间锁定行，并且只允许在提交后进行读取

现在我们已经了解了不同隔离级别的工作原理，让我们运行一些测试来检查锁定场景。首先，为了保持简短，我们将在默认隔离级别 REPEATABLE READ 下运行所有测试。但是，稍后我们可以运行所有其他级别的测试。

## 4.监视

我们将在这里看到的工具不一定适用于生产使用。相反，它们会让我们了解在引擎盖下发生了什么。

这些命令将描述 MySQL 如何处理事务，哪些锁与哪些事务相关，或者如何从这些事务中获取更多数据。因此，这些工具将在我们的测试中帮助我们，但在生产环境中可能不适用，或者至少在错误已经发生时不适用。

### 4.1.InnoDB 状态

**命令`[SHOW ENGINE INNODB STATUS](https://web.archive.org/web/20221009095608/https://dev.mysql.com/doc/refman/8.0/en/show-engine.html)`向我们展示了大量关于内部结构、对象和指标**的信息。根据可用和活动连接的数量，输出可能会被截断。然而，我们只需要查看我们的用例的事务部分。

在“交易”部分，我们会看到类似这样的内容:

*   活动事务的数量
*   每个交易的状态
*   每个事务中涉及的表的数量
*   事务获取的锁的数量
*   可能是已执行的语句，该语句可能持有事务
*   关于锁等待的信息

那里有更多的东西可看，但这对我们来说已经足够了。

### 4.2.过程列表

**命令`[SHOW PROCESSLIST](https://web.archive.org/web/20221009095608/https://dev.mysql.com/doc/refman/5.7/en/show-processlist.html#:~:text=The%20SHOW%20PROCESSLIST%20statement%20is,those%20belonging%20to%20other%20users.)`显示当前打开会话的表格**，表格显示以下信息:

*   会话 id
*   用户名
*   主机已连接
*   数据库
*   命令/当前活动语句类型
*   执行时间
*   连接的状态
*   会话描述

这个命令让我们了解不同的活动会话、它们的状态和它们的活动。

### 4.3.选择指令

MySQL 通过一些表公开了一些有用的信息，我们可以使用它们来理解在给定场景中应用的锁策略的种类。它们还保存诸如当前事务的 id 之类的东西。

**为了本文的目的，我们将使用表格`information_schema.innodb_trx`和`performance_schema.data_locks`。**

## 5.测试设置

为了运行我们的测试，我们将使用 MySQL 的一个 [docker](/web/20221009095608/https://www.baeldung.com/ops/docker-guide) 映像来创建我们的数据库并填充我们的测试模式，以便我们可以练习一些事务场景:

```java
# Create MySQL container 
docker run --network host --name example_db -e MYSQL_ROOT_PASSWORD=root -d mysql
```

一旦我们有了数据库服务器，我们就可以通过连接到它并执行脚本来创建模式:

```java
# Logging in MySQL 
docker exec -it example_db mysql -uroot -p
```

然后，在输入密码后，让我们创建数据库并插入一些数据:

```java
CREATE DATABASE example_db;
USE example_db;
CREATE TABLE zipcode ( 
    code varchar(100) not null, 
    city varchar(100) not null, 
    country varchar(3) not null,
    PRIMARY KEY (code) 
);
INSERT INTO zipcode(code, city, country) 
VALUES ('08025', 'Barcelona', 'ESP'), 
       ('10583', 'New York', 'USA'), 
       ('11075-430', 'Santos', 'BRA'), 
       ('SW6', 'London', 'GBR'); 
```

## 6.测试场景

需要记住的最重要的一点是，当一个事务正在等待另一个事务获取锁时，会发生“超过锁等待超时”错误。

事务等待的时间取决于在全局或会话级别定义的属性`[innodb_lock_wait_timeout](https://web.archive.org/web/20221009095608/https://dev.mysql.com/doc/refman/8.0/en/innodb-parameters.html#sysvar_innodb_lock_wait_timeout)`中的值。

面临这种错误的可能性取决于复杂性和每秒的事务数量。然而，我们将尝试重现一些常见的场景。

**另外一点可能值得一提的是，一个简单的重试策略就可以解决这个错误导致的问题。**

为了在测试过程中帮助我们，我们将对打开的所有会话运行以下命令:

```java
USE example_db;
-- Set our timeout to 10 seconds
SET @@SESSION.innodb_lock_wait_timeout = 10; 
```

这将锁等待超时定义为 10 秒，防止我们等待太久才看到错误。

### 6.1.行锁

因为行锁是在不同的情况下获得的，所以让我们试着重现一个例子。

首先，我们将使用前面看到的登录 MySQL 脚本从两个不同的会话连接到服务器。之后，让我们在两个会话中运行下面的语句:

```java
SET autocommit=0;
UPDATE zipcode SET code = 'SW6 1AA' WHERE code = 'SW6';
```

10 秒钟后，第二个会话将失败:

```java
mysql>  UPDATE zipcode SET code = 'SW6 1AA' WHERE code = 'SW6';
ERROR 1205 (HY000): Lock wait timeout exceeded; try restarting transaction 
```

由于禁用了自动提交，第一个会话启动了一个事务，因此会发生此错误。接下来，一旦`UPDATE`语句在事务中运行，就会获得该行的排他锁。但是，不执行任何提交，使事务保持打开状态，并导致另一个事务一直等待。由于提交从未发生，锁等待的超时达到了极限。这也适用于`DELETE`语句。

### 6.2.检查数据锁表中的行锁

现在，让我们在两个会话中回滚，并像以前一样在第一个会话中运行脚本，但这一次，在第二个会话中，让我们运行以下语句:

```java
SET autocommit=0;
UPDATE zipcode SET code = 'Test' WHERE code = '08025';
```

正如我们所观察到的，两条语句都成功执行，因为它们不再需要同一行的锁。

为了确认这一点，我们将在任何会话或新会话中运行以下语句:

```java
SELECT * FROM performance_schema.data_locks;
```

**上面的语句返回四行，其中两行是表意向锁，指定一个事务可能要锁定表中的一行，另外两行是记录锁**。查看列`LOCK_TYPE`、`LOCK_MODE`和`LOCK_DATA`，我们可以确认我们刚刚描述的锁:

[![](img/beca1d67364187540a9150e4daafc759.png)](/web/20221009095608/https://www.baeldung.com/wp-content/uploads/2022/08/data_lock_result_set_full.png)

在会话和查询中再次运行 rollback，结果是一个空数据集。

### 6.3.行锁和索引

这次让我们在我们的`WHERE`子句中使用不同的列。在第一个会话中，我们将运行:

```java
SET autocommit=0;
UPDATE zipcode SET city = 'SW6 1AA' WHERE country = 'USA';
```

在第二个示例中，让我们运行这些语句:

```java
SET autocommit=0;
UPDATE zipcode SET city = '11025-030' WHERE country = 'BRA';
```

意想不到的事情刚刚发生了。尽管语句的目标是两个不同的行，我们还是得到了锁超时错误。好的，如果我们在对表`performance_schema.data_locks`运行`SELECT`语句后立即重复同样的测试，我们将会看到，实际上，第一个会话锁定了所有的行，而第二个会话正在等待。

这个问题与 MySQL [如何执行查询](https://web.archive.org/web/20221009095608/https://dev.mysql.com/doc/refman/8.0/en/using-explain.html#:~:text=The%20EXPLAIN%20statement%20provides%20information,about%20the%20statement%20execution%20plan.)来查找更新的候选项有关，因为`WHERE`子句中使用的列没有索引。MySQL 必须扫描所有的行，以找到符合`WHERE`条件的行，这也会导致这些行被锁定。

**确保我们的陈述是最佳的很重要**。

### 6.4.多表的行锁和更新/删除

锁超时错误的其他常见情况是涉及多个表的`DELETE`和`UPDATE`语句。锁定的行数取决于语句执行计划，但是我们应该记住，所有涉及的表都可能有一些行被锁定。

例如，让我们回滚所有其他事务并执行以下语句:

```java
CREATE TABLE zipcode_backup SELECT * FROM zipcode;
SET autocommit=0;
DELETE FROM zipcode_backup WHERE code IN (SELECT code FROM zipcode); 
```

这里，我们创建了一个表，并启动了一个事务，该事务在一条语句中从`zipcode`表中读取数据，并向`zipcode_backup`表中写入数据。

下一步是在第二个会话中运行以下语句:

```java
SET autocommit=0;
UPDATE zipcode SET code = 'SW6 1AA' WHERE code = 'SW6';
```

事务二再次超时，因为第一个事务获得了表中行的锁。让我们运行`data_lock`表中的`SELECT`语句来演示发生了什么。然后，让我们回滚两个会话。

### 6.5.填充临时表时锁定行

在本例中，让我们在新脚本的第一个会话中混合执行 DDL 和 DMLs:

```java
CREATE TEMPORARY TABLE temp_zipcode SELECT * FROM zipcode; 
```

然后，如果我们重复之前在第二个会话中使用的语句，我们将能够再次看到锁定错误。

### 6.6.共享和排他锁

让我们不要忘记在每个测试结束时回滚两个会话事务。

我们已经讨论了共享锁和排他锁。然而，我们没有看到如何使用`LOCK IN SHARE MODE`和`FOR UPDATE`选项明确地定义它们。首先，让我们使用共享模式:

```java
SET autocommit=0;
SELECT * FROM zipcode WHERE code = 'SW6' LOCK IN SHARE MODE;
```

现在，我们将运行与之前相同的更新，结果仍然是超时。除此之外，我们应该记住这里允许读。

与`SHARE MODE`相反，`FOR UPDATE`不允许读锁，如下所示，当我们在第一个会话中运行一个语句时:

```java
SET autocommit=0;
SELECT * FROM zipcode WHERE code = 'SW6' FOR UPDATE;
```

然后，我们使用第一个会话中使用的`SHARE MODE`选项运行相同的`SELECT`语句，但是现在在第二个会话中，我们将再次观察超时错误。概括地说，`SHARE MODE`锁可以被多个会话获取，它锁定写操作。排他锁或`FOR UPDATE`选项允许读取，但不允许锁定读取或写入。

### 6.7.桌子锁

[表锁](https://web.archive.org/web/20221009095608/https://dev.mysql.com/doc/refman/8.0/en/lock-tables.html)没有超时，不建议用于 InnoDB:

```java
LOCK TABLE zipcode WRITE; 
```

一旦我们运行这个，我们可以打开另一个会话，尝试选择或更新，并检查它是否将被锁定，但这一次，没有超时发生。更进一步，我们可以打开第三个会话并运行:

```java
SHOW PROCESSLIST;
```

它显示了活动会话及其状态，我们将看到第一个会话正在休眠，第二个会话正在等待表的元数据锁。在这种情况下，解决方案是运行下一个命令:

```java
UNLOCK TABLES;
```

我们可能会发现会话等待获取一些元数据锁的其他场景是在 DDL 的执行过程中，比如`ALTER TABLE` s。

### 6.8.间隙锁

[间隙锁](https://web.archive.org/web/20221009095608/https://dev.mysql.com/doc/refman/5.6/en/innodb-locking.html#innodb-gap-locks)发生在索引记录的特定间隔被锁定时，另一个会话试图在该间隔内执行一些操作。在这种情况下，即使是镶件也会受到影响。

让我们考虑在第一个会话中执行的以下语句:

```java
CREATE TABLE address_type ( id bigint(20) not null, name varchar(255) not null, PRIMARY KEY (id) );
SET autocommit=0;
INSERT INTO address_type(id, name) VALUES (1, 'Street'), (2, 'Avenue'), (5, 'Square');
COMMIT;
SET autocommit=0;
SELECT * FROM address_type WHERE id BETWEEN 1 and 5 LOCK IN SHARE MODE;
```

在第二个会话中，我们将运行以下语句:

```java
SET autocommit=0;
INSERT INTO address_type(id, name) VALUES (3, 'Road'), (4, 'Park');
```

在我们运行数据锁之后，我们在第三个会话中选择语句，这样我们就可以检查新的`LOCK MODE`值，`GAP`。这也适用于`UPDATE`和`DELETE`语句。

### 6.9.僵局

默认情况下，MySQL 试图识别死锁，如果它设法解决事务之间的依赖关系图，它会自动终止其中一个任务，以便让其他任务继续执行。否则，我们会得到一个锁超时错误，就像我们之前看到的那样。

让我们模拟一个简单的死锁场景。对于第一个会话，我们执行:

```java
SET autocommit=0;
SELECT * FROM address_type WHERE id = 1 FOR UPDATE;
SELECT tx.trx_id FROM information_schema.innodb_trx tx WHERE tx.trx_mysql_thread_id = connection_id(); 
```

最后一条`SELECT`语句将给出当前的事务 ID。我们稍后需要它来检查日志。然后，对于第二个会话，让我们运行:

```java
SET autocommit=0;
SELECT * FROM address_type WHERE id = 2 FOR UPDATE;
SELECT tx.trx_id FROM information_schema.innodb_trx tx WHERE tx.trx_mysql_thread_id = connection_id();
SELECT * FROM address_type WHERE id = 1 FOR UPDATE;
```

在序列中，我们返回到会话一并运行:

```java
SELECT * FROM address_type WHERE id = 2 FOR UPDATE;
```

很快，我们会得到一个错误:

```java
ERROR 1213 (40001): Deadlock found when trying to get lock; try restarting transaction 
```

最后，我们进入第三个阶段，我们运行:

```java
SHOW ENGINE INNODB STATUS;
```

该命令的输出应该类似于以下内容:

```java
------------------------
LATEST DETECTED DEADLOCK
------------------------
*** (1) TRANSACTION:
TRANSACTION 4036, ACTIVE 11 sec starting index read
mysql tables in use 1, locked 1
LOCK WAIT 3 lock struct(s), heap size 1128, 2 row lock(s)
MySQL thread id 9, OS thread handle 139794615064320, query id 252...
SELECT * FROM address_type WHERE id = 1 FOR UPDATE
*** (1) HOLDS THE LOCK(S):
RECORD LOCKS ... index PRIMARY of table `example_db`.`address_type` trx id 4036 lock_mode X locks rec but not gap
Record lock 
...

*** (1) WAITING FOR THIS LOCK TO BE GRANTED:
RECORD LOCKS ... index PRIMARY of table `example_db`.`address_type` trx id 4036 lock_mode X locks rec but not gap waiting
Record lock
...
*** (2) TRANSACTION:
TRANSACTION 4035, ACTIVE 59 sec starting index read
mysql tables in use 1, locked 1
LOCK WAIT 3 lock struct(s), ... , 2 row lock(s)
MySQL thread id 11, .. query id 253 ...
SELECT * FROM address_type WHERE id = 2 FOR UPDATE
*** (2) HOLDS THE LOCK(S):
RECORD LOCKS ... index PRIMARY of table `example_db`.`address_type` trx id 4035 lock_mode X locks rec but not gap
Record lock
...
*** (2) WAITING FOR THIS LOCK TO BE GRANTED:
RECORD LOCKS ... index PRIMARY of table `example_db`.`address_type` trx id 4035 lock_mode X locks rec but not gap waiting
Record lock
...
*** WE ROLL BACK TRANSACTION (2)
------------
TRANSACTIONS
------------
Trx id counter 4037
...
LIST OF TRANSACTIONS FOR EACH SESSION:
...
---TRANSACTION 4036, ACTIVE 18 sec
3 lock struct(s), heap size 1128, 2 row lock(s)
MySQL thread id 9, ... , query id 252 ... 
```

使用我们之前获得的事务 id，我们可以找到很多有用的信息，比如出错时的连接状态、行锁的数量、最后执行的命令、持有锁的描述以及事务正在等待的锁的描述。之后，它对死锁中涉及的其他事务重复同样的操作。最后，我们还会找到关于哪些事务被回滚的信息。

## 7.结论

在本文中，我们研究了 MySQL 中的锁，它们是如何工作的，以及它们何时导致“超过锁等待超时”错误。

我们定义了测试场景，允许我们重现这个错误，并在处理事务时检查数据库服务器的内部细微差别。