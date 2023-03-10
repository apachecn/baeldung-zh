# spring Data Rest——序列化实体 ID

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-data-rest-serialize-entity-id>

## 1.概观

正如我们所知，当我们想快速开始 RESTful web 服务时， [Spring Data Rest](/web/20220831094937/https://www.baeldung.com/spring-data-rest-intro) 模块可以让我们的生活变得更容易。然而，这个模块有一个默认的行为，这有时会令人困惑。

在本教程中，我们将**了解为什么 Spring Data Rest 默认情况下不序列化实体 id。此外，我们将讨论改变这种行为的各种解决方案。**

## 2.默认行为

在我们进入细节之前，让我们通过一个简单的例子来理解序列化实体 id 的含义。

所以，这里有一个示例实体，`Person`:

```java
@Entity
public class Person {

    @Id
    @GeneratedValue
    private Long id;

    private String name;

    // getters and setters

}
```

此外，我们还有一个存储库，`PersonRepository`:

```java
public interface PersonRepository extends JpaRepository<Person, Long> {

}
```

如果我们使用 Spring Boot，只需添加 [`spring-boot-starter-data-rest`](https://web.archive.org/web/20220831094937/https://search.maven.org/search?q=a:spring-boot-starter-data-rest) 依赖项就可以启用 Spring 数据休息模块:

```java
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-rest</artifactId>
</dependency>
```

有了这两个类和 Spring Boot 的自动配置，我们的 REST 控制器就可以自动使用了。

下一步，让我们请求资源`http://localhost:8080/persons`，并检查框架生成的默认 JSON 响应:

```java
{
    "_embedded" : {
        "persons" : [ {
            "name" : "John Doe",
            "_links" : {
                "self" : {
                    "href" : "http://localhost:8080/persons/1"
                },
                "person" : {
                    "href" : "http://localhost:8080/persons/1{?projection}",
                    "templated" : true
                }
            }
        }, ...]
    ...
}
```

为了简洁起见，我们省略了一些部分。正如我们注意到的，对于实体`Person.` ，只有`name`字段被序列化，而`id`字段被去掉了。

因此，这是 Spring Data Rest 中的一个设计决策。在大多数情况下，公开我们的内部 id 并不理想，因为它们对外部系统毫无意义。

在理想情况下，**身份是 RESTful 架构中资源的 URL**。

我们还应该看到，只有当我们使用 Spring Data Rest 的端点时才会出现这种情况。我们的自定义`@Controller`或`@RestController`端点不会受到影响，除非我们使用[Spring hatas](/web/20220831094937/https://www.baeldung.com/spring-hateoas-tutorial)的`RepresentationModel`及其子节点——如`CollectionModel`和`EntityModel`——来构建我们的响应。

幸运的是，公开实体 id 是可配置的。所以，我们仍然有实现它的灵活性。

在接下来的部分中，我们将看到在 Spring Data Rest 中公开实体 id 的不同方式。

## 3.使用`RepositoryRestConfigurer`

**公开实体 id 最常见的解决方案是配置`RepositoryRestConfigurer`** :

```java
@Configuration
public class RestConfiguration implements RepositoryRestConfigurer {

    @Override
    public void configureRepositoryRestConfiguration(
      RepositoryRestConfiguration config, CorsRegistry cors) {
        config.exposeIdsFor(Person.class);
    }
}
```

**在 Spring Data Rest 版本 3.1 或 Spring Boot 版本 2.1 之前，我们会使用`RepositoryRestConfigurerAdapter`** :

```java
@Configuration
public class RestConfiguration extends RepositoryRestConfigurerAdapter {
    @Override
    public void configureRepositoryRestConfiguration(RepositoryRestConfiguration config) {
        config.exposeIdsFor(Person.class);
    }
}
```

虽然差不多，但是要注意版本。作为旁注，自 Spring Data Rest 版本 3.1 [`RepositoryRestConfigurerAdapter`](https://web.archive.org/web/20220831094937/https://docs.spring.io/spring-data/rest/docs/3.1.0.RELEASE/api/org/springframework/data/rest/webmvc/config/RepositoryRestConfigurerAdapter.html) 被弃用，在最新的 [4.0.x](https://web.archive.org/web/20220831094937/https://docs.spring.io/spring-data/rest/docs/4.0.x/api/) 分支中已经被移除。

在我们对实体`Person,` 进行配置之后，响应也为我们提供了`id`字段:

```java
{
    "_embedded" : {
        "persons" : [ {
            "id" : 1,
            "name" : "John Doe",
            "_links" : {
                "self" : {
                    "href" : "http://localhost:8080/persons/1"
                },
                "person" : {
                    "href" : "http://localhost:8080/persons/1{?projection}",
                    "templated" : true
                }
            }
        }, ...]
    ...
}
```

显然，当我们想要为所有实体启用 id 公开时，如果我们有许多实体，这种解决方案是不实际的。

因此，让我们通过一种通用方法来改进我们的`RestConfiguration`:

```java
@Configuration
public class RestConfiguration implements RepositoryRestConfigurer {

    @Autowired
    private EntityManager entityManager;

    @Override
    public void configureRepositoryRestConfiguration(
      RepositoryRestConfiguration config, CorsRegistry cors) {
        Class[] classes = entityManager.getMetamodel()
          .getEntities().stream().map(Type::getJavaType).toArray(Class[]::new);
        config.exposeIdsFor(classes);
    }
}
```

当我们使用 JPA 来管理持久性时，我们可以以通用的方式访问实体的元数据。JPA 的`EntityManager`已经存储了我们需要的元数据。因此，**我们实际上可以通过`entityManager.getMetamodel()`方法**来收集实体类类型。

因此，这是一个更全面的解决方案，因为每个实体的 id 暴露都是自动启用的。

## 4.使用`@Projection`

另一个解决方案是使用`@Projection`注释。通过定义一个`PersonView`接口，我们也可以公开`id`字段:

```java
@Projection(name = "person-view", types = Person.class)
public interface PersonView {

    Long getId();

    String getName();

}
```

然而，我们现在应该使用不同的请求来测试，`http://localhost:8080/persons?projection=person-view`:

```java
{
    "_embedded" : {
        "persons" : [ {
            "id" : 1,
            "name" : "John Doe",
            "_links" : {
                "self" : {
                    "href" : "http://localhost:8080/persons/1"
                },
                "person" : {
                    "href" : "http://localhost:8080/persons/1{?projection}",
                    "templated" : true
                }
            }
        }, ...]
    ...
}
```

**为了启用由存储库生成的所有端点的投影，我们可以在`PersonRepository`上使用`@RepositoryRestResource`注释**:

```java
@RepositoryRestResource(excerptProjection = PersonView.class)
public interface PersonRepository extends JpaRepository<Person, Long> {

} 
```

更改之后，我们可以使用我们通常的请求`http://localhost:8080/persons`来列出 person 实体。

**但是需要注意的是`excerptProjection`并没有自动应用单项资源**。我们仍然必须使用`http://localhost:8080/persons/1?projection=person-view`来获取单个`Person`的响应及其实体 id。

此外，我们应该记住，我们的投影中定义的字段并不总是按顺序排列的:

```java
{
    ...            
    "persons" : [ {
        "name" : "John Doe",
        "id" : 1,
        ...
    }, ...]
    ...
}
```

为了让**保持字段顺序，我们可以将`@JsonPropertyOrder`注释放在我们的`PersonView`类**上:

```java
@JsonPropertyOrder({"id", "name"})
@Projection(name = "person-view", types = Person.class)
public interface PersonView { 
    //...
}
```

## 5.在 Rest 存储库上使用 dto

覆盖 rest 控制器处理程序是另一种解决方案。Spring Data Rest 允许我们插入定制的处理程序。因此，**我们仍然可以使用底层存储库来获取数据，但是在响应到达客户端**之前覆盖它。在这种情况下，我们将编写更多的代码，但我们将拥有完全定制的能力。

### 5.1.履行

首先，我们定义一个 DTO 对象来表示我们的`Person`实体:

```java
public class PersonDto {

    private Long id;

    private String name;

    public PersonDto(Person person) {
        this.id = person.getId();
        this.name = person.getName();
    }

    // getters and setters
}
```

正如我们所看到的，我们在这里添加了一个`id`字段，它对应于`Person`的实体 id。

下一步，我们将使用一些内置的助手类来重用 Spring Data Rest 的响应构建机制，同时尽可能保持响应结构不变。

因此，让我们定义我们的`PersonController`来覆盖内置端点:

```java
@RepositoryRestController
public class PersonController {

    @Autowired
    private PersonRepository repository;

    @GetMapping("/persons")
    ResponseEntity<?> persons(PagedResourcesAssembler resourcesAssembler) {
        Page<Person> persons = this.repository.findAll(Pageable.ofSize(20));
        Page<PersonDto> personDtos = persons.map(PersonDto::new);
        PagedModel<EntityModel<PersonDto>> pagedModel = resourcesAssembler.toModel(personDtos);
        return ResponseEntity.ok(pagedModel);
    }

}
```

我们应该注意以下几点，以确保 Spring 将我们的控制器类识别为一个插件，而不是一个独立的控制器:

1.  必须用`@RepositoryRestController`代替`@RestController`或`@Controller`
2.  `PersonController`类必须放在 Spring 的组件扫描可以拾取的包下，或者，我们可以使用`@Bean.`显式定义它
3.  `@GetMapping`路径必须与`PersonRepository`提供的路径相同。如果我们用`@RepositoryRestResource(path = “…”),`定制路径，那么控制器的 get 映射也必须反映这一点。

最后，让我们试试我们的端点，`http://localhost:8080/persons`:

```java
{
    "_embedded" : {
        "personDtoes" : [ {
            "id" : 1,
            "name" : "John Doe"
        }, ...]
    }, ...
}
```

我们可以在响应中看到`id`字段。

### 5.2.缺点

如果我们选择 DTO 的 Spring Data Rest 库，我们应该考虑几个方面。

一些开发人员不喜欢将实体模型直接序列化到响应中。当然，它也有一些缺点。**暴露所有实体字段可能会导致数据泄漏、意外的延迟获取和性能问题**。

然而，**为所有端点编写我们的`@RepositoryRestController`是一种妥协**。它带走了框架的一些好处。此外，在这种情况下，我们需要维护更多的代码。

## 6.结论

在本文中，我们讨论了在使用 Spring Data Rest 时公开实体 id 的多种方法。

像往常一样，我们可以在 Github 上找到本文[中使用的所有代码示例。](https://web.archive.org/web/20220831094937/https://github.com/eugenp/tutorials/tree/master/persistence-modules/spring-data-rest-2)