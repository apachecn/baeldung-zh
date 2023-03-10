# Spring Data Couchbase 简介

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-data-couchbase>

## 1。简介

在这篇关于 Spring 数据的教程中，我们将讨论如何使用 Spring 数据存储库和模板抽象为 Couchbase 文档建立一个持久层，以及使用视图和/或索引准备 Couchbase 以支持这些抽象所需的步骤。

## 2。Maven 依赖关系

首先，我们将下面的 Maven 依赖项添加到我们的`pom.xml`文件中:

```java
<dependency>
    <groupId>org.springframework.data</groupId>
    <artifactId>spring-data-couchbase</artifactId>
    <version>2.0.0.RELEASE</version>
</dependency>
```

注意，通过包含这种依赖，我们自动获得了一个本地 Couchbase SDK 的兼容版本，所以我们不需要显式地包含它。

为了增加对 JSR-303 bean 验证的支持，我们还包括了以下依赖项:

```java
<dependency>
    <groupId>org.hibernate</groupId>
    <artifactId>hibernate-validator</artifactId>
    <version>5.2.4.Final</version>
</dependency>
```

Spring Data Couchbase 通过传统的 date 和 Calendar 类以及 Joda 时间库支持日期和时间持久性，我们包括如下内容:

```java
<dependency>
    <groupId>joda-time</groupId>
    <artifactId>joda-time</artifactId>
    <version>2.9.2</version>
</dependency>
```

## 3。配置

接下来，我们需要通过指定 Couchbase 集群的一个或多个节点以及存储文档的存储桶的名称和密码来配置 Couchbase 环境。

### 3.1。Java 配置

对于 Java 类配置，我们简单地扩展了`AbstractCouchbaseConfiguration`类:

```java
@Configuration
@EnableCouchbaseRepositories(basePackages={"com.baeldung.spring.data.couchbase"})
public class MyCouchbaseConfig extends AbstractCouchbaseConfiguration {

    @Override
    protected List<String> getBootstrapHosts() {
        return Arrays.asList("localhost");
    }

    @Override
    protected String getBucketName() {
        return "baeldung";
    }

    @Override
    protected String getBucketPassword() {
        return "";
    }
}
```

如果您的项目需要对 Couchbase 环境进行更多的定制，您可以通过覆盖`getEnvironment()`方法来提供:

```java
@Override
protected CouchbaseEnvironment getEnvironment() {
   ...
}
```

### 3.2。XML 配置

下面是等效的 XML 配置:

```java
<?xml version="1.0" encoding="UTF-8"?>
<beans:beans xmlns:beans="http://www.springframework.org/schema/beans"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xmlns="http://www.springframework.org/schema/data/couchbase
  xsi:schemaLocation="http://www.springframework.org/schema/beans
    http://www.springframework.org/schema/beans/spring-beans.xsd
    http://www.springframework.org/schema/data/couchbase
    http://www.springframework.org/schema/data/couchbase/spring-couchbase.xsd">

    <couchbase:cluster>
        <couchbase:node>localhost</couchbase:node>
    </couchbase:cluster>

    <couchbase:clusterInfo login="baeldung" password="" />

    <couchbase:bucket bucketName="baeldung" bucketPassword=""/>

    <couchbase:repositories base-package="com.baeldung.spring.data.couchbase"/>
</beans:beans>
```

注意:“`clusterInfo`”节点接受集群凭证或存储桶凭证，并且是**必需的**，以便库可以确定您的 Couchbase 集群是否支持 N1QL(NoSQL 数据库的 SQL 超集，在 Couchbase 4.0 和更高版本中可用)。

如果您的项目需要一个定制的 Couchbase 环境，您可以使用`<couchbase:env/>`标签提供一个。

## 4。数据模型

让我们创建一个表示 JSON 文档的实体类来持久化。我们首先用`@Document,`注释这个类，然后用`@Id`注释一个`String`字段来表示 Couchbase 文档键。

您可以使用来自 Spring 数据的`@Id`注释，也可以使用来自本地 Couchbase SDK 的注释。请注意，如果您在两个不同的字段上使用同一个类中的两个`@Id`注释，那么用 Spring 数据`@Id`注释注释的字段将优先，并将被用作文档键。

