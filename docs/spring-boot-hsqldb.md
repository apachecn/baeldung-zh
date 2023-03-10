# 将 Spring Boot 与 HSQLDB 集成

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-boot-hsqldb>

## 1。概述

Spring Boot 让使用不同的数据库系统变得非常容易，没有手动依赖管理的麻烦。

更具体地说， [Spring Data JPA](/web/20220909204051/https://www.baeldung.com/the-persistence-layer-with-spring-data-jpa) starter 提供了与几个`[DataSource](https://web.archive.org/web/20220909204051/https://docs.oracle.com/en/java/javase/11/docs/api/java.sql/javax/sql/DataSource.html)`实现无缝集成所需的所有功能。

在本教程中，**我们将学习如何将 Spring Boot 与 [HSQLDB](https://web.archive.org/web/20220909204051/http://hsqldb.org/)** 集成。

## 2。美芬依赖

为了演示将 Spring Boot 与 HSQLDB 集成是多么容易，**我们将创建一个简单的 JPA 存储库层，它使用内存中的 HSQLDB 数据库**对客户实体执行 CRUD 操作。

这里是 [Spring Boot 启动器](/web/20220909204051/https://www.baeldung.com/spring-boot-starters)，我们将使用它来启动并运行我们的示例存储库层:

```java
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-jpa</artifactId>
    <version>2.2.2.RELEASE</version>
</dependency>
<dependency>
    <groupId>org.hsqldb</groupId>
    <artifactId>hsqldb</artifactId>
    <version>2.4.0</version>
    <scope>runtime</scope>
</dependency> 
```

注意，我们还包括了 [HSQLDB](https://web.archive.org/web/20220909204051/https://search.maven.org/search?q=g:org.hsqldb%20AND%20a:hsqldb) 依赖项。如果没有它，Spring Boot 将尝试通过 [HikariCP](/web/20220909204051/https://www.baeldung.com/hikaricp) 为我们自动配置一个`DataSource` bean 和一个 JDBC 连接池。

因此，**如果我们不在`pom.xml`文件中指定一个有效的`DataSource`依赖项，我们将得到一个构建失败**。

另外，我们一定要在 Maven Central 上查看一下`[spring-boot-starter-data-jpa](https://web.archive.org/web/20220909204051/https://search.maven.org/search?q=g:org.springframework.boot%20AND%20a:spring-boot-starter-data-jpa)`的最新版本。

## 3。连接到 HSQLDB 数据库

为了练习我们的演示存储库层，我们将使用内存中的数据库。然而，也可以使用基于文件的数据库。我们将在下面的小节中探索这些方法。

### 3.1。运行外部 HSQLDB 服务器

让我们看看如何运行外部 HSQLDB 服务器并创建基于文件的数据库。总的来说，安装 HSQLDB 和运行服务器很简单。

以下是我们应该遵循的步骤:

*   首先，我们将[下载 HSQLDB](https://web.archive.org/web/20220909204051/https://sourceforge.net/projects/hsqldb/files/latest/download) 并将其解压缩到一个文件夹中
*   由于 HSQLDB 没有提供现成的默认数据库，为了举例，我们将创建一个名为`“testdb”`的数据库
*   我们将启动一个命令提示符并导航到 HSQLDB `data`文件夹
*   在`data`文件夹中，我们将运行以下命令:

    ```java
    java -cp ../lib/hsqldb.jar org.hsqldb.server.Server --database.0 file.testdb --dbname0.testdb
    ```

*   上面的命令将启动 HSQLDB 服务器并创建我们的数据库，其源文件将存储在`data`文件夹中
*   我们可以通过转到`data`文件夹来确保数据库已经被实际创建，该文件夹应该包含一组名为`“testdb.lck”`、`“testdb.log”`、`“testdb.properties”`和`“testdb.script”`的文件(文件的数量根据我们正在创建的数据库的类型而有所不同)

一旦建立了数据库，我们需要创建一个到它的连接。

**要在 Windows** 上做到这一点，让我们转到数据库`bin`文件夹并运行`runManagerSwing.bat`文件。这将打开 HSQLDB 数据库管理器的初始屏幕，我们可以在其中输入连接凭据:

*   **类型:** HSQL 数据库引擎
*   **网址:** `jdbc:hsqldb:hsql://localhost/testdb`
*   **用户:**“SA”(系统管理员)
*   **密码:**将该字段留空

**在 Linux/Unix/Mac** 上，我们可以使用 NetBeans、Eclipse 或 IntelliJ IDEA 通过 IDE 的可视化工具创建数据库连接，使用相同的凭证。

在这些工具中，通过在数据库管理器或 IDE 中执行 SQL 脚本来创建数据库表是非常简单的。

连接后，我们可以创建一个`customers`表:

```java
CREATE TABLE customers (
   id INT  NOT NULL,
   name VARCHAR (45),
   email VARCHAR (45),      
   PRIMARY KEY (ID)
); 
```

只需几个简单的步骤，我们就创建了一个基于文件的 HSQLDB 数据库，其中包含一个`customers`表。

### 3.2。`application.properties`文件

如果我们希望从 Spring Boot 连接到以前的基于文件的数据库，下面是我们应该包含在`application.properties`文件中的设置:

```java
spring.datasource.driver-class-name=org.hsqldb.jdbc.JDBCDriver 
spring.datasource.url=jdbc:hsqldb:hsql://localhost/testdb 
spring.datasource.username=sa 
spring.datasource.password= 
spring.jpa.hibernate.ddl-auto=update 
```

或者，如果我们使用内存数据库，我们应该使用这些:

```java
spring.datasource.driver-class-name=org.hsqldb.jdbc.JDBCDriver
spring.datasource.url=jdbc:hsqldb:mem:testdb;DB_CLOSE_DELAY=-1
spring.datasource.username=sa
spring.datasource.password=
spring.jpa.hibernate.ddl-auto=create 
```

请注意追加到数据库 URL 末尾的`DB_CLOSE_DELAY=-1`参数。当使用内存数据库时，我们需要指定这个，**,这样 JPA 实现(Hibernate)就不会在应用程序运行时关闭数据库**。

## 4。`Customer`实体

数据库连接设置已经完成，接下来我们需要定义我们的`Customer`实体:

```java
@Entity
@Table(name = "customers")
public class Customer {

    @Id
    @GeneratedValue(strategy = GenerationType.AUTO)
    private long id;

    private String name;

    private String email;

    // standard constructors / setters / getters / toString
} 
```

## 5。`Customer`储存库

此外，我们需要实现一个瘦持久层，这允许我们在我们的`Customer` JPA 实体上拥有基本的 CRUD 功能。

我们可以通过扩展`[CrudRepository](https://web.archive.org/web/20220909204051/https://docs.spring.io/spring-data/commons/docs/current/api/org/springframework/data/repository/CrudRepository.html)`接口轻松实现这一层:

```java
@Repository
public interface CustomerRepository extends CrudRepository<Customer, Long> {}
```

## 6。测试`Customer`库

最后，我们应该确保 Spring Boot 实际上可以连接到 HSQLDB。我们可以通过测试存储库层来轻松实现这一点。

让我们开始测试存储库的`findById()`和`findAll()`方法:

```java
@RunWith(SpringRunner.class)
@SpringBootTest
public class CustomerRepositoryTest {

    @Autowired
    private CustomerRepository customerRepository;

    @Test
    public void whenFindingCustomerById_thenCorrect() {
        customerRepository.save(new Customer("John", "[[email protected]](/web/20220909204051/https://www.baeldung.com/cdn-cgi/l/email-protection)"));
        assertThat(customerRepository.findById(1L)).isInstanceOf(Optional.class);
    }

    @Test
    public void whenFindingAllCustomers_thenCorrect() {
        customerRepository.save(new Customer("John", "[[email protected]](/web/20220909204051/https://www.baeldung.com/cdn-cgi/l/email-protection)"));
        customerRepository.save(new Customer("Julie", "[[email protected]](/web/20220909204051/https://www.baeldung.com/cdn-cgi/l/email-protection)"));
        assertThat(customerRepository.findAll()).isInstanceOf(List.class);
    }
} 
```

最后，我们来测试一下`save()`方法:

```java
@Test
public void whenSavingCustomer_thenCorrect() {
    customerRepository.save(new Customer("Bob", "[[email protected]](/web/20220909204051/https://www.baeldung.com/cdn-cgi/l/email-protection)"));
    Customer customer = customerRepository.findById(1L).orElseGet(() 
      -> new Customer("john", "[[email protected]](/web/20220909204051/https://www.baeldung.com/cdn-cgi/l/email-protection)"));
    assertThat(customer.getName()).isEqualTo("Bob");
}
```

## 7。结论

在本文中，**我们学习了如何将 Spring Boot 与 HSQLDB 集成，**以及如何在基本 JPA 存储库层的开发中使用基于文件的或内存中的数据库。

像往常一样，本文中展示的所有代码示例都可以在 [GitHub](https://web.archive.org/web/20220909204051/https://github.com/eugenp/tutorials/tree/master/persistence-modules/spring-boot-persistence-2) 上获得。