# 春季数据 JPA 预测

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-data-jpa-projections>

## 1。概述

当使用 [Spring Data JPA](/web/20220628160318/https://www.baeldung.com/the-persistence-layer-with-spring-and-jpa) 实现持久层时，存储库通常会返回根类的一个或多个实例。然而，通常情况下，我们并不需要返回对象的所有属性。

在这种情况下，我们可能希望将数据作为定制类型的对象进行检索。这些类型反映了根类的局部视图，只包含我们关心的属性。这就是投影派上用场的地方。

## 2。初始设置

第一步是设置项目并填充数据库。

### 2.1。Maven 依赖关系

关于依赖性，请查看本教程的[第 2 节。](/web/20220628160318/https://www.baeldung.com/spring-data-case-insensitive-queries)

### 2.2。实体类别

让我们定义两个实体类:

```java
@Entity
public class Address {

    @Id
    private Long id;

    @OneToOne
    private Person person;

    private String state;

    private String city;

    private String street;

    private String zipCode;

    // getters and setters
}
```

并且:

```java
@Entity
public class Person {

    @Id
    private Long id;

    private String firstName;

    private String lastName;

    @OneToOne(mappedBy = "person")
    private Address address;

    // getters and setters
}
```

`Person`和`Address`实体之间是双向一对一的关系；`Address`是拥有方，`Person`是逆方。

注意，在本教程中，我们使用一个嵌入式数据库，H2。

配置嵌入式数据库时，Spring Boot 会自动为我们定义的实体生成底层表格。

### 2.3。SQL 脚本

我们将使用`projection-insert-data.sql`脚本来普及两个支持表:

```java
INSERT INTO person(id,first_name,last_name) VALUES (1,'John','Doe');
INSERT INTO address(id,person_id,state,city,street,zip_code) 
  VALUES (1,1,'CA', 'Los Angeles', 'Standford Ave', '90001');
```

为了在每次测试运行后清理数据库，我们可以使用另一个脚本，`projection-clean-up-data.sql`:

```java
DELETE FROM address;
DELETE FROM person;
```

### 2.4.测试类

然后，为了确认投影产生正确的数据，我们需要一个测试类:

```java
@DataJpaTest
@RunWith(SpringRunner.class)
@Sql(scripts = "/projection-insert-data.sql")
@Sql(scripts = "/projection-clean-up-data.sql", executionPhase = AFTER_TEST_METHOD)
public class JpaProjectionIntegrationTest {
    // injected fields and test methods
}
```

有了给定的注释， **Spring Boot 创建数据库，注入依赖项，并在每个测试方法执行之前和之后填充和清理表格。**

## 3。基于界面的投影

当投影一个实体时，依赖接口是很自然的，因为我们不需要提供实现。

### 3.1。封闭投影

回头看看`Address`类，我们可以看到**它有许多属性，但并不是所有的属性都有用。**例如，有时一个邮政编码就足以表示一个地址。

让我们为`Address`类声明一个投影接口:

```java
public interface AddressView {
    String getZipCode();
}
```

然后我们将在存储库接口中使用它:

```java
public interface AddressRepository extends Repository<Address, Long> {
    List<AddressView> getAddressByState(String state);
}
```

很容易看出，使用投影接口定义存储库方法与使用实体类非常相似。

唯一的区别是**投影接口，而不是实体类，被用作返回集合中的元素类型。**

让我们快速测试一下`Address`投影:

```java
@Autowired
private AddressRepository addressRepository;

@Test
public void whenUsingClosedProjections_thenViewWithRequiredPropertiesIsReturned() {
    AddressView addressView = addressRepository.getAddressByState("CA").get(0);
    assertThat(addressView.getZipCode()).isEqualTo("90001");
    // ...
}
```

在幕后， **Spring 为每个实体对象创建一个投影接口的代理实例，所有对代理的调用都被转发到那个对象。**

我们可以递归地使用投影。例如，这里有一个`Person`类的投影接口:

```java
public interface PersonView {
    String getFirstName();

    String getLastName();
}
```

现在我们将添加一个返回类型为`PersonView,`的嵌套投影方法，在`Address`投影中:

```java
public interface AddressView {
    // ...
    PersonView getPerson();
}
```

注意返回嵌套投影的方法必须与返回相关实体的根类中的方法同名。

我们将通过在刚刚编写的测试方法中添加一些语句来验证嵌套投影:

```java
// ...
PersonView personView = addressView.getPerson();
assertThat(personView.getFirstName()).isEqualTo("John");
assertThat(personView.getLastName()).isEqualTo("Doe");
```

注意**递归投影只有在我们从拥有侧遍历到相反侧时才有效。**如果我们反过来做，嵌套投影将被设置为`null`。

### 3.2。开放式投影

到目前为止，我们已经经历了封闭投影，它表示方法与实体属性名称完全匹配的投影接口。

还有另一种基于界面的投影，开放投影。这些投影使我们能够用不匹配的名字和运行时计算的返回值来定义接口方法。

让我们回到`Person`投影界面，添加一个新方法:

```java
public interface PersonView {
    // ...

    @Value("#{target.firstName + ' ' + target.lastName}")
    String getFullName();
}
```

`@Value`注释的参数是一个 SpEL 表达式，其中`target`指示符表示支持实体对象。

现在我们将定义另一个存储库接口:

```java
public interface PersonRepository extends Repository<Person, Long> {
    PersonView findByLastName(String lastName);
}
```

为了简单起见，我们只返回一个投影对象，而不是一个集合。

该测试证实了开放预测的预期效果:

```java
@Autowired
private PersonRepository personRepository;

@Test 
public void whenUsingOpenProjections_thenViewWithRequiredPropertiesIsReturned() {
    PersonView personView = personRepository.findByLastName("Doe");

    assertThat(personView.getFullName()).isEqualTo("John Doe");
}
```

不过，开放式投影确实有一个缺点；Spring Data 不能优化查询执行，因为它事先不知道将使用哪些属性。因此，当封闭投影不能处理我们的需求时，我们应该只使用开放投影。

## 4。基于类别的预测

我们可以定义自己的投影类，而不是使用 Spring Data 从投影接口创建的代理。

例如，这里有一个用于`Person`实体的投影类:

```java
public class PersonDto {
    private String firstName;
    private String lastName;

    public PersonDto(String firstName, String lastName) {
        this.firstName = firstName;
        this.lastName = lastName;
    }

    // getters, equals and hashCode
}
```

对于与存储库接口协同工作的投影类，其构造函数的参数名必须与根实体类的属性相匹配。

我们还必须定义`equals`和`hashCode`实现；它们允许 Spring 数据处理集合中的投影对象。

现在让我们向`Person`存储库中添加一个方法:

```java
public interface PersonRepository extends Repository<Person, Long> {
    // ...

    PersonDto findByFirstName(String firstName);
}
```

这个测试验证了我们基于类的投影:

```java
@Test
public void whenUsingClassBasedProjections_thenDtoWithRequiredPropertiesIsReturned() {
    PersonDto personDto = personRepository.findByFirstName("John");

    assertThat(personDto.getFirstName()).isEqualTo("John");
    assertThat(personDto.getLastName()).isEqualTo("Doe");
}
```

注意，在基于类的方法中，我们不能使用嵌套投影。

## 5。动态预测

一个实体类可能有许多投影。在某些情况下，我们可能使用某种类型，但在其他情况下，我们可能需要另一种类型。有时候，我们也需要使用实体类本身。

仅仅为了支持多种返回类型而定义单独的存储库接口或方法是很麻烦的。为了解决这个问题，Spring Data 提供了一个更好的解决方案，动态预测。

**我们可以通过用`Class`参数**声明一个存储库方法来应用动态投影

```java
public interface PersonRepository extends Repository<Person, Long> {
    // ...

    <T> T findByLastName(String lastName, Class<T> type);
}
```

通过将投影类型或实体类传递给这样的方法，我们可以检索所需类型的对象:

```java
@Test
public void whenUsingDynamicProjections_thenObjectWithRequiredPropertiesIsReturned() {
    Person person = personRepository.findByLastName("Doe", Person.class);
    PersonView personView = personRepository.findByLastName("Doe", PersonView.class);
    PersonDto personDto = personRepository.findByLastName("Doe", PersonDto.class);

    assertThat(person.getFirstName()).isEqualTo("John");
    assertThat(personView.getFirstName()).isEqualTo("John");
    assertThat(personDto.getFirstName()).isEqualTo("John");
}
```

## 6。结论

在本文中，我们讨论了各种类型的 Spring 数据 JPA 预测。

这篇文章的源代码可以在 GitHub 的[上找到。这是一个 Maven 项目，应该能够按原样运行。](https://web.archive.org/web/20220628160318/https://github.com/eugenp/tutorials/tree/master/persistence-modules/spring-data-jpa-filtering)