为了表示 JSON 文档的属性，我们添加了用`@Field`标注的私有成员变量。我们使用`@NotNull`注释来根据需要标记某些字段:

```java
@Document
public class Person {
    @Id
    private String id;

    @Field
    @NotNull
    private String firstName;

    @Field
    @NotNull
    private String lastName;

    @Field
    @NotNull
    private DateTime created;

    @Field
    private DateTime updated;

    // standard getters and setters
}
```

注意，用`@Id`注释的属性仅仅代表文档键，并不一定是存储的 JSON 文档的一部分，除非它也用`@Field`注释，如下所示:

```java
@Id
@Field
private String id;
```

如果您想在实体类中命名一个不同于存储在 JSON 文档中的字段，只需限定它的`@Field`注释，如下例所示:

```java
@Field("fname")
private String firstName;
```

下面的例子展示了持久化的`Person`文档的外观:

```java
{
    "firstName": "John",
    "lastName": "Smith",
    "created": 1457193705667
    "_class": "com.baeldung.spring.data.couchbase.model.Person"
}
```

请注意，Spring Data 会自动为每个文档添加一个包含实体的完整类名的属性。默认情况下，这个属性被命名为`“_class”`，尽管您可以通过覆盖`typeKey()`方法在 Couchbase 配置类中覆盖它。

例如，如果您想要指定一个名为`“dataType”`的字段来保存类名，您可以将它添加到您的 Couchbase 配置类中:

```java
@Override
public String typeKey() {
    return "dataType";
}
```

覆盖`typeKey()`的另一个普遍原因是，如果您使用的 Couchbase Mobile 版本不支持带下划线前缀的字段。在这种情况下，您可以像前面的例子一样选择自己的替代类型字段，或者使用 Spring 提供的替代类型:

```java
@Override
public String typeKey() {
    // use "javaClass" instead of "_class"
    return MappingCouchbaseConverter.TYPEKEY_SYNCGATEWAY_COMPATIBLE;
}
```

## 5。Couchbase 知识库

Spring Data Couchbase 提供了与 JPA 等其他 Spring 数据模块相同的内置查询和派生查询机制。

我们通过扩展`CrudRepository<String,Person>`并添加一个可派生的查询方法来为`Person`类声明一个存储库接口:

```java
public interface PersonRepository extends CrudRepository<Person, String> {
    List<Person> findByFirstName(String firstName);
}
```

## 6。N1QL 通过索引支持

如果使用 Couchbase 4.0 或更高版本，那么默认情况下，使用 N1QL 引擎处理定制查询(除非它们对应的存储库方法用`@View`进行了注释，以指示使用下一节中描述的后备视图)。

要添加对 N1QL 的支持，您必须在 bucket 上创建一个主索引。您可以通过使用`cbq`命令行查询处理器(参见 Couchbase 文档，了解如何为您的环境启动`cbq`工具)并发出以下命令来创建索引:

```java
CREATE PRIMARY INDEX ON baeldung USING GSI;
```

在上面的命令中，`GSI`代表`global secondary index`，这是一种特别适合优化支持 OLTP 系统的临时 N1QL 查询的索引类型，如果没有另外指定，这是默认的索引类型。

与基于视图的索引不同，GSI 索引不会在集群中的所有索引节点上自动复制，因此，如果集群包含多个索引节点，则需要在集群中的每个节点上创建每个 GSI 索引，并且必须在每个节点上提供不同的索引名称。

您还可以创建一个或多个辅助索引。当您这样做时，Couchbase 将根据需要使用它们来优化其查询处理。

例如，要在`firstName`字段上添加索引，在`cbq`工具中发出以下命令:

```java
CREATE INDEX idx_firstName ON baeldung(firstName) USING GSI;
```

## 7。背景视图

对于每个存储库接口，您将需要创建一个 Couchbase 设计文档和目标存储桶中的一个或多个视图。设计文档名称必须是实体类名的`lowerCamelCase`版本(如`“person”`)。

无论您运行的是哪个版本的 Couchbase Server，您都必须创建一个名为`“all”`的支持视图来支持内置的“`findAll”`存储库方法。这是我们的`Person`类的`“all”`视图的地图函数:

