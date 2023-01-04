# Spring 数据 JPA 存储库填充器

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-data-jpa-repository-populators>

## 1.介绍

在这篇简短的文章中，我们将通过一个简单的例子来探索 **Spring JPA 存储库填充器**。Spring Data JPA repository populator 是`data.sql`脚本的一个很好的替代品。

spring Data JPA repository populator 支持 JSON 和 XML 文件格式。在接下来的章节中，我们将看到如何使用 Spring Data JPA repository populator。

## 2.示例应用程序

首先，假设我们有一个`Fruit`实体类和一个水果库存来填充我们的数据库:

```
@Entity
public class Fruit {
    @Id
    private long id;
    private String name;
    private String color;

    // getters and setters
}
```

我们将扩展`JpaRepository `以从数据库中读取`Fruit`数据:

```
@Repository
public interface FruitRepository extends JpaRepository<Fruit, Long> {
    // ...
}
```

在下一节中，我们将使用 JSON 格式来存储和填充初始水果数据。

## 3.JSON 存储库填充器

让我们用`Fruit`数据创建一个 JSON 文件。我们将在`src/main/resources`中创建这个文件，并将其命名为`fruit-data.json`:

```
[
    {
        "_class": "com.baeldung.entity.Fruit",
        "name": "apple",
        "color": "red",
        "id": 1
    },
    {
        "_class": "com.baeldung.entity.Fruit",
        "name": "guava",
        "color": "green",
        "id": 2
    }
]
```

**应该在每个 JSON 对象的`_class`字段中给出实体类名。**剩余的键映射到我们的`Fruit`实体的列。

现在，我们将在`pom.xml`中添加 [`jackson-databind`](https://web.archive.org/web/20220629001722/https://search.maven.org/search?q=g:com.fasterxml.jackson.core%20a:jackson-databind) 依赖关系:

```
<dependency>
    <groupId>com.fasterxml.jackson.core</groupId>
    <artifactId>jackson-databind</artifactId>
    <version>2.9.8</version>
</dependency>
```

最后，我们必须添加一个存储库填充器 bean。这个存储库填充器 bean 将从`fruit-data.json`文件中读取数据，并在应用程序启动时将其填充到数据库中:

```
@Bean
public Jackson2RepositoryPopulatorFactoryBean getRespositoryPopulator() {
    Jackson2RepositoryPopulatorFactoryBean factory = new Jackson2RepositoryPopulatorFactoryBean();
    factory.setResources(new Resource[]{new ClassPathResource("fruit-data.json")});
    return factory;
}
```

我们已经准备好对我们的配置进行单元测试:

```
@Test
public void givenFruitJsonPopulatorThenShouldInsertRecordOnStart() {
    List<Fruit> fruits = fruitRepository.findAll();
    assertEquals("record count is not matching", 2, fruits.size());

    fruits.forEach(fruit -> {
        if (1 == fruit.getId()) {
            assertEquals("apple", fruit.getName());
            assertEquals("red", fruit.getColor());
        } else if (2 == fruit.getId()) {
            assertEquals("guava", fruit.getName());
            assertEquals("green", fruit.getColor());
        }
    });
}
```

## 4.XML 存储库填充器

在这一节中，我们将看到如何在存储库填充器中使用 XML 文件。首先，我们将创建一个包含所需的`Fruit`细节的 XML 文件。

这里，一个 XML 文件代表一个水果的数据。

`apple-fruit-data.xml`:

```
<fruit>
    <id>1</id>
    <name>apple</name>
    <color>red</color>
</fruit>
```

`guava-fruit-data.xml`:

```
<fruit>
    <id>2</id>
    <name>guava</name>
    <color>green</color>
</fruit>
```

同样，我们将这些 XML 文件存储在`src/main/resources` `.`中

此外，我们将在`pom.xml`中添加`[spring-oxm](https://web.archive.org/web/20220629001722/https://search.maven.org/search?q=g:org.springframework%20a:spring-oxm)` maven 依赖项:

```
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-oxm</artifactId>
    <version>5.1.5.RELEASE</version>
</dependency> 
```

此外，我们需要给我们的实体类添加`@XmlRootElement` 注释:

```
@XmlRootElement
@Entity
public class Fruit {
    // ...
}
```

最后，我们将定义一个存储库填充器 bean。该 bean 将读取 XML 文件并填充数据:

```
@Bean
public UnmarshallerRepositoryPopulatorFactoryBean repositoryPopulator() {
    Jaxb2Marshaller unmarshaller = new Jaxb2Marshaller();
    unmarshaller.setClassesToBeBound(Fruit.class);

    UnmarshallerRepositoryPopulatorFactoryBean factory = new UnmarshallerRepositoryPopulatorFactoryBean();
    factory.setUnmarshaller(unmarshaller);
    factory.setResources(new Resource[] { new ClassPathResource("apple-fruit-data.xml"), 
      new ClassPathResource("guava-fruit-data.xml") });
    return factory;
}
```

我们可以对 XML 存储库填充器进行单元测试，就像我们对 JSON 填充器进行单元测试一样。

## 4.结论

在本教程中，我们学习了如何使用 **Spring 数据 JPA 存储库填充器**。用于本教程的完整源代码可以在 GitHub 的[上找到。](https://web.archive.org/web/20220629001722/https://github.com/eugenp/tutorials/tree/master/persistence-modules/spring-data-jpa-repo)