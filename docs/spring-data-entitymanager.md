# 如何用 Spring 数据访问 EntityManager

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-data-entitymanager>

## 1.概观

我们在处理 Spring 数据应用时，通常不需要直接访问 [`EntityManager`](/web/20220707143816/https://www.baeldung.com/hibernate-entitymanager) 。但是，有时我们可能想要访问它，例如，创建自定义查询或分离实体。

在这个简短的教程中，我们将看到如何通过扩展一个 Spring 数据[库](/web/20220707143816/https://www.baeldung.com/spring-data-repositories)来访问`EntityManager` 。

## 2.使用 Spring 数据访问`EntityManager`

**我们可以通过创建一个定制的存储库来获得`EntityManager`，比如扩展一个内置的`JpaRepository`。**

首先，让我们定义一个 [`Entity`](/web/20220707143816/https://www.baeldung.com/jpa-entities) ，例如，对于我们希望存储在数据库中的用户:

```java
@Entity
public class User {
    @Id
    @GeneratedValue
    private Long id;
    private String name;
    private String email;
    // ...
}
```

**我们没有直接接触到****`EntityManager``JpaRepository.`** 因此，我们需要自己创造。

让我们用自定义查找方法创建一个:

```java
public interface CustomUserRepository {
    User customFindMethod(Long id);
}
```

**使用`@PeristenceContext`，我们可以在实现类**中注入`EntityManager`:

```java
public class CustomUserRepositoryImpl implements CustomUserRepository {

    @PersistenceContext
    private EntityManager entityManager;

    @Override
    public User customFindMethod(Long id) {
        return (User) entityManager.createQuery("FROM User u WHERE u.id = :id")
          .setParameter("id", id)
          .getSingleResult();
    }
}
```

**同样，我们可以使用`@PersistenceUnit`注释**，在这种情况下，我们将访问`EntityManagerFactory`，并从中访问`EntityManager`。

最后，**让我们创建一个扩展了`JpaRepository` 和 `CustomRepository`** 的`Repository`:

```java
@Repository
public interface UserRepository extends JpaRepository<User, Long>, CustomUserRepository {
}
```

此外，我们可以制作一个`[Spring Boot](/web/20220707143816/https://www.baeldung.com/spring-boot)`应用程序并进行测试，以检查一切是否正常运行:

```java
@SpringBootTest(classes = CustomRepositoryApplication.class)
class CustomRepositoryUnitTest {

    @Autowired
    private UserRepository userRepository;

    @Test
    public void givenCustomRepository_whenInvokeCustomFindMethod_thenEntityIsFound() {
        User user = new User();
        user.setEmail("[[email protected]](/web/20220707143816/https://www.baeldung.com/cdn-cgi/l/email-protection)");
        user.setName("userName");

        User persistedUser = userRepository.save(user);

        assertEquals(persistedUser, userRepository.customFindMethod(user.getId()));
    }
}
```

## 3.结论

在本文中，我们查看了一个在 Spring 数据应用程序中访问`EntityManager` 的快速示例。

我们可以访问自定义存储库中的`EntityManager`,并通过扩展其功能来使用我们的 Spring 数据存储库。

和往常一样，这些例子的代码可以在 GitHub 的[上找到。](https://web.archive.org/web/20220707143816/https://github.com/eugenp/tutorials/tree/master/persistence-modules/spring-data-jpa-repo-2)