```java
function (doc, meta) {
    if(doc._class == "com.baeldung.spring.data.couchbase.model.Person") {
        emit(meta.id, null);
    }
}
```

当使用 4.0 之前的 Couchbase 版本时，每个自定义存储库方法都必须有一个支持视图(在 4.0 或更高版本中，支持视图的使用是可选的)。

视图支持的自定义方法必须用`@View`进行注释，如下例所示:

```java
@View
List<Person> findByFirstName(String firstName);
```

支持视图的默认命名约定是使用跟在“`find”`关键字后面的方法名的`lowerCamelCase`版本(例如`“byFirstName”`)。

下面是如何为`“byFirstName”`视图编写映射函数:

```java
function (doc, meta) {
    if(doc._class == "com.baeldung.spring.data.couchbase.model.Person"
      && doc.firstName) {
        emit(doc.firstName, null);
    }
}
```

通过用相应的支持视图的名称限定每个`@View`注释，您可以忽略这个命名约定并使用您自己的视图名称。例如:

```java
@View("myCustomView")
List<Person> findByFirstName(String lastName);
```

## 8。服务层

对于我们的服务层，我们定义了一个接口和两个实现:一个使用 Spring 数据仓库抽象，另一个使用 Spring 数据模板抽象。这里是我们的`PersonService`界面:

```java
public interface PersonService {
    Person findOne(String id);
    List<Person> findAll();
    List<Person> findByFirstName(String firstName);

    void create(Person person);
    void update(Person person);
    void delete(Person person);
}
```

### 8.1。储存库服务

下面是一个使用我们上面定义的存储库的实现:

```java
@Service
@Qualifier("PersonRepositoryService")
public class PersonRepositoryService implements PersonService {

    @Autowired
    private PersonRepository repo; 

    public Person findOne(String id) {
        return repo.findOne(id);
    }

    public List<Person> findAll() {
        List<Person> people = new ArrayList<Person>();
        Iterator<Person> it = repo.findAll().iterator();
        while(it.hasNext()) {
            people.add(it.next());
        }
        return people;
    }

    public List<Person> findByFirstName(String firstName) {
        return repo.findByFirstName(firstName);
    }

    public void create(Person person) {
        person.setCreated(DateTime.now());
        repo.save(person);
    }

    public void update(Person person) {
        person.setUpdated(DateTime.now());
        repo.save(person);
    }

    public void delete(Person person) {
        repo.delete(person);
    }
}
```

### 8.2。模板服务

对于基于模板的实现，我们必须创建上面第 7 节中列出的支持视图。`CouchbaseTemplate`对象在我们的 Spring 上下文中是可用的，并且可能被注入到服务类中。

下面是使用模板抽象的实现:

```java
@Service
@Qualifier("PersonTemplateService")
public class PersonTemplateService implements PersonService {
    private static final String DESIGN_DOC = "person";
    @Autowired
    private CouchbaseTemplate template;

    public Person findOne(String id) {
       return template.findById(id, Person.class);
    }

    public List<Person> findAll() {
        return template.findByView(ViewQuery.from(DESIGN_DOC, "all"), Person.class);
    }

    public List<Person> findByFirstName(String firstName) {
        return template.findByView(ViewQuery.from(DESIGN_DOC, "byFirstName"), Person.class);
    }

    public void create(Person person) {
        person.setCreated(DateTime.now());
        template.insert(person);
    }

    public void update(Person person) {
        person.setUpdated(DateTime.now());
        template.update(person);
    }

    public void delete(Person person) {
        template.remove(person);
    }
}
```

## 9。结论

我们已经展示了如何配置一个项目来使用 Spring Data Couchbase 模块，以及如何编写一个简单的实体类及其存储库接口。我们编写了一个简单的服务接口，并提供了一个使用存储库的实现和另一个使用 Spring 数据模板 API 的实现。

你可以在 GitHub 项目中查看本教程的完整源代码。

欲了解更多信息，请访问 [Spring Data Couchbase](https://web.archive.org/web/20220628113149/https://spring.io/projects/spring-data-couchbase) 项目网站。