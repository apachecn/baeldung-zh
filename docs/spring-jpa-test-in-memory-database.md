# 使用内存数据库的自包含测试

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-jpa-test-in-memory-database>

## 1。概述

在本教程中，我们将**创建一个简单的 Spring 应用程序，它依赖于一个内存数据库来测试**。

对于标准配置文件，应用程序将有一个独立的 MySQL 数据库配置，这需要安装和运行 MySQL 服务器，并设置适当的用户和数据库。

为了使应用程序的测试更容易，我们将放弃 MySQL 所需的额外配置，而是使用一个`H2`内存数据库来运行 JUnit 测试。

## 2。Maven 依赖关系

对于开发，我们需要以下依赖项:

```java
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-test</artifactId>
    <version>5.1.5.RELEASE</version>
</dependency>
<dependency>
    <groupId>org.springframework.data</groupId>
    <artifactId>spring-data-jpa</artifactId>
    <version>2.1.5.RELEASE</version>
</dependency>
<dependency>
    <groupId>com.h2database</groupId>
    <artifactId>h2</artifactId>
    <version>1.4.194</version>
</dependency>
<dependency>
    <groupId>org.hibernate</groupId>
    <artifactId>hibernate-core</artifactId>
    <version>5.2.17.Final</version>
</dependency>
```

最新版本的 [spring-test](https://web.archive.org/web/20220627173048/https://search.maven.org/classic/#search%7Cga%7C1%7Ca%3A%22spring-test%22%20AND%20g%3A%22org.springframework%22) 、 [spring-data-jpa](https://web.archive.org/web/20220627173048/https://search.maven.org/classic/#search%7Cga%7C1%7Ca%3A%22spring-data-jpa%22) 、 [h2](https://web.archive.org/web/20220627173048/https://search.maven.org/classic/#search%7Cga%7C1%7Ca%3A%22h2%22%20AND%20g%3A%22com.h2database%22) 和 [hibernate-core](https://web.archive.org/web/20220627173048/https://search.maven.org/classic/#search%7Cga%7C1%7Ca%3A%22hibernate-core%22%20AND%20g%3A%22org.hibernate%22) 可以从 Maven Central 下载。

## 3。数据模型和存储库

让我们创建一个简单的`Student` 类，它将被标记为一个实体:

```java
@Entity
public class Student {

    @Id
    private long id;

    private String name;

    // standard constructor, getters, setters
}
```

接下来，让我们基于 Spring 数据 JPA 创建一个存储库接口:

```java
public interface StudentRepository extends JpaRepository<Student, Long> {
}
```

这将使 Spring 能够创建对操纵*学生*对象的支持。

## 4。单独的财产来源

为了允许标准模式和测试模式使用不同的数据库配置，我们可以从一个文件中读取数据库属性，该文件的位置根据应用程序的运行模式而不同。

**对于正常模式，属性文件将驻留在 `src/main/resources`中，对于测试方法，我们将使用`src/test/resources`文件夹**中的属性文件。

运行测试时，应用程序将首先在`src/test/resources`文件夹中查找文件。如果在这个位置没有找到文件，那么它将使用在`src/main/resources`文件夹中定义的文件。如果文件存在于`test`路径，那么它将覆盖来自`main`路径的文件。

### 4.1。定义属性文件

让我们在`src/main/resources`文件夹中创建一个`persistence-student.properties`文件，它定义了 MySQL 数据源的属性:

```java
dbc.driverClassName=com.mysql.jdbc.Driver
jdbc.url=jdbc:mysql://localhost:3306/myDb
jdbc.user=tutorialuser
jdbc.pass=tutorialpass

hibernate.dialect=org.hibernate.dialect.MySQL5Dialect
hibernate.hbm2ddl.auto=create-drop
```

在上述配置的情况下，我们需要创建`myDb`数据库并设置`tutorialuser/tutorialpass`用户。

因为我们想要使用内存数据库进行测试，所以我们将在`src/test/resources`文件夹中创建一个具有相同名称的类似文件，包含具有相同关键字和`H2`数据库特定值的属性:

```java
jdbc.driverClassName=org.h2.Driver
jdbc.url=jdbc:h2:mem:myDb;DB_CLOSE_DELAY=-1

hibernate.dialect=org.hibernate.dialect.H2Dialect
hibernate.hbm2ddl.auto=create
```

我们已经将`H2`数据库配置为驻留在内存中并自动创建，然后在 JVM 退出时关闭并删除。

### 4.2。JPA 配置

让我们创建一个`@Configuration`类，它搜索名为`persistence-student.properties`的文件作为属性源，并使用其中定义的数据库属性创建一个`DataSource`:

```java
@Configuration
@EnableJpaRepositories(basePackages = "com.baeldung.persistence.dao")
@PropertySource("persistence-student.properties")
@EnableTransactionManagement
public class StudentJpaConfig {

    @Autowired
    private Environment env;

    @Bean
    public DataSource dataSource() {
        DriverManagerDataSource dataSource = new DriverManagerDataSource();
        dataSource.setDriverClassName(env.getProperty("jdbc.driverClassName"));
        dataSource.setUrl(env.getProperty("jdbc.url"));
        dataSource.setUsername(env.getProperty("jdbc.user"));
        dataSource.setPassword(env.getProperty("jdbc.pass"));

        return dataSource;
    }

    // configure entityManagerFactory

    // configure transactionManager

    // configure additional Hibernate Properties
}
```

## 5。创建一个 JUnit 测试

让我们基于上述配置编写一个简单的 JUnit 测试，它使用`StudentRepository` 来保存和检索一个`Student` 实体:

```java
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(
  classes = { StudentJpaConfig.class }, 
  loader = AnnotationConfigContextLoader.class)
@Transactional
public class InMemoryDBTest {

    @Resource
    private StudentRepository studentRepository;

    @Test
    public void givenStudent_whenSave_thenGetOk() {
        Student student = new Student(1, "john");
        studentRepository.save(student);

        Student student2 = studentRepository.findOne(1);
        assertEquals("john", student2.getName());
    }
}
```

**我们的测试将以完全独立的方式运行** —它将创建一个内存中的`H2`数据库，执行语句，然后关闭连接并删除数据库，正如我们在日志中看到的:

```java
INFO: HHH000400: Using dialect: org.hibernate.dialect.H2Dialect
Hibernate: drop table Student if exists
Hibernate: create table Student (id bigint not null, name varchar(255), primary key (id))
Mar 24, 2017 12:41:51 PM org.hibernate.tool.schema.internal.SchemaCreatorImpl applyImportSources
INFO: HHH000476: Executing import script 'org.hiber[[email protected]](/web/20220627173048/https://www.baeldung.com/cdn-cgi/l/email-protection)1b8f9e2'
Hibernate: select student0_.id as id1_0_0_, student0_.name as name2_0_0_ from Student student0_ where student0_.id=?
Hibernate: drop table Student if exists
```

## 6。结论

在这个简单的例子中，我们展示了如何使用内存数据库运行自包含测试。

和往常一样，完整的源代码可以在 GitHub 上找到[。](https://web.archive.org/web/20220627173048/https://github.com/eugenp/tutorials/tree/master/persistence-modules/spring-jpa)