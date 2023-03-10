# spring Data JPA——在所有存储库中添加一个方法

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-data-jpa-method-in-all-repositories>

 ![announcement - icon](img/e987061484112a9ac3655b4f8f1d178b.png)

JPA can behave very differently depending on the exact circumstances under which it is used. Code that works in our local environment or in staging performs very poorly (or even flat out fails) when thrown against real-scale databases in production environments.

Debugging these JPA issues in production is pretty difficult - existing APMs don’t provide enough granular insights at the code level, and tracking every single place someone queried entities one by one instead of in bulk can be a grueling, time-consuming task.

**Lightrun** is a new approach to debugging in production. Using Lightrun’s Logs and Snapshots, you can now get debugger-level granularity in production without opening inbound ports, redeploying, restarting, or even stropping the running application.

In addition, instrumenting Lightrun Metrics at runtime allows you to track down persistence issues securely and in real-time. Want to see it in action? **Check out our 2-minute tutorial** for debugging JPA performance issues in production using Lightrun:

[>> Debugging Spring Persistence and JPA Issues Using Lightrun](/web/20220526055533/https://www.baeldung.com/lightrun-n-jpa)

## 1。概述

Spring Data 仅仅通过定义存储库接口，就使得处理实体的过程变得更加容易。这些附带了一组预定义的方法，并允许在每个接口中添加自定义方法。

然而，如果我们想要添加一个在所有存储库中都可用的定制方法，这个过程就有点复杂了。所以，这就是我们在这里用 Spring Data JPA 探索的。

有关配置和使用 Spring 数据 JPA 的更多信息，请查看我们以前的文章:[使用 Spring 4 进行 Hibernate 的指南](/web/20220526055533/https://www.baeldung.com/the-persistence-layer-with-spring-and-jpa)和[Spring 数据 JPA 简介](/web/20220526055533/https://www.baeldung.com/the-persistence-layer-with-spring-data-jpa)。

## 2。定义基本存储库接口

首先，我们必须创建一个新的接口来声明我们的自定义方法:

```java
@NoRepositoryBean
public interface ExtendedRepository<T, ID extends Serializable> 
  extends JpaRepository<T, ID> {

    public List<T> findByAttributeContainsText(String attributeName, String text);
}
```

我们的接口扩展了`JpaRepository`接口，因此我们将从所有标准行为中受益。

**你还会注意到我们添加了`@NoRepositoryBean`注释。这是必要的，因为否则，默认的 Spring 行为是为`Repository.`** 的所有子接口创建一个实现

在这里，我们希望提供应该使用的实现，因为这只是一个由实际的特定于实体的 DAO 接口扩展的接口。

## 3。实现一个基类

接下来，我们将提供我们对`ExtendedRepository`接口的实现:

```java
public class ExtendedRepositoryImpl<T, ID extends Serializable>
  extends SimpleJpaRepository<T, ID> implements ExtendedRepository<T, ID> {

    private EntityManager entityManager;

    public ExtendedRepositoryImpl(JpaEntityInformation<T, ?> 
      entityInformation, EntityManager entityManager) {
        super(entityInformation, entityManager);
        this.entityManager = entityManager;
    }

    // ...
}
```

**这个类扩展了`SimpleJpaRepository`类，这是 Spring 用来为存储库接口提供实现的默认类。**

这要求我们创建一个带有`JpaEntityInformation`和`EntityManager`参数的构造函数，它从父类调用构造函数。

我们还需要在自定义方法中使用`EntityManager`属性。

此外，我们必须实现从`ExtendedRepository`接口继承的自定义方法:

```java
@Transactional
public List<T> findByAttributeContainsText(String attributeName, String text) {
    CriteriaBuilder builder = entityManager.getCriteriaBuilder();
    CriteriaQuery<T> cQuery = builder.createQuery(getDomainClass());
    Root<T> root = cQuery.from(getDomainClass());
    cQuery
      .select(root)
      .where(builder
        .like(root.<String>get(attributeName), "%" + text + "%"));
    TypedQuery<T> query = entityManager.createQuery(cQuery);
    return query.getResultList();
}
```

这里，`findByAttributeContainsText()`方法搜索 T 类型的所有对象，这些对象具有包含作为参数给出的`String`值的特定属性。

## 4。JPA 配置

为了告诉 Spring 使用我们的定制类而不是默认类来构建存储库实现，**我们可以使用`repositoryBaseClass`属性**:

```java
@Configuration
@EnableJpaRepositories(basePackages = "org.baeldung.persistence.dao", 
  repositoryBaseClass = ExtendedRepositoryImpl.class)
public class StudentJPAH2Config {
    // additional JPA Configuration
}
```

## 5。创建实体存储库

接下来，让我们看看如何使用我们的新界面。

首先，让我们添加一个简单的`Student`实体:

```java
@Entity
public class Student {

    @Id
    private long id;
    private String name;

    // standard constructor, getters, setters
}
```

然后，我们可以为扩展了`ExtendedRepository`接口的`Student`实体创建一个 DAO:

```java
public interface ExtendedStudentRepository extends ExtendedRepository<Student, Long> {
}
```

就是这样！现在我们的实现将拥有自定义的`findByAttributeContainsText()`方法。

同样，我们通过扩展`ExtendedRepository`接口定义的任何接口都会有相同的方法。

## 6。测试存储库

让我们创建一个`JUnit`测试来展示这个定制方法的作用:

```java
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(classes = { StudentJPAH2Config.class })
public class ExtendedStudentRepositoryIntegrationTest {

    @Resource
    private ExtendedStudentRepository extendedStudentRepository;

    @Before
    public void setup() {
        Student student = new Student(1, "john");
        extendedStudentRepository.save(student);
        Student student2 = new Student(2, "johnson");
        extendedStudentRepository.save(student2);
        Student student3 = new Student(3, "tom");
        extendedStudentRepository.save(student3);
    }

    @Test
    public void givenStudents_whenFindByName_thenOk(){
        List<Student> students 
          = extendedStudentRepository.findByAttributeContainsText("name", "john");

        assertEquals("size incorrect", 2, students.size());        
    }
}
```

该测试首先使用`extendedStudentRepository` bean 来创建 3 个学生记录。然后，调用`findByAttributeContains()`方法来查找姓名中包含文本“john”的所有学生。

`ExtendedStudentRepository`类可以使用像`save()`这样的标准方法和我们添加的自定义方法。

## 7。结论

在这篇简短的文章中，我们展示了如何向 Spring Data JPA 中的所有存储库添加自定义方法。

示例的完整源代码可以在 GitHub 的[中找到。](https://web.archive.org/web/20220526055533/https://github.com/eugenp/tutorials/tree/master/persistence-modules/spring-data-jpa-repo)