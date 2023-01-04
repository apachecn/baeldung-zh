# Flyway 回调指南

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/flyway-callbacks>

## 1。简介

Flyway 库允许我们通过跟踪存储为 SQL 源代码的更改来版本化数据库。**每组变化被称为一个`migration`。**

使用包括`migrate`、`clean, info, validate,`、`baseline`和`repair`在内的一组命令，将各个迁移按顺序应用于数据库。它们是根据目标数据库的当前版本以受控方式应用的。

虽然迁移通常足以涵盖大多数用例，但是有许多场景非常适合回调。

在本文中，我们将使用 Flyway 回调来挂钩它提供的各种命令的生命周期。

## 2。用例场景

我们可能有一个非常具体的需求，需要回调所提供的那种灵活性。以下是一些可能的使用案例:

*   **重建物化视图—**每当我们应用影响那些视图的基表的迁移时，我们可能想要**重建物化视图**。SQL 回调非常适合执行这种逻辑
*   **刷新缓存—**也许我们有一个迁移，它修改了碰巧被缓存的数据。我们可以使用回调来**刷新缓存**，确保我们的应用程序从数据库中提取新数据
*   **调用外部系统—**使用回调，我们可以**使用任意技术调用外部系统**。例如，我们可能想要发布事件、发送电子邮件或触发服务器重启

## 3。支持的回调

