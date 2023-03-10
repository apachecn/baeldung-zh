# JPA 查询语言指南

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/querydsl-with-jpa-tutorial>

 ![announcement-icon.png](img/3389c2017376cafcdb912bf911dd5a62.png)

JPA can behave very differently depending on the exact circumstances under which it is used. Code that works in our local environment or in staging performs very poorly (or even flat out fails) when thrown against real-scale databases in production environments.

Debugging these JPA issues in production is pretty difficult - existing APMs don’t provide enough granular insights at the code level, and tracking every single place someone queried entities one by one instead of in bulk can be a grueling, time-consuming task.

**Lightrun** is a new approach to debugging in production. Using Lightrun’s Logs and Snapshots, you can now get debugger-level granularity in production without opening inbound ports, redeploying, restarting, or even stropping the running application.

In addition, instrumenting Lightrun Metrics at runtime allows you to track down persistence issues securely and in real-time. Want to see it in action? **Check out our 2-minute tutorial** for debugging JPA performance issues in production using Lightrun:

[>> Debugging Spring Persistence and JPA Issues Using Lightrun](/web/20220525013414/https://www.baeldung.com/lightrun-n-jpa)

## 1。概述

Querydsl 是一个扩展的 Java 框架，它有助于用类似于 SQL 的领域特定语言创建和运行**类型安全查询。**

在本文中，我们将探索带有 Java 持久性 API 的 Querydsl。

这里有一个小提示，Hibernate 的 HQL 是 Querydsl 的第一个目标语言，但是现在它支持 JPA、JDO、JDBC、Lucene、Hibernate Search、MongoDB、Collections 和 RDFBean 作为后端。

## 2。准备工作

让我们首先将必要的依赖项添加到 Maven 项目中:

```java
<properties>
    <querydsl.version>2.5.0</querydsl.version>
</properties>

<dependency>
    <groupId>com.querydsl</groupId>
    <artifactId>querydsl-apt</artifactId>
    <version>${querydsl.version}</version>
    <scope>provided</scope>
</dependency>

<dependency>
    <groupId>com.querydsl</groupId>
    <artifactId>querydsl-jpa</artifactId>
    <version>${querydsl.version}</version>
</dependency>

<dependency>
    <groupId>org.slf4j</groupId>
    <artifactId>slf4j-log4j12</artifactId>
    <version>1.6.1</version>
</dependency>
```

现在让我们配置 Maven APT 插件:

```java
<project>
    <build>
    <plugins>
    ...
    <plugin>
        <groupId>com.mysema.maven</groupId>
        <artifactId>apt-maven-plugin</artifactId>
        <version>1.1.3</version>
        <executions>
        <execution>
            <goals>
                <goal>process</goal>
            </goals>
            <configuration>
                <outputDirectory>target/generated-sources</outputDirectory>
                <processor>com.querydsl.apt.jpa.JPAAnnotationProcessor</processor>
            </configuration>
        </execution>
        </executions>
    </plugin>
    ...
    </plugins>
    </build>
</project>
```

`JPAAnnotationProcessor`将找到用`javax.persistence.Entity`注释标注的域类型，并为它们生成查询类型。

## 3。使用 Querydsl 的查询

查询是基于生成的查询类型构造的，这些查询类型反映了您的域类型的属性。同样，函数/方法调用是以完全类型安全的方式构造的。

查询路径和操作在所有实现中都是相同的，而且`Query`接口有一个公共的基本接口。

### 3.1。一个实体和 Querydsl 查询类型

让我们首先定义一个简单的实体，我们将在下面的例子中使用它:

```java
@Entity
public class Person {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column
    private String firstname;

    @Column
    private String surname;

    Person() {
    }

    public Person(String firstname, String surname) {
        this.firstname = firstname;
        this.surname = surname;
    }

    // standard getters and setters

}
```

Querydsl 将生成一个简单名称为`QPerson`的查询类型，并将其放入与`Person`相同的包中。`QPerson`可以作为 Querydsl 查询中的静态类型变量，作为`Person`类型的代表。

首先–`QPerson`有一个默认的实例变量，可以作为静态字段访问:

```java
QPerson person = QPerson.person;
```

或者，您可以像这样定义自己的`Person`变量:

```java
QPerson person = new QPerson("Erich", "Gamma");
```

### 3.2。使用`JPAQuery` 构建查询

我们现在可以使用`JPAQuery`实例进行查询:

```java
JPAQuery query = new JPAQuery(entityManager);
```

注意`entityManager`是一个 JPA `EntityManager`。

现在让我们检索名字为“`Kent`”的所有人，作为一个简单的例子:

```java
QPerson person = QPerson.person;
List<Person> persons = query.from(person).where(person.firstName.eq("Kent")).list(person);
```

`from`调用定义查询源和投影，`where`部分定义过滤器，`list`告诉 Querydsl 返回所有匹配的元素。

我们也可以使用多个过滤器:

```java
query.from(person).where(person.firstName.eq("Kent"), person.surname.eq("Beck"));
```

或者:

```java
query.from(person).where(person.firstName.eq("Kent").and(person.surname.eq("Beck")));
```

在原生 JPQL 格式中，查询将如下所示:

```java
select person from Person as person where person.firstName = "Kent" and person.surname = "Beck"
```

如果您想通过“或”来组合过滤器，请使用以下模式:

```java
query.from(person).where(person.firstName.eq("Kent").or(person.surname.eq("Beck")));
```

## 4。Querydsl 中的排序和聚合

现在让我们看看 Querydsl 库中的排序和聚合是如何工作的。

### 4.1。订购

我们将从按`surname`字段降序排列我们的结果开始:

```java
QPerson person = QPerson.person;
List<Person> persons = query.from(person)
    .where(person.firstname.eq(firstname))
    .orderBy(person.surname.desc())
    .list(person);
```

### 4.2。聚合

现在让我们使用一个简单的聚合，因为我们有几个可用的(总和、平均值、最大值、最小值):

```java
QPerson person = QPerson.person;    
int maxAge = query.from(person).list(person.age.max()).get(0);
```

### 4.3。`GroupBy`聚合着

`com.mysema.query.group.GroupBy`类提供了聚合功能，我们可以用它来聚合内存中的查询结果。

这里有一个简单的例子，结果以`Map`的形式返回，其中`firstname`是键，`max age`是值:

```java
QPerson person = QPerson.person;   
Map<String, Integer> results = 
  query.from(person).transform(
      GroupBy.groupBy(person.firstname).as(GroupBy.max(person.age)));
```

## 5。使用 Querydsl 进行测试

现在，让我们使用 Querydsl 定义一个 DAO 实现——并定义以下搜索操作:

```java
public List<Person> findPersonsByFirstnameQuerydsl(String firstname) {
    JPAQuery query = new JPAQuery(em);
    QPerson person = QPerson.person;
    return query.from(person).where(person.firstname.eq(firstname)).list(person);
}
```

现在让我们使用这个新的 DAO 构建几个测试，让我们使用 Querydsl 来搜索新创建的`Person`对象(在`PersonDao`类中实现),并在另一个使用`GroupBy`类的测试聚合中进行测试:

```java
@Autowired
private PersonDao personDao;

@Test
public void givenExistingPersons_whenFindingPersonByFirstName_thenFound() {
    personDao.save(new Person("Erich", "Gamma"));
    Person person = new Person("Kent", "Beck");
    personDao.save(person);
    personDao.save(new Person("Ralph", "Johnson"));

    Person personFromDb =  personDao.findPersonsByFirstnameQuerydsl("Kent").get(0);
    Assert.assertEquals(person.getId(), personFromDb.getId());
}

@Test
public void givenExistingPersons_whenFindingMaxAgeByName_thenFound() {
    personDao.save(new Person("Kent", "Gamma", 20));
    personDao.save(new Person("Ralph", "Johnson", 35));
    personDao.save(new Person("Kent", "Zivago", 30));

    Map<String, Integer> maxAge = personDao.findMaxAgeByName();
    Assert.assertTrue(maxAge.size() == 2);
    Assert.assertSame(35, maxAge.get("Ralph"));
    Assert.assertSame(30, maxAge.get("Kent"));
}
```

## 6。结论

本教程演示了如何使用 Querydsl 构建 JPA 项目。

本文的**完整实现**可以在 github 项目的[中找到——这是一个基于 Eclipse 的 maven 项目，因此它应该很容易导入和运行。](https://web.archive.org/web/20220525013414/https://github.com/eugenp/tutorials/tree/master/persistence-modules/querydsl)

这里需要注意的是——运行一个简单的 maven 构建(mvn clean install)来将类型生成到`target/generated-sources`中——然后，如果您使用的是 Eclipse——将该文件夹作为项目的源文件夹。