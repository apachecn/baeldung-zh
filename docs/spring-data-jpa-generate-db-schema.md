# 用 Spring 数据 JPA 生成数据库模式

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-data-jpa-generate-db-schema>

## 1.概观

创建持久层时，我们需要将 SQL 数据库模式与我们在代码中创建的对象模型相匹配。这可能需要手动完成大量工作。

在本教程中，**我们将看到如何根据代码**中的实体模型生成和导出数据库模式。

首先，我们将介绍用于模式生成的 JPA 配置属性。然后，我们将探索如何在 Spring Data JPA 中使用这些属性。

最后，我们将探索使用 Hibernate 的本地 API 生成 DDL 的替代方法。

## 2.JPA 模式生成

JPA 2.1 引入了数据库模式生成标准。因此，从这个版本开始，我们可以通过一组预定义的配置属性来控制如何生成和导出数据库模式。

### 2.1.剧本`action`

首先，**为了控制我们将生成哪些 DDL 命令**，JPA 引入了脚本`action`配置选项:

```java
javax.persistence.schema-generation.scripts.action
```

我们可以从四个不同的选项中选择:

*   `none`–不生成任何 DDL 命令
*   `create`–仅生成数据库创建命令
*   `drop`–仅生成数据库删除命令
*   `drop-and-create`–生成数据库删除命令，然后是创建命令

### 2.2.剧本`target`

其次，对于每个指定的脚本`action`，我们需要定义相应的`target`配置:

```java
javax.persistence.schema-generation.scripts.create-target
javax.persistence.schema-generation.scripts.drop-target
```

本质上，脚本`target` **定义了包含模式创建或删除命令**的文件的位置。例如，如果我们选择`drop-and-create `作为脚本`action`，我们将需要指定两个`target`

### 2.3.模式`Source`

最后，为了从我们的实体模型生成模式 DDL 命令，我们应该包含模式`source`配置，并选择`metadata`选项:

```java
javax.persistence.schema-generation.create-source=metadata
javax.persistence.schema-generation.drop-source=metadata
```

在下一节中，我们将展示如何使用 Spring Data JPA 自动生成具有标准 JPA 属性的数据库模式。

## 3.用 Spring 数据 JPA 生成模式

### 3.1.模特们

让我们想象我们正在实现一个用户账户系统，它有一个名为`Account`的[实体](/web/20220627081244/https://www.baeldung.com/jpa-entities):

```java
@Entity
@Table(name = "accounts")
public class Account {

    @Id
    @GeneratedValue
    private Long id;

    @Column(nullable = false, length = 100)
    private String name;

    @Column(name = "email_address")
    private String emailAddress;

    @OneToMany(mappedBy = "account", cascade = CascadeType.ALL)
    private List<AccountSettings> accountSettings = new ArrayList<>();

    // getters and setters
}
```

每个帐户可以有多个帐户设置，因此这里我们将有一个[一对多](/web/20220627081244/https://www.baeldung.com/hibernate-one-to-many)映射:

```java
@Entity
@Table(name = "account_settings")
public class AccountSetting {

    @Id
    @GeneratedValue
    private Long id;

    @Column(name = "name", nullable = false)
    private String settingName;

    @Column(name = "value", nullable = false)
    private String settingValue;

    @ManyToOne
    @JoinColumn(name ="account_id", nullable = false)
    private Account account;

    // getters and setters
} 
```

### 3.2.Spring 数据 JPA 配置

现在，为了生成数据库模式，我们需要将模式生成属性传递给正在使用的持久性提供者。为此，我们将在配置文件中的`spring.jpa.properties`前缀下设置本机 JPA 属性:

```java
spring.jpa.properties.javax.persistence.schema-generation.scripts.action=create
spring.jpa.properties.javax.persistence.schema-generation.scripts.create-target=create.sql
spring.jpa.properties.javax.persistence.schema-generation.scripts.create-source=metadata
```

因此， **Spring Data JPA 在创建`EntityManagerFactory` bean 时会将这些属性传递给持久性提供者**。

### 3.3.`create.sql`文件

因此，在应用程序启动时，上述配置将基于实体映射元数据生成数据库创建命令。此外，DDL 命令被导出到`create.sql`文件中，该文件创建在我们的主项目文件夹中:

```java
create table account_settings (
    id bigint not null,
    name varchar(255) not null,
    value varchar(255) not null,
    account_id bigint not null,
    primary key (id)
)

create table accounts (
    id bigint not null,
    email_address varchar(255),
    name varchar(100) not null,
    primary key (id)
)

alter table account_settings
   add constraint FK54uo82jnot7ye32pyc8dcj2eh
   foreign key (account_id)
   references accounts (id)
```

## 4.使用 Hibernate API 生成模式

如果我们使用 Hibernate，**我们可以直接使用它的本地 API，`SchemaExport`，来生成我们的模式 DDL 命令**。同样，Hibernate API 使用我们的应用程序实体模型来生成和导出数据库模式。

有了 Hibernate 的`SchemaExport`，我们可以显式地使用`drop`、`createOnly,`和`create`方法:

```java
MetadataSources metadataSources = new MetadataSources(serviceRegistry);
metadataSources.addAnnotatedClass(Account.class);
metadataSources.addAnnotatedClass(AccountSettings.class);
Metadata metadata = metadataSources.buildMetadata();

SchemaExport schemaExport = new SchemaExport();
schemaExport.setFormat(true);
schemaExport.setOutputFile("create.sql");
schemaExport.createOnly(EnumSet.of(TargetType.SCRIPT), metadata);
```

当我们运行这段代码时，我们的数据库创建命令被导出到主项目文件夹中的`create.sql`文件中。

`SchemaExport`是 [Hibernate 引导 API](/web/20220627081244/https://www.baeldung.com/hibernate-5-bootstrapping-api) 的一部分。

## 5.模式生成选项

尽管模式生成可以在开发过程中节省我们的时间，但是我们应该只在基本的场景中使用它。

例如，我们可以用它来快速启动开发或测试数据库。

相比之下，对于更复杂的场景，比如数据库迁移，**我们应该使用更精细的工具，比如 [Liquibase](/web/20220627081244/https://www.baeldung.com/liquibase-refactor-schema-of-java-app) 或[Flyway](/web/20220627081244/https://www.baeldung.com/database-migrations-with-flyway)。**

## 6.结论

在本教程中，我们看到了如何在 JPA `schema-generation`属性的帮助下生成和导出我们的数据库模式。随后，我们看到了如何使用 Hibernate 的本地 API`SchemaExport`来实现相同的结果。

和往常一样，我们可以在 GitHub 上找到示例代码[。](https://web.archive.org/web/20220627081244/https://github.com/eugenp/tutorials/tree/master/persistence-modules/spring-data-jpa-crud-2)