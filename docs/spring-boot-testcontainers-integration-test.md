# 使用 Spring Boot 和 Testcontainers 进行数据库集成测试

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-boot-testcontainers-integration-test>

## 1。概述

Spring Data JPA 提供了一种简单的方法来创建数据库查询，并使用嵌入式 H2 数据库测试它们。

但是在某些情况下，**在真实数据库上测试更有利可图，**尤其是当我们使用依赖于提供者的查询时。

在本教程中，我们将演示**如何使用 [Testcontainers](/web/20220926190100/https://www.baeldung.com/docker-test-containers) 对 Spring Data JPA 和 PostgreSQL 数据库进行集成测试。**

在之前的教程中，我们主要使用`@Query`注释创建了一些数据库[查询，现在我们将对其进行测试。](/web/20220926190100/https://www.baeldung.com/spring-data-jpa-query)

## 2。配置

为了在我们的测试中使用 PostgreSQL 数据库，**我们必须添加具有`test`范围**的 [Testcontainers 依赖关系](https://web.archive.org/web/20220926190100/https://search.maven.org/classic/#search%7Cga%7C1%7Cg%3A%22org.testcontainers%22%20AND%20a%3A%22postgresql%22):

```java
<dependency>
    <groupId>org.testcontainers</groupId>
    <artifactId>postgresql</artifactId>
    <version>1.17.3</version>
    <scope>test</scope>
</dependency> 
```

让我们在测试资源目录下创建一个`application.properties`文件，在这个文件中，我们指示 Spring 使用正确的驱动程序类，并在每次测试运行时创建方案:

```java
spring.datasource.driver-class-name=org.testcontainers.jdbc.ContainerDatabaseDriver
spring.jpa.hibernate.ddl-auto=create
```

## 3。单一测试用途

要开始在单个测试类中使用 PostgreSQL 实例，我们必须首先创建一个容器定义，然后使用它的参数来建立连接:

```java
@RunWith(SpringRunner.class)
@SpringBootTest
@ContextConfiguration(initializers = {UserRepositoryTCIntegrationTest.Initializer.class})
public class UserRepositoryTCIntegrationTest extends UserRepositoryCommonIntegrationTests {

    @ClassRule
    public static PostgreSQLContainer postgreSQLContainer = new PostgreSQLContainer("postgres:11.1")
      .withDatabaseName("integration-tests-db")
      .withUsername("sa")
      .withPassword("sa");

    static class Initializer
      implements ApplicationContextInitializer<ConfigurableApplicationContext> {
        public void initialize(ConfigurableApplicationContext configurableApplicationContext) {
            TestPropertyValues.of(
              "spring.datasource.url=" + postgreSQLContainer.getJdbcUrl(),
              "spring.datasource.username=" + postgreSQLContainer.getUsername(),
              "spring.datasource.password=" + postgreSQLContainer.getPassword()
            ).applyTo(configurableApplicationContext.getEnvironment());
        }
    }
}
```

在上面的例子中，在执行测试方法之前，我们使用 JUnit 的`@ClassRule`来建立一个数据库容器**。我们还创建了一个实现`ApplicationContextInitializer.`的静态内部类。最后一步，我们将 `@ContextConfiguration`注释应用到我们的测试类，并将初始化器类作为参数。**

通过执行这三个动作，我们可以在发布 Spring 上下文之前设置连接属性。

现在让我们使用前一篇文章中的两个更新查询:

```java
@Modifying
@Query("update User u set u.status = :status where u.name = :name")
int updateUserSetStatusForName(@Param("status") Integer status, 
  @Param("name") String name);

@Modifying
@Query(value = "UPDATE Users u SET u.status = ? WHERE u.name = ?", 
  nativeQuery = true)
int updateUserSetStatusForNameNative(Integer status, String name);
```

并在配置好的环境中测试它们:

```java
@Test
@Transactional
public void givenUsersInDB_WhenUpdateStatusForNameModifyingQueryAnnotationJPQL_ThenModifyMatchingUsers(){
    insertUsers();
    int updatedUsersSize = userRepository.updateUserSetStatusForName(0, "SAMPLE");
    assertThat(updatedUsersSize).isEqualTo(2);
}

@Test
@Transactional
public void givenUsersInDB_WhenUpdateStatusForNameModifyingQueryAnnotationNative_ThenModifyMatchingUsers(){
    insertUsers();
    int updatedUsersSize = userRepository.updateUserSetStatusForNameNative(0, "SAMPLE");
    assertThat(updatedUsersSize).isEqualTo(2);
}

private void insertUsers() {
    userRepository.save(new User("SAMPLE", "[[email protected]](/web/20220926190100/https://www.baeldung.com/cdn-cgi/l/email-protection)", 1));
    userRepository.save(new User("SAMPLE1", "em[[email protected]](/web/20220926190100/https://www.baeldung.com/cdn-cgi/l/email-protection)", 1));
    userRepository.save(new User("SAMPLE", "[[email protected]](/web/20220926190100/https://www.baeldung.com/cdn-cgi/l/email-protection)", 1));
    userRepository.save(new User("SAMPLE3", "[[email protected]](/web/20220926190100/https://www.baeldung.com/cdn-cgi/l/email-protection)", 1));
    userRepository.flush();
}
```

在上面的场景中，第一个测试以成功结束，但是第二个测试抛出了消息`InvalidDataAccessResourceUsageException`:

```java
Caused by: org.postgresql.util.PSQLException: ERROR: column "u" of relation "users" does not exist
```

如果我们使用 H2 嵌入式数据库运行相同的测试，两个测试都会成功完成，但是 PostgreSQL 不接受 SET 子句中的别名。我们可以通过删除有问题的别名来快速修复查询:

```java
@Modifying
@Query(value = "UPDATE Users u SET status = ? WHERE u.name = ?", 
  nativeQuery = true)
int updateUserSetStatusForNameNative(Integer status, String name);
```

这一次两个测试都成功完成。在这个例子中，**我们使用 Testcontainers 来识别原生查询的一个问题，否则这个问题会在切换到生产中的真实数据库后暴露出来。**我们还应该注意到，使用`JPQL`查询通常更安全，因为 Spring 会根据所使用的数据库提供者正确地翻译它们。

### 3.1.每个测试一个数据库，带配置

到目前为止，在运行测试类中的所有测试之前，我们已经使用 JUnit 4 规则启动了一个数据库实例。最终，这种方法将在每个测试类之前创建一个数据库实例，并在运行每个类中的所有测试之后拆除它。

这种方法在测试实例之间建立了最大的隔离。此外，多次启动数据库的开销会降低测试速度。

除了 JUnit 4 规则方法，**我们可以[修改 JDBC URL](https://web.archive.org/web/20220926190100/https://www.testcontainers.org/modules/databases/jdbc/) 并指示 Testcontainers 为每个测试类**创建一个数据库实例。这种方法不需要我们在测试中编写一些基础设施代码就可以工作。

例如，为了重写上面的例子，我们所要做的就是将它添加到我们的`application.properties`:

```java
spring.datasource.url=jdbc:tc:postgresql:11.1:///integration-tests-db
```

`“tc:”` 将使 Testcontainers 实例化数据库实例，而无需任何代码更改。因此，我们的测试类将会非常简单:

```java
@RunWith(SpringRunner.class)
@SpringBootTest(classes = Application.class)
public class UserRepositoryTCJdbcLiveTest extends UserRepositoryCommon {

    @Test
    @Transactional
    public void givenUsersInDB_WhenUpdateStatusForNameModifyingQueryAnnotationNative_ThenModifyMatchingUsers() {
        // same as above
    }
}
```

如果我们每个测试类有一个数据库实例，这种方法是首选。

## 4。共享数据库实例

在前面的段落中，我们描述了如何在一个单独的测试中使用 Testcontainers。在实际场景中，由于启动时间相对较长，我们希望在多个测试中重用同一个数据库容器。

现在让我们通过扩展`PostgreSQLContainer`并覆盖`start()`和`stop()`方法来创建一个用于数据库容器创建的公共类:

```java
public class BaeldungPostgresqlContainer extends PostgreSQLContainer<BaeldungPostgresqlContainer> {
    private static final String IMAGE_VERSION = "postgres:11.1";
    private static BaeldungPostgresqlContainer container;

    private BaeldungPostgresqlContainer() {
        super(IMAGE_VERSION);
    }

    public static BaeldungPostgresqlContainer getInstance() {
        if (container == null) {
            container = new BaeldungPostgresqlContainer();
        }
        return container;
    }

    @Override
    public void start() {
        super.start();
        System.setProperty("DB_URL", container.getJdbcUrl());
        System.setProperty("DB_USERNAME", container.getUsername());
        System.setProperty("DB_PASSWORD", container.getPassword());
    }

    @Override
    public void stop() {
        //do nothing, JVM handles shut down
    }
}
```

通过将`stop()`方法留空，我们允许 JVM 处理容器关闭。我们还实现了一个简单的单例模式，其中只有第一个测试触发容器启动，每个后续测试都使用现有的实例。在`start()`方法中，我们使用`System#setProperty` 将连接参数设置为环境变量。

我们现在可以将它们放在我们的`application.properties` 文件中:

```java
spring.datasource.url=${DB_URL}
spring.datasource.username=${DB_USERNAME}
spring.datasource.password=${DB_PASSWORD}
```

现在让我们在测试定义中使用我们的实用程序类:

```java
@RunWith(SpringRunner.class)
@SpringBootTest
public class UserRepositoryTCAutoIntegrationTest {

    @ClassRule
    public static PostgreSQLContainer postgreSQLContainer = BaeldungPostgresqlContainer.getInstance();

    // tests
}
```

和前面的例子一样，我们对保存容器定义的字段应用了`the @ClassRule` 注释。这样，在创建 Spring 上下文之前,`DataSource`连接属性就被填充了正确的值。

**我们现在可以使用同一个数据库实例**实现多个测试，只需定义一个用我们的`BaeldungPostgresqlContainer`实用程序类实例化的`@ClassRule`注释字段。

## 5。结论

在本文中，我们展示了使用 Testcontainers 在真实数据库实例上执行测试的方法。

我们查看了单次测试使用的例子，使用了 Spring 的`ApplicationContextInitializer`机制，并实现了一个可重用数据库实例化的类。

我们还展示了 Testcontainers 如何帮助识别跨多个数据库提供者的兼容性问题，特别是对于本地查询。

和往常一样，本文中使用的完整代码可以从 GitHub 上的[处获得。](https://web.archive.org/web/20220926190100/https://github.com/eugenp/tutorials/tree/master/persistence-modules/spring-data-jpa-enterprise)