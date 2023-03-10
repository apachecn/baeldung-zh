# 将 Java 连接到 MySQL 数据库

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-connect-mysql>

## 1.概观

有许多方法可以从 Java 连接到 MySQL 数据库，在本教程中，我们将探索几种选项来看看如何实现这一点。

我们将从使用 JDBC 和 Hibernate 的最流行的选项开始。

**然后，我们还会看看一些外部库，包括 MyBatis、Apache Cayenne 和 Spring Data** 。在这个过程中，我们将提供一些实际的例子。

## 2。前提条件

我们将假设我们已经在本地主机(默认端口 3306)上安装并运行了一个 MySQL 服务器，并且我们有一个包含以下人员表的测试模式:

```java
CREATE TABLE person 
( 
    ID         INT, 
    FIRST_NAME VARCHAR(100), 
    LAST_NAME  VARCHAR(100)  
);
```

我们还需要`mysql-connector-java`神器，它总是可以从 [Maven Central](https://web.archive.org/web/20221006163534/https://search.maven.org/classic/#search%7Cga%7C1%7Ca%3A%22mysql-connector-java%22%20AND%20g%3A%22mysql%22) 获得:

```java
<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
    <version>8.0.19</version>
</dependency>
```

## 3。 **连接使用 JDBC**

[JDBC](/web/20221006163534/https://www.baeldung.com/java-jdbc) (Java 数据库连接)是一个用于连接和执行数据库查询的 API。

### 3.1.公共属性

**在本文的过程中，我们通常会使用几个常见的 JDBC 属性**:

*   Connection URL – a string that the JDBC driver uses to connect to a database. It can contain information such as where to search for the database, the name of the database to connect to and other configuration properties:

    ```java
    jdbc:mysql://[host][,failoverhost...]
        [:port]/[database]
        [?propertyName1][=propertyValue1]
        [&propertyName2;][=propertyValue2]...
    ```

    我们将这样设置这个属性:`jdbc:mysql://localhost:3306/test?serverTimezone=UTC`

*   驱动程序类——要使用的[驱动程序](/web/20221006163534/https://www.baeldung.com/java-jdbc#jdbc-drivers)的全限定类名。在我们的例子中，我们将使用 MySQL 驱动程序:`com.mysql.cj.jdbc.Driver`
*   用户名和密码 MySQL 帐户的凭证

### 3.2.JDBC 连接示例

让我们看看如何连接到我们的数据库，并通过一个[多资源尝试](/web/20221006163534/https://www.baeldung.com/java-try-with-resources#resources)来执行一个简单的全选:

```java
String sqlSelectAllPersons = "SELECT * FROM person";
String connectionUrl = "jdbc:mysql://localhost:3306/test?serverTimezone=UTC";

try (Connection conn = DriverManager.getConnection(connectionUrl, "username", "password"); 
        PreparedStatement ps = conn.prepareStatement(sqlSelectAllPersons); 
        ResultSet rs = ps.executeQuery()) {

        while (rs.next()) {
            long id = rs.getLong("ID");
            String name = rs.getString("FIRST_NAME");
            String lastName = rs.getString("LAST_NAME");

            // do something with the extracted data...
        }
} catch (SQLException e) {
    // handle the exception
}
```

正如我们所看到的，在`try`主体中，我们遍历结果集并从 person 表中提取值。

## 4。 **连接使用 ORMs**

**更典型地，我们将使用对象关系映射(ORM)框架**连接到我们的 MySQL 数据库。因此，让我们来看一些使用这些框架中更流行的连接示例。

### 4.1.本机 Hibernate APIs

在这一节中，我们将看到如何使用 [Hibernate](https://web.archive.org/web/20221006163534/https://hibernate.org/) 来管理到数据库的 JDBC 连接。

首先，我们需要添加`[hibernate-core](https://web.archive.org/web/20221006163534/https://search.maven.org/classic/#search%7Cga%7C1%7Cg%3A%22org.hibernate%22%20AND%20a%3A%22hibernate-core%22)` Maven 依赖项:

```java
<dependency>
    <groupId>org.hibernate</groupId>
    <artifactId>hibernate-core</artifactId>
    <version>5.4.10.Final</version>
</dependency>
```

Hibernate 要求必须为每个表创建一个实体类。让我们继续定义`Person`类:

```java
@Entity
@Table(name = "Person")
public class Person {
    @Id
    Long id;
    @Column(name = "FIRST_NAME")
    String firstName;

    @Column(name = "LAST_NAME")
    String lastName;

    // getters & setters
} 
```

**另一个重要方面是创建 Hibernate 资源文件，通常命名为`hibernate.cfg.xml`** ，我们将在其中定义配置信息:

```java
<?xml version="1.0" encoding="utf-8"?>
<!DOCTYPE hibernate-configuration PUBLIC
        "-//Hibernate/Hibernate Configuration DTD 3.0//EN"
        "http://www.hibernate.org/dtd/hibernate-configuration-3.0.dtd">

<hibernate-configuration>
    <session-factory>
        <!-- Database connection settings -->
        <property name="connection.driver_class">com.mysql.cj.jdbc.Driver</property>
        <property name="connection.url">jdbc:mysql://localhost:3306/test?serverTimezone=UTC</property>
        <property name="connection.username">username</property>
        <property name="connection.password">password</property>

        <!-- SQL dialect -->
        <property name="dialect">org.hibernate.dialect.MySQL5Dialect</property>

        <!-- Validate the database schema on startup -->
        <property name="hbm2ddl.auto">validate</property>

        <!-- Names the annotated entity class -->
        <mapping class="Person"/>
    </session-factory>
</hibernate-configuration>
```

Hibernate 有很多[配置属性](https://web.archive.org/web/20221006163534/https://docs.jboss.org/hibernate/orm/3.3/reference/en/html/session-configuration.html)。除了标准的连接属性，值得一提的是方言属性，它允许我们为数据库指定 SQL 方言的名称。

框架使用这个属性来正确地将 [Hibernate 查询语言](https://web.archive.org/web/20221006163534/https://docs.jboss.org/hibernate/core/3.3/reference/en/html/queryhql.html) (HQL)语句转换成给定数据库的适当 SQL。Hibernate 提供了 40 多种 SQL 方言。**由于我们在本文中关注 MySQL，我们将坚持使用`MySQL5Dialect`方言。**

最后，Hibernate 还需要通过映射标签知道实体类的全限定名。完成配置后，我们将使用`SessionFactory`类，该类负责创建和汇集 JDBC 连接。

通常，只需为应用程序设置一次:

```java
SessionFactory sessionFactory;
// configures settings from hibernate.cfg.xml 
StandardServiceRegistry registry = new StandardServiceRegistryBuilder().configure().build(); 
try {
    sessionFactory = new MetadataSources(registry).buildMetadata().buildSessionFactory(); 
} catch (Exception e) {
    // handle the exception
}
```

现在我们已经建立了连接，我们可以运行一个查询来从 person 表中选择所有的人:

```java
Session session = sessionFactory.openSession();
session.beginTransaction();

List<Person> result = session.createQuery("from Person", Person.class).list();

result.forEach(person -> {
    //do something with Person instance...   
});

session.getTransaction().commit();
session.close();
```

### 4.2.米巴蒂斯

**[MyBatis](https://web.archive.org/web/20221006163534/https://github.com/mybatis/mybatis-3) 于 2010 年推出，是一个 SQL mapper 框架，其优点是简单**。在另一个教程中，我们谈到了[如何将 MyBatis 与 Spring 和 Spring Boot](/web/20221006163534/https://www.baeldung.com/spring-mybatis) 集成。在这里，我们将重点讨论如何直接配置 MyBatis。

要使用它，我们需要添加 [`mybatis`](https://web.archive.org/web/20221006163534/https://search.maven.org/classic/#artifactdetails%7Corg.mybatis%7Cmybatis%7C3.5.3%7Cjar) 依赖项:

```java
<dependency>
    <groupId>org.mybatis</groupId>
    <artifactId>mybatis</artifactId>
    <version>3.5.3</version>
</dependency>
```

假设我们重用了上面没有注释的`Person`类，我们可以继续创建一个`PersonMapper`接口:

```java
public interface PersonMapper {
    String selectAll = "SELECT * FROM Person"; 

    @Select(selectAll)
    @Results(value = {
       @Result(property = "id", column = "ID"),
       @Result(property = "firstName", column = "FIRST_NAME"),
       @Result(property = "lastName", column = "LAST_NAME")
    })
    List<Person> selectAll();
}
```

下一步是关于 MyBatis 的配置:

```java
Configuration initMybatis() throws SQLException {
    DataSource dataSource = getDataSource();
    TransactionFactory trxFactory = new JdbcTransactionFactory();

    Environment env = new Environment("dev", trxFactory, dataSource);
    Configuration config = new Configuration(env);
    TypeAliasRegistry aliases = config.getTypeAliasRegistry();
    aliases.registerAlias("person", Person.class);

    config.addMapper(PersonMapper.class);
    return config;
}

DataSource getDataSource() throws SQLException {
    MysqlDataSource dataSource = new MysqlDataSource();
    dataSource.setDatabaseName("test");
    dataSource.setServerName("localhost");
    dataSource.setPort(3306);
    dataSource.setUser("username");
    dataSource.setPassword("password");
    dataSource.setServerTimezone("UTC");

    return dataSource;
}
```

**配置包括创建一个`Configuration`对象，该对象是设置的容器，如** `**Environment**.` ，它还包含数据源设置。

然后我们可以使用`Configuration`对象，它通常为应用程序创建一次`SqlSessionFactory`:

```java
Configuration configuration = initMybatis();
SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(configuration);
try (SqlSession session = sqlSessionFactory.openSession()) {
    PersonMapper mapper = session.getMapper(PersonMapper.class);
    List<Person> persons = mapper.selectAll();

    // do something with persons list ...
}
```

### 4.3.阿帕奇卡宴

Apache Cayenne 是一个持久性框架，其首次发布可以追溯到 2002 年。要了解更多，我们建议阅读我们的[阿帕奇卡宴](/web/20221006163534/https://www.baeldung.com/apache-cayenne-orm)介绍。

像往常一样，让我们添加 `[cayenne-server](https://web.archive.org/web/20221006163534/https://search.maven.org/classic/#artifactdetails%7Corg.apache.cayenne%7Ccayenne-server%7C4.0.2%7Cjar)` Maven 依赖项:

```java
<dependency>
    <groupId>org.apache.cayenne</groupId>
    <artifactId>cayenne-server</artifactId>
    <version>4.0.2</version>
</dependency>
```

**我们将特别关注 MySQL 连接设置。在这种情况下，我们将配置`cayenne-project.xml`** :

```java
<?xml version="1.0" encoding="utf-8"?>
<domain project-version="9"> 
    <map name="datamap"/> 
	<node name="datanode" 
	    factory="org.apache.cayenne.configuration.server.XMLPoolingDataSourceFactory" 
		schema-update-strategy="org.apache.cayenne.access.dbsync.CreateIfNoSchemaStrategy"> 
	    <map-ref name="datamap"/> 
		<data-source>
		    <driver value="com.mysql.cj.jdbc.Driver"/> 
			<url value="jdbc:mysql://localhost:3306/test?serverTimezone=UTC"/> 
			<connectionPool min="1" max="1"/> 
			<login userName="username" password="password"/> 
		</data-source> 
	</node> 
</domain>
```

在以`[CayenneDataObject](https://web.archive.org/web/20221006163534/https://cayenne.apache.org/docs/4.0/api/org/apache/cayenne/CayenneDataObject.html)`的形式自动生成`datamap.map.xml`和`Person`类之后，我们可以执行一些查询。

例如，我们将像前面一样继续全选:

```java
ServerRuntime cayenneRuntime = ServerRuntime.builder()
    .addConfig("cayenne-project.xml")
    .build();

ObjectContext context = cayenneRuntime.newContext();
List<Person> persons = ObjectSelect.query(Person.class).select(context);

// do something with persons list...
```

## 5。 **使用 Spring 数据连接**

[Spring Data](https://web.archive.org/web/20221006163534/https://spring.io/projects/spring-data) 是一个基于 Spring 的数据访问编程模型。从技术上讲，Spring Data 是一个伞状项目，它包含许多特定于给定数据库的子项目。

让我们看看如何使用其中的两个项目来连接 MySQL 数据库。

### 5.1.春季数据/ JPA

Spring Data JPA 是一个健壮的框架，它有助于减少样板代码，并提供了一种通过几个预定义的存储库接口之一实现基本 CRUD 操作的机制。除此之外，它还有许多其他有用的功能。

请务必查看我们的[Spring Data JPA 简介](/web/20221006163534/https://www.baeldung.com/the-persistence-layer-with-spring-data-jpa)以了解更多信息。

这个`spring-data-jpa`神器可以在 [Maven Central](https://web.archive.org/web/20221006163534/https://search.maven.org/classic/#search%7Cga%7C1%7Cg%3A%22org.springframework.data%22%20AND%20a%3A%22spring-data-jpa%22) 上找到:

```java
<dependency>
    <groupId>org.springframework.data</groupId>
    <artifactId>spring-data-jpa</artifactId>
    <version>2.2.4.RELEASE</version>
</dependency>
```

我们将继续使用`Person`类。下一步是使用注释配置 JPA:

```java
@Configuration
@EnableJpaRepositories("packages.to.scan")
public class JpaConfiguration {
    @Bean
    public DataSource dataSource() {
        DriverManagerDataSource dataSource = new DriverManagerDataSource();
        dataSource.setDriverClassName("com.mysql.cj.jdbc.Driver");
        dataSource.setUrl("jdbc:mysql://localhost:3306/test?serverTimezone=UTC");
        dataSource.setUsername( "username" );
        dataSource.setPassword( "password" );
        return dataSource;
    }

    @Bean
    public JpaTransactionManager transactionManager(EntityManagerFactory emf) {
      return new JpaTransactionManager(emf);
    }

    @Bean
    public JpaVendorAdapter jpaVendorAdapter() {
      HibernateJpaVendorAdapter jpaVendorAdapter = new HibernateJpaVendorAdapter();
      jpaVendorAdapter.setDatabase(Database.MYSQL);
      jpaVendorAdapter.setGenerateDdl(true);
      return jpaVendorAdapter;
    }

    @Bean
    public LocalContainerEntityManagerFactoryBean entityManagerFactory() {
      LocalContainerEntityManagerFactoryBean lemfb = new LocalContainerEntityManagerFactoryBean();
      lemfb.setDataSource(dataSource());
      lemfb.setJpaVendorAdapter(jpaVendorAdapter());
      lemfb.setPackagesToScan("packages.containing.entity.classes");
      return lemfb;
    }
}
```

为了让 Spring 数据实现 CRUD 操作，我们必须创建一个接口来扩展 [`CrudRepository`](https://web.archive.org/web/20221006163534/https://docs.spring.io/spring-data/commons/docs/current/api/org/springframework/data/repository/CrudRepository.html) 接口:

```java
@Repository
public interface PersonRepository extends CrudRepository<Person, Long> {

}
```

最后，让我们看一个包含 Spring 数据的 select-all 示例:

```java
personRepository.findAll().forEach(person -> {
    // do something with the extracted person
});
```

### 5.2.春季数据/ JDBC

Spring Data JDBC 是 Spring Data 家族的一个有限实现，其主要目标是允许对关系数据库的简单访问。

由于这个原因，它不提供像缓存、脏跟踪、延迟加载和许多其他 JPA 特性。

这次我们需要的 Maven 依赖项是`[spring-data-jdbc](https://web.archive.org/web/20221006163534/https://search.maven.org/classic/#artifactdetails%7Corg.springframework.data%7Cspring-data-jdbc%7C1.1.4.RELEASE%7Cjar)`:

```java
<dependency>
    <groupId>org.springframework.data</groupId>
    <artifactId>spring-data-jdbc</artifactId>
    <version>1.1.4.RELEASE</version>
</dependency>
```

与我们在上一节中为 Spring Data JPA 使用的配置相比，这个配置更轻:

```java
@Configuration
@EnableJdbcRepositories("packages.to.scan")
public class JdbcConfiguration extends AbstractJdbcConfiguration {
    // NamedParameterJdbcOperations is used internally to submit SQL statements to the database
    @Bean
    NamedParameterJdbcOperations operations() {
        return new NamedParameterJdbcTemplate(dataSource());
    }

    @Bean
    PlatformTransactionManager transactionManager() {
        return new DataSourceTransactionManager(dataSource());
    }

    @Bean
    public DataSource dataSource() {
        DriverManagerDataSource dataSource = new DriverManagerDataSource();
        dataSource.setDriverClassName("com.mysql.cj.jdbc.Driver");
        dataSource.setUrl("jdbc:mysql://localhost:3306/test?serverTimezone=UTC");
        dataSource.setUsername("username");
        dataSource.setPassword("password");
        return dataSource;
    }
}
```

**在 Spring 数据 JDBC 的情况下，我们必须定义一个新的`Person`类或者修改现有的类来添加一些 Spring 特有的注释**。

这是因为 Spring 数据 JDBC 将直接处理实体映射，而不是 Hibernate:

```java
import org.springframework.data.annotation.Id;
import org.springframework.data.relational.core.mapping.Column;
import org.springframework.data.relational.core.mapping.Table;

@Table(value = "Person")
public class Person {
    @Id
    Long id;

    @Column(value = "FIRST_NAME")
    String firstName;

    @Column(value = "LAST_NAME")
    String lastName;

    // getters and setters
}
```

有了 Spring 数据 JDBC，我们也可以使用`CrudRepository`接口。因此，声明将与我们在上面的 Spring Data JPA 示例中编写的声明相同。同样，这同样适用于全选示例。

## 6.结论

在本教程中，我们已经看到了从 Java 连接到 MySQL 数据库的几种不同方式。我们从基本的 JDBC 连接开始。然后我们看了常用的 ORM，比如 Hibernate、Mybatis 和 Apache Cayenne。最后，我们看了一下春季数据 JPA 和春季数据 JDBC。

使用 JDBC 或 Hibernate APIs 意味着更多的样板代码。使用健壮的框架，如 Spring Data 或 Mybatis，需要更多的配置，但有很大的优势，因为它们提供了默认的实现和特性，如缓存和延迟加载。