每个可用的 Flyway 命令都有相应的`before`和`after`回调事件。关于这些命令的更多信息，请参考[我们的主要飞行路线文章](/web/20221128042254/https://www.baeldung.com/database-migrations-with-flyway)，或者[官方文档](https://web.archive.org/web/20221128042254/https://flywaydb.org/documentation/)。

*   BEFORE_ events 在操作执行前触发。
*   AFTER_ events 在操作成功后触发。这些`after` 事件还有几个更细粒度的事件:
    *   操作失败后，将触发等效错误。
    *   OPERATION_FINISH 事件在操作完成后触发。
*   `migrate`和`undo` 也有 _EACH 事件，每个迁移都会触发该事件。`migrate`和 undo 命令提供了这些额外的回调函数，因为通常情况下运行这些命令会导致许多迁移的执行。

在[事件](https://web.archive.org/web/20221128042254/https://flywaydb.org/documentation/usage/api/javadoc/org/flywaydb/core/api/callback/Event.html)类中可以找到回调事件的完整列表。

例如，`clean`命令的回调事件是`BEFORE_CLEAN`和`AFTER_CLEAN`。Flyway 在`clean` 命令执行前后立即触发它们。

回想一下我们在简介中讨论的内容，可用的命令有:`migrate`、`clean, info, validate,`、`baseline`和`repair`。

Flyway 的作者提供了这些额外的钩子，让我们在 Flyway 使用的最高粒度级别上控制定制回调逻辑，也就是说，单个迁移。

## 4。依赖性

为了了解回调在实践中是如何工作的，让我们看一个简单的例子。我们可以通过在我们的`pom.xml`中声明 flyway-core 为依赖项来开始我们的示例:

```java
<dependency>
    <groupId>org.flywaydb</groupId>
    <artifactId>flyway-core</artifactId>
    <version>8.5.13</version>
</dependency>
```

我们可以在 [Maven Central](https://web.archive.org/web/20221128042254/https://search.maven.org/classic/#search%7Cga%7C1%7Cg%3A%22org.flywaydb%22%20AND%20a%3A%22flyway-core%22) 上找到`flyway-core`的最新版本。

## 5。回调

Flyway 使我们能够使用两种不同的方法创建回调，Java 或 SQL。前者是最灵活的一种。它为我们提供了执行任意代码的自由。

后者让我们直接与数据库交互。

### 5.1。Java 回调

**Java API 契约在`Callback`接口中定义。**

在最简单的情况下，要创建自定义回调，我们需要实现`Callback`接口，如我们的`ExampleFlywayCallback:`所示

```java
public class ExampleFlywayCallback implements Callback {

    private final Log log = LogFactory.getLog(getClass());

    @Override
    public boolean supports(Event event, Context context) {
        return event == Event.AFTER_EACH_MIGRATE;
    }

    @Override
    public boolean canHandleInTransaction(Event event, Context context) {
        return true;
    }

    @Override
    public void handle(Event event, Context context) {
        if (event == Event.AFTER_EACH_MIGRATE) {
            log.info("> afterEachMigrate");
        }
    }

    @Override
    public String getCallbackName() {
        return ExampleFlywayCallback.class.getSimpleName();
    }
}
```

### 5.2。SQL 回调

**SQL 回调契约通过使用包含在配置为`locations(s)`的目录中的具有特定名称的文件来定义。** Flyway 将在其配置的`locations(s)`中查找 SQL 回调文件，并相应地执行它们。

例如，在执行`migrate`命令期间，配置为`location`的目录中名为`beforeEachMigrate.sql`的文件将在每个迁移脚本之前运行。

## 6。配置和执行

在下面的例子中，我们配置了 Java 回调，并指定了两个 SQL 脚本位置:一个包含我们的迁移，另一个包含 SQL 回调。

没有必要为迁移和 SQL 回调配置单独的位置，但是我们在示例中以这种方式进行设置，以演示如何将它们分开:

```java
@Test
public void migrateWithSqlAndJavaCallbacks() {
    Flyway flyway = Flyway.configure()
      .dataSource(dataSource)
      .locations("db/migration", "db/callbacks")
      .callbacks(new ExampleFlywayCallback())
      .load();
    flyway.migrate();
}
```

如果我们在 Java 和 SQL 中都定义了一个`beforeEachMigrate`,那么知道 Java 回调将首先被执行，然后紧接着执行 SQL 回调会很有帮助。

这可以从上述测试的输出中看出:

```java
21:50:45.677 [main] INFO  c.b.f.FlywayApplicationUnitTest - > migrateWithSqlAndJavaCallbacks
21:50:45.848 [main] INFO  o.f.c.i.license.VersionPrinter - Flyway Community Edition 8.0.0 by Redgate
21:50:45.849 [main] INFO  o.f.c.i.d.base.BaseDatabaseType - Database: jdbc:h2:mem:DATABASE (H2 1.4)
21:50:45.938 [main] INFO  o.f.core.internal.command.DbValidate - Successfully validated 2 migrations (execution time 00:00.021s)
21:50:45.951 [main] INFO  o.f.c.i.s.JdbcTableSchemaHistory - Creating Schema History table "PUBLIC"."flyway_schema_history" ...
21:50:46.003 [main] INFO  o.f.c.i.c.SqlScriptCallbackFactory - Executing SQL callback: beforeMigrate - 
21:50:46.015 [main] INFO  o.f.core.internal.command.DbMigrate - Current version of schema "PUBLIC": << Empty Schema >>
21:50:46.023 [main] INFO  o.f.c.i.c.SqlScriptCallbackFactory - Executing SQL callback: beforeEachMigrate - 
21:50:46.024 [main] INFO  o.f.core.internal.command.DbMigrate - Migrating schema "PUBLIC" to version "1.0 - add table one"
21:50:46.025 [main] INFO  c.b.f.ExampleFlywayCallback - > afterEachMigrate
21:50:46.046 [main] INFO  o.f.c.i.c.SqlScriptCallbackFactory - Executing SQL callback: beforeEachMigrate - 
21:50:46.046 [main] INFO  o.f.core.internal.command.DbMigrate - Migrating schema "PUBLIC" to version "1.1 - add table two"
21:50:46.047 [main] INFO  c.b.f.ExampleFlywayCallback - > afterEachMigrate
21:50:46.067 [main] INFO  o.f.core.internal.command.DbMigrate - Successfully applied 2 migrations to schema "PUBLIC", now at version v1.1 (execution time 00:00.060s)
```

## 7。结论

在本文中，我们研究了如何在 Java 和 SQL 中使用 Flyway 回调机制。我们查看了可能的用例，并详述了一个例子。

和往常一样，所有源代码都可以在 GitHub 上找到[。](https://web.archive.org/web/20221128042254/https://github.com/eugenp/tutorials/tree/master/persistence-modules/flyway)