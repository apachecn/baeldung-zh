# 使用 Spring 数据仓库进行不区分大小写的查询

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-data-case-insensitive-queries>

## 1.概观

默认情况下，Spring Data JPA 查询区分大小写。换句话说，字段值比较区分大小写。

在本教程中，我们将探索如何在 Spring 数据 JPA 存储库中快速创建一个不区分大小写的查询。

## 2.属国

首先，让我们确保我们的`pom.xml`中有 [Spring 数据](/web/20220627173802/https://www.baeldung.com/the-persistence-layer-with-spring-data-jpa)和 [H2](/web/20220627173802/https://www.baeldung.com/java-in-memory-databases) 数据库依赖项:

```
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-jpa</artifactId>
    <version>2.6.1</version>
</dependency>
<dependency>
    <groupId>com.h2database</groupId>
    <artifactId>h2</artifactId>
    <scope>runtime</scope>
    <version>1.4.199</version>
</dependency>
```

这些的最新版本可以在 Maven Central 上获得。

## 3.初始设置

假设我们有一个带有`id, firstName`和`lastName`属性的`Passenger`实体:

```
@Entity
class Passenger {

    @Id
    @GeneratedValue
    @Column(nullable = false)
    private Long id;

    @Basic(optional = false)
    @Column(nullable = false)
    private String firstName;

    @Basic(optional = false)
    @Column(nullable = false)
    private String lastName;

    // constructor, static factory, getters, setters
}
```

同样，让我们通过用一些样本数据填充数据库来准备我们的测试类:

```
@DataJpaTest
@RunWith(SpringRunner.class)
public class PassengerRepositoryIntegrationTest {

    @PersistenceContext
    private EntityManager entityManager;
    @Autowired
    private PassengerRepository repository;

    @Before
    public void before() {
        entityManager.persist(Passenger.from("Jill", "Smith"));
        entityManager.persist(Passenger.from("Eve", "Jackson"));
        entityManager.persist(Passenger.from("Fred", "Bloggs"));
        entityManager.persist(Passenger.from("Ricki", "Bobbie"));
        entityManager.persist(Passenger.from("Siya", "Kolisi"));
    }

    //...
}
```

## 4.`IgnoreCase`对于不区分大小写的查询

现在，假设我们想要执行一个不区分大小写的搜索来查找给定`firstName.`的所有乘客

为此，我们将我们的`PassengerRepository` 定义为:

```
@Repository
public interface PassengerRepository extends JpaRepository<Passenger, Long> {
    List<Passenger> findByFirstNameIgnoreCase(String firstName);
}
```

这里，**`IgnoreCase`关键字确保查询匹配不区分大小写。**

我们还可以借助 JUnit 测试来验证这一点:

```
@Test
public void givenPassengers_whenMatchingIgnoreCase_thenExpectedReturned() {
    Passenger jill = Passenger.from("Jill", "Smith");
    Passenger eve = Passenger.from("Eve", "Jackson");
    Passenger fred = Passenger.from("Fred", "Bloggs");
    Passenger siya = Passenger.from("Siya", "Kolisi");
    Passenger ricki = Passenger.from("Ricki", "Bobbie");

    List<Passenger> passengers = repository.findByFirstNameIgnoreCase("FrED");

    assertThat(passengers, contains(fred));
    assertThat(passengers, not(contains(eve)));
    assertThat(passengers, not(contains(siya)));
    assertThat(passengers, not(contains(jill)));
    assertThat(passengers, not(contains(ricki)));
}
```

尽管已经将 `“FrED”`作为参数传递，但我们返回的列表`passengers`包含一个`Passenger`，其中`firstName`显然是`“Fred”.`，在`IgnoreCase`关键字的帮助下，我们实现了一个不区分大小写的匹配。

## 5.结论

在这个快速教程中，我们学习了如何在 Spring 数据存储库中创建不区分大小写的查询。

最后，GitHub 上的[提供了代码示例。](https://web.archive.org/web/20220627173802/https://github.com/eugenp/tutorials/tree/master/persistence-modules/spring-data-jpa-repo)