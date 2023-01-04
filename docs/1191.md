# Spring 数据中的 Vavr 支持

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-vavr>

## 1。概述

在这个快速教程中，我们将看看`Spring Data –` 中对`Vavr`的支持，这是在`2.0.0` Spring 构建快照中添加的。

更具体地说，我们将展示一个使用`Vavr` `Option`和`Vavr`集合作为`Spring Data JPA`存储库的返回类型的例子。

## 2。Maven 依赖关系

首先，让我们设置一个`Spring Boot`项目，因为它通过将`spring-boot-parent`依赖项添加到`pom.xml`中，使得配置`Spring Data`更快:

```
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.7.2</version>
    <relativePath />
</parent>
```

显然，我们还需要`vavr`依赖项，以及其他一些用于`Spring Data`和测试的依赖项:

```
<dependency>
    <groupId>io.vavr</groupId>
    <artifactId>vavr</artifactId>
    <version>0.9.0</version>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-jpa</artifactId>
</dependency>
<dependency>
    <groupId>com.h2database</groupId>
    <artifactId>h2</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-test</artifactId>
</dependency>
```

最新版本的 [vavr](https://web.archive.org/web/20221126222759/https://search.maven.org/classic/#search%7Cga%7C1%7Ca%3A%22vavr%22%20AND%20g%3A%22io.vavr%22) 、[spring-boot-starter-data-JPA](https://web.archive.org/web/20221126222759/https://search.maven.org/classic/#search%7Cga%7C1%7Ca%3A%22spring-boot-starter-data-jpa%22)、 [spring-boot-starter-test](https://web.archive.org/web/20221126222759/https://search.maven.org/classic/#search%7Cga%7C1%7Ca%3A%22spring-boot-starter-test%22%20AND%20g%3A%22org.springframework.boot%22) 和 [h2](https://web.archive.org/web/20221126222759/https://search.maven.org/classic/#search%7Cga%7C1%7Ca%3A%22h2%22%20AND%20g%3A%22com.h2database%22) 可以从 Maven Central 下载。

在这个例子中，我们只使用了`Spring Boot`，因为它提供了`Spring Data`自动配置。如果您在一个非`Boot`项目中工作，您可以直接添加`spring-data-commons`依赖项和`Vavr`支持:

```
<dependency>
    <groupId>org.springframework.data</groupId>
    <artifactId>spring-data-commons</artifactId>
    <version>2.0.0.RELEASE</version>
</dependency>
```

## 3。`Spring Data JPA` 储存库与`Vavr`

`Spring Data`现在支持使用`Vavr`的`Option`和`Vavr`集合:`Seq`、`Set`和`Map`作为返回类型来定义存储库查询方法。

首先，让我们创建一个简单的实体类来操作:

```
@Entity
public class User {

    @Id
    @GeneratedValue(strategy = GenerationType.AUTO)
    private long id;

    private String name;

    // standard constructor, getters, setters
}
```

接下来，让我们通过实现`Repository`接口并定义两个查询方法来创建`JPA`存储库:

```
public interface VavrUserRepository extends Repository<User, Long> {

    Option<User> findById(long id);

    Seq<User> findByName(String name);

    User save(User user);
}
```

这里，我们使用`Vavr` `Option`作为返回零个或一个结果的方法，使用`Vavr` `Seq`作为返回多个`User`记录的查询方法。

我们还需要一个主`Spring Boot`类来自动配置`Spring Data`并引导我们的应用程序:

```
@SpringBootApplication
public class Application {
    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }
}
```

由于我们已经添加了`h2`依赖项，`Spring Boot`将使用内存中的`H2`数据库自动配置一个`DataSource`。

## 4。测试`JPA`库

让我们添加一个 JUnit 测试来验证我们的存储库方法:

```
@RunWith(SpringRunner.class)
@SpringBootTest(classes = Application.class)
public class VavrRepositoryIntegrationTest {

    @Autowired
    private VavrUserRepository userRepository;

    @Before
    public void setup() {
        User user1 = new User();
        user1.setName("John");
        User user2 = new User();
        user2.setName("John");

        userRepository.save(user1);
        userRepository.save(user2);
    }

    @Test
    public void whenAddUsers_thenGetUsers() {
        Option<User> user = userRepository.findById(1L);
        assertFalse(user.isEmpty());
        assertTrue(user.get().getName().equals("John"));

        Seq<User> users = userRepository.findByName("John");
        assertEquals(2, users.size());
    }
}
```

在上面的测试中，我们首先向数据库添加两个用户记录，然后调用存储库的查询方法。如您所见，这些方法返回正确的`Vavr`对象。

## 5。结论

在这个简单的例子中，我们展示了如何使用`Vavr`类型定义一个`Spring Data`存储库。

和往常一样，完整的源代码可以在 GitHub 上找到[。](https://web.archive.org/web/20221126222759/https://github.com/eugenp/tutorials/tree/master/vavr-modules/vavr)