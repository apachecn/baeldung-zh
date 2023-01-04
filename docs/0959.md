# Liquibase 回滚简介

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/liquibase-rollback>

## 1。概述

在我们的[上一篇文章](/web/20221202033054/https://www.baeldung.com/liquibase-refactor-schema-of-java-app)中，我们展示了 Liquibase 作为管理数据库模式和数据的工具。

在本文中，我们将深入探讨回滚功能，以及如何撤销 Liquibase 操作。

自然，这是任何生产级系统的关键特性。

## 2。Liquibase 迁移的类别

有两种 Liquibase 操作，会产生不同的 rollback 语句:

*   **自动**，迁移可以确定性地生成回滚所需的步骤
*   **手动**，我们需要发出回滚命令，因为迁移指令不能用于确定性地识别语句

例如，**一个`“create table”`语句的回滚将是到`drop”`已创建的表**。这是可以确定的，因此可以自动生成 rollback 语句。

另一方面，**一个`“drop table”`命令的回滚语句不可能被确定**。不可能确定表的最后状态，因此不能自动生成 rollback 语句。**这些类型的迁移语句需要一个手动回滚指令。**

## 3。编写简单的回滚语句

让我们编写一个简单的变更集，它将在执行时创建一个表，并向变更集添加一个 rollback 语句:

```
<changeSet id="testRollback" author="baeldung">
    <createTable tableName="baeldung_turorial">
        <column name="id" type="int"/>
        <column name="heading" type="varchar(36)"/>
        <column name="author" type="varchar(36)"/>
    </createTable>
    <rollback>
        <dropTable tableName="baeldung_test"/>
    </rollback>
</changeSet>
```

上述例子属于上述第一类。如果我们不添加，它将自动创建一个 rollback 语句。但是我们可以通过创建 rollback 语句来覆盖默认行为。

我们可以使用以下命令运行迁移:

```
mvn liquibase:update
```

执行后，我们可以使用以下命令回滚操作:

```
mvn liquibase:rollback
```

这将执行变更集的回滚段，并应恢复在更新阶段完成的任务。但是如果我们单独发出这个命令，构建将会失败。

原因是——我们没有指定回滚的限制；通过回滚到初始阶段，数据库将被完全清除。因此，当满足条件时，必须定义以下三个约束之一来限制回滚操作:

*   `rollbackTag`
*   `rollbackCount`
*   `rollbackDate`

### 3.1。回滚到标签

我们可以将数据库的特定状态定义为一个标签。因此，我们可以引用该状态。回滚到标记名“1.0”如下所示:

```
mvn liquibase:rollback -Dliquibase.rollbackTag=1.0
```

这将执行在标记“1.0”之后执行的所有变更集的 rollback 语句。

### 3.2。按计数回滚

这里，我们定义了需要回滚多少个变更集。如果我们将其定义为一个，最后一次变更集执行将被回滚:

```
mvn liquibase:rollback -Dliquibase.rollbackCount=1
```

### 3.3。回滚到日期

我们可以将回滚目标设置为一个日期，因此，在该日期之后执行的任何变更集都将被回滚:

```
mvn liquibase:rollback "-Dliquibase.rollbackDate=Jun 03, 2017"
```

日期格式必须是 ISO 数据格式，或者应该与执行平台的`DateFormat.getDateInstance()`值相匹配。

## 4。回滚变更集选项

让我们探讨一下 rollback 语句在变更集中可能的用法。

### 4.1。多语句回滚

单个回滚标记可以包含不止一条要执行的指令:

```
<changeSet id="multiStatementRollback" author="baeldung">
    <createTable tableName="baeldung_tutorial2">
        <column name="id" type="int"/>
        <column name="heading" type="varchar(36)"/>
    </createTable>
    <createTable tableName="baeldung_tutorial3">
        <column name="id" type="int"/>
        <column name="heading" type="varchar(36)"/>
    </createTable>
    <rollback>
        <dropTable tableName="baeldung_tutorial2"/>
        <dropTable tableName="baeldung_tutorial3"/>
    </rollback>
</changeSet>
```

这里，我们将两个表放在同一个回滚标记中。我们也可以将任务分解到多个语句上。

### 4.2。多个回滚标签

在一个变更集中，我们可以有多个回滚标记。它们按照在变更集中出现的顺序执行:

```
<changeSet id="multipleRollbackTags" author="baeldung">
    <createTable tableName="baeldung_tutorial4">
        <column name="id" type="int"/>
        <column name="heading" type="varchar(36)"/>
    </createTable>
    <createTable tableName="baeldung_tutorial5">
        <column name="id" type="int"/>
        <column name="heading" type="varchar(36)"/>
    </createTable>
    <rollback>
        <dropTable tableName="baeldung_tutorial4"/>
    </rollback>
    <rollback>
        <dropTable tableName="baeldung_tutorial5"/>
    </rollback>
</changeSet> 
```

### 4.3。引用另一个变更集进行回滚

如果我们要更改数据库的一些细节，我们可以引用另一个变更集，可能是原始的变更集。这将减少代码重复，并且可以正确地恢复已完成的更改:

```
<changeSet id="referChangeSetForRollback" author="baeldung">
    <dropTable tableName="baeldung_tutorial2"/>
    <dropTable tableName="baeldung_tutorial3"/>
    <rollback changeSetId="multiStatementRollback" changeSetAuthor="baeldung"/>
</changeSet>
```

### 4.4。空回滚标签

默认情况下，如果我们没有提供回滚脚本，Liquibase 会尝试生成一个回滚脚本。如果我们需要中断此功能，我们可以使用空的回滚标记，以便回滚操作不会被恢复:

```
<changeSet id="emptyRollback" author="baeldung">
    <createTable tableName="baeldung_tutorial">
        <column name="id" type="int"/>
        <column name="heading" type="varchar(36)"/>
        <column name="author" type="varchar(36)"/>
    </createTable>
    <rollback/>
</changeSet>
```

## 5。回滚命令选项

除了将数据库回滚到以前的状态，Liquibase 还有许多不同的用途。它们是，生成回滚 SQL，创建未来的回滚脚本，最后，我们可以测试迁移和回滚，两者都在一个步骤中完成。

### 5.1。生成回滚脚本

与回滚一样，我们有三个选项来生成回滚 SQL。这些是:

*   **roll back SQL**`<tag>`–将数据库回滚到上述标签的脚本
*   **rollbackToDateSQL**
*   **roll back count SQL**`<value>`–一个 SQL 脚本，用于将数据库回滚到前面提到的步骤数的状态

让我们来看一个实际例子:

```
mvn liquibase:rollbackCountSQL 2
```

### 5.2。生成未来回滚脚本

此命令生成所需的 rollback SQL 命令，以便将数据库从当前状态(此时符合运行条件的变更集已完成)恢复到当前状态。如果需要为我们将要执行的更改提供回滚脚本，这将非常有用:

如果需要为我们将要执行的更改提供回滚脚本，这将非常有用:

```
mvn liquibase:futureRollbackSQL 
```

### 5.3。运行更新测试回滚

该命令执行数据库更新，然后回滚更改集，使数据库恢复到当前状态。在执行完成后，未来回滚和更新测试回滚都不会改变当前数据库。但是更新测试回滚执行实际的迁移，然后回滚。

这可用于测试更新更改的执行，而无需永久更改数据库:

```
mvn liquibase:updateTestingRollback
```

## 6。结论

在这个快速教程中，我们探索了 Liquibase 回滚功能的一些命令行和变更集特性。

和往常一样，源代码可以在 GitHub 上找到[。](https://web.archive.org/web/20221202033054/https://github.com/eugenp/tutorials/tree/master/persistence-modules/liquibase)