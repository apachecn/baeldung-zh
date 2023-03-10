# 用 Spring Boot 修复飞行路线

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-boot-flyway-repair>

## 1.概观

Flyway 迁移并不总是按计划进行。在本教程中，我们将**探索从失败的迁移中恢复的选项**。

## 2.设置

让我们从一个基本的 Flyway 配置的 Spring Boot 项目开始。它有`[flyway-core](https://web.archive.org/web/20220628131626/https://search.maven.org/search?q=g:org.flywaydb%20AND%20a:flyway-core)`、`[spring-boot-starter-jdbc](https://web.archive.org/web/20220628131626/https://search.maven.org/search?q=a:spring-boot-starter-jdbc),` 和`[flyway-maven-plugin](https://web.archive.org/web/20220628131626/https://search.maven.org/search?q=g:org.flywaydb%20AND%20a:flyway-maven-plugin)` 依赖关系。

更多配置细节请参考我们的文章[介绍 Flyway](/web/20220628131626/https://www.baeldung.com/database-migrations-with-flyway) 。

### 2.1.配置

首先，让我们添加两个不同的概要文件。这将使我们能够轻松地针对不同的数据库引擎运行迁移:

```java
<profile>
    <id>h2</id>
    <activation>
        <activeByDefault>true</activeByDefault>
    </activation>
    <dependencies>
        <dependency>
            <groupId>com.h2database</groupId>
	    <artifactId>h2</artifactId>
        </dependency>
    </dependencies>
</profile>
<profile>
    <id>postgre</id>
    <dependencies>
        <dependency>
            <groupId>org.postgresql</groupId>
            <artifactId>postgresql</artifactId>
        </dependency>
    </dependencies>
</profile>
```

让我们也为每个概要文件添加 Flyway 数据库配置文件。

首先，我们创建`application-h2.properties`:

```java
flyway.url=jdbc:h2:file:./testdb;DB_CLOSE_ON_EXIT=FALSE;AUTO_RECONNECT=TRUE;MODE=MySQL;DATABASE_TO_UPPER=false;
flyway.user=testuser
flyway.password=password
```

之后，让我们创建 PostgreSQL `application-postgre.properties`:

```java
flyway.url=jdbc:postgresql://127.0.0.1:5431/testdb
flyway.user=testuser
flyway.password=password
```

注意:我们可以调整 PostgreSQL 配置来匹配一个已经存在的数据库，或者我们可以使用代码示例中的[和`docker-compose`文件。](https://web.archive.org/web/20220628131626/https://github.com/eugenp/tutorials/tree/master/persistence-modules/flyway)

### 2.2.迁移

让我们添加我们的第一个迁移文件，`V1_0__add_table.sql`:

```java
create table table_one (
  id numeric primary key
);
```

现在让我们添加第二个包含错误的迁移文件，`V1_1__add_table.sql:`

```java
create table <span style="color: #ff0000">table_one</span> (
  id numeric primary key
);
```

我们故意使用相同的表名，犯了一个错误。这将导致浮动迁移错误。

## 3.运行迁移

现在，让我们运行应用程序并尝试应用迁移。

首先为默认的`h2`配置文件:

```java
mvn spring-boot:run
```

然后对于`postgre`轮廓:

```java
mvn spring-boot:run -Ppostgre
```

正如所料，第一次迁移成功，而第二次迁移失败:

```java
Migration V1_1__add_table.sql failed
...
Message    : Table "TABLE_ONE" already exists; SQL statement:
```

### 3.1.检查状态

在修复数据库之前，让我们通过运行以下命令来检查 Flyway 迁移状态:

```java
mvn flyway:info -Ph2
```

如预期的那样，这将返回:

```java
+-----------+---------+-------------+------+---------------------+---------+
| Category  | Version | Description | Type | Installed On        | State   |
+-----------+---------+-------------+------+---------------------+---------+
| Versioned | 1.0     | add table   | SQL  | 2020-07-17 12:57:35 | Success |
| Versioned | 1.1     | add table   | SQL  | 2020-07-17 12:57:35 | <span style="color: #ff0000">Failed</span>  |
+-----------+---------+-------------+------+---------------------+---------+ 
```

但是当我们用以下命令检查 PostgreSQL 的状态时:

```java
mvn flyway:info -Ppostgre
```

我们注意到第二次迁移的状态是`Pending` 而不是`Failed:`

```java
+-----------+---------+-------------+------+---------------------+---------+
| Category  | Version | Description | Type | Installed On        | State   |
+-----------+---------+-------------+------+---------------------+---------+
| Versioned | 1.0     | add table   | SQL  | 2020-07-17 12:57:48 | Success |
| Versioned | 1.1     | add table   | SQL  |                     | <span style="color: #339966">Pending</span> |
+-----------+---------+-------------+------+---------------------+---------+
```

不同之处在于, **PostgreSQL 支持 DDL 事务**,而其他如 H2 或 MySQL 则不支持。因此，PostgreSQL **能够为失败的迁移**回滚事务。让我们看看当我们试图修复数据库时，这种差异是如何影响事情的。

### 3.2.更正错误并重新运行迁移

让我们通过将表名从`table_one`更正为`table_two.`来修复迁移文件`V1_1__add_table.sql`

现在，让我们再次尝试运行该应用程序:

```java
mvn spring-boot:run -Ph2
```

我们现在注意到 H2 迁移失败，原因如下:

```java
Validate failed: 
Detected failed migration to version 1.1 (add table)
```

只要此版本存在已经失败的迁移，Flyway 就不会重新运行`version 1.1`迁移。

另一方面，`postgre`概要文件运行成功。如前所述，由于回滚，状态是干净的，可以应用正确的迁移。

实际上，通过运行`mvn flyway:info -Ppostgre` ，我们可以看到应用了`Success`的两个迁移。因此，总之，对于 PostgreSQL，我们所要做的就是纠正我们的迁移脚本并重新触发迁移。

## 4.手动修复数据库状态

修复数据库状态的第一种方法是**手动从`flyway_schema_history` 表**中删除 Flyway 条目。

让我们简单地对数据库运行以下 SQL 语句:

```java
delete from flyway_schema_history where version = '1.1';
```

现在，当我们再次运行`mvn spring-boot:run`时，我们看到迁移被成功应用。

然而，直接操作数据库可能并不理想。所以，让我们看看还有什么其他的选择。

## 5.车道修理

### 5.1.修复失败的迁移

让我们通过添加另一个运行应用程序的中断迁移`V1_2__add_table.sql` 文件`,` 继续前进，并返回到迁移失败的状态。

另一种修复数据库状态的方法是使用 `[flyway:repair](https://web.archive.org/web/20220628131626/https://flywaydb.org/documentation/command/repair)` 工具。更正 SQL 文件后，我们可以运行以下命令，而不是手动操作`flyway_schema_history`表:

```java
mvn flyway:repair
```

这将导致:

```java
Successfully repaired schema history table "PUBLIC"."flyway_schema_history"
```

在幕后，Flyway 只是从`flyway_schema_history`表中删除失败的迁移条目。

现在，我们可以再次运行`flyway:info`，并看到上次迁移的状态从`Failed`更改为`Pending`。

让我们再次运行应用程序。如我们所见，修正后的迁移现在已成功应用。

### 5.2.重新对齐校验和

通常建议永远不要更改成功应用的迁移。但是可能会有无法回避的情况。

因此，在这样的场景中，让我们通过在文件开头添加注释来改变迁移`V1_1__add_table.sql` 。

现在运行应用程序，我们看到一条**“迁移校验和不匹配”的错误消息**，如下所示:

```java
Migration checksum mismatch for migration version 1.1
-> Applied to database : 314944264
-> Resolved locally    : 1304013179
```

发生这种情况是因为我们改变了已经应用的迁移，并且 Flyway 检测到不一致。

为了让**重新排列校验和，我们可以使用相同的`flyway:repair`命令**。但是，这一次不会执行迁移。只有`flyway_schema_history`表中的`version 1.1`条目的校验和将被更新，以反映更新的迁移文件。

通过在修复后再次运行应用程序，我们注意到应用程序现在可以成功启动了。

注意，在这种情况下，我们通过 Maven 使用了`flyway:repair`。另一种方法是安装 Flyway 命令行工具，运行 [`flyway repair`](https://web.archive.org/web/20220628131626/https://flywaydb.org/documentation/command/repair) 。效果是一样的: **`flyway repair`将从`flyway_schema_history`表中删除失败的迁移，并重新调整已经应用的迁移的校验和**。

## 6.Flyway 回调

如果我们不想手动干预，我们可以考虑一种方法，在失败的迁移之后，**自动从`flyway_schema_history`中清除失败的条目。为此，我们可以使用`afterMigrateError` [的飞程回调](/web/20220628131626/https://www.baeldung.com/flyway-callbacks)。**

我们首先创建 SQL 回调文件`db/callback/afterMigrateError__repair.sql`:

```java
DELETE FROM flyway_schema_history WHERE success=false;
```

每当发生迁移错误时，这将自动从 Flyway 状态历史中删除任何失败的条目。

让我们创建一个`application-callbacks.properties`配置文件配置，它将在飞行路线位置列表中包含`db/callback`文件夹:

```java
spring.flyway.locations=classpath:db/migration,classpath:db/callback
```

现在，在添加了另一个中断的迁移`V1_3__add_table.sql,` 之后，我们运行包含`callbacks`配置文件的应用程序:

```java
mvn spring-boot:run -Dspring-boot.run.profiles=h2,callbacks
...
Migrating schema "PUBLIC" to version 1.3 - add table
Migration of schema "PUBLIC" to version 1.3 - add table failed!
...
Executing SQL callback: afterMigrateError - repair 
```

正如所料，迁移失败了，但是`afterMigrateError`回调运行并清理了`flyway_schema_history`。

只需更正`V1_3__add_table.sql`迁移文件并再次运行应用程序，就足以应用于更正后的迁移。

## 7.摘要

在本文中，我们研究了从失败的 Flyway 迁移中恢复的不同方法。

我们看到了像 PostgreSQL 这样的数据库——即支持 DDL 事务的数据库——不需要额外的工作就可以修复 Flyway 数据库状态。

另一方面，对于像 H2 这样没有这种支持的数据库，我们看到了如何使用 **Flyway repair 来清除 Flyway 历史**并最终应用正确的迁移。

和往常一样，完整的代码可以在 GitHub 上找到[。](https://web.archive.org/web/20220628131626/https://github.com/eugenp/tutorials/tree/master/persistence-modules/flyway-repair)