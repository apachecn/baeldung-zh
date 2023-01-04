# 使用 JPA 元模型的标准查询

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/hibernate-criteria-queries-metamodel>

## 1.概观

在本教程中，我们将讨论如何在 Hibernate 中编写条件查询时使用 JPA 静态元模型类。

我们需要对 Hibernate 中的条件查询 API 有一个基本的了解，所以如果需要的话，请查看我们关于[条件查询](/web/20220926190808/https://www.baeldung.com/hibernate-criteria-queries)的教程以获得关于这个主题的更多信息。

## 2.为什么选择 JPA 元模型？

通常，当我们编写一个条件查询时，我们需要引用实体类及其属性。

现在，一种方法是以字符串的形式提供属性的名称。但是，这有几个缺点。

首先，我们必须查找实体属性的名称。而且，如果列名在项目生命周期的后期被更改，我们必须重构使用该名称的每个查询。

社区引入了 [JPA 元模型](https://web.archive.org/web/20220926190808/https://docs.jboss.org/hibernate/orm/5.0/topical/html/metamodelgen/MetamodelGenerator.html#_canonical_metamodel)来避免这些缺点，并提供对托管实体类的元数据的静态访问。

## 3.实体类

让我们考虑这样一个场景，我们正在为我们的一个客户构建一个学生门户管理系统，出现了一个基于他们毕业年份在`Students` 上提供搜索功能的需求。

首先，让我们看看我们的`Student` 类:

```
@Entity
@Table(name = "students")
public class Student {

    @Id
    @GeneratedValue(strategy = GenerationType.AUTO)
    private int id;

    @Column(name = "first_name")
    private String firstName;

    @Column(name = "last_name")
    private String lastName;

    @Column(name = "grad_year")
    private int gradYear;

    // standard getters and setters
}
```

## 4.生成 JPA 元模型类

接下来，我们需要生成元模型类，为此，我们将使用 [JBoss](https://web.archive.org/web/20220926190808/https://docs.jboss.org/hibernate/orm/5.0/topical/html/metamodelgen/MetamodelGenerator.html) 提供的元模型生成器工具。JBoss 只是生成元模型的众多工具之一。其他合适的工具包括 [EclipseLink](https://web.archive.org/web/20220926190808/https://wiki.eclipse.org/UserGuide/JPA/Using_the_Canonical_Model_Generator_(ELUG)) 、 [OpenJPA](https://web.archive.org/web/20220926190808/https://openjpa.apache.org/builds/2.4.1/apache-openjpa/docs/ch13s04.html) 和 [DataNucleus](https://web.archive.org/web/20220926190808/https://www.datanucleus.org/products/accessplatform_5_2/jpa/query.html#metamodel) 。

要使用 JBoss 工具，我们需要在我们的`pom.xml`文件中添加[最新的依赖关系](https://web.archive.org/web/20220926190808/https://search.maven.org/search?q=g:org.hibernate%20AND%20a:hibernate-jpamodelgen)，一旦我们触发 maven build 命令，该工具将生成元模型类:

```
<dependency>
    <groupId>org.hibernate</groupId>
    <artifactId>hibernate-jpamodelgen</artifactId>
    <version>5.3.7.Final</version>
</dependency>
```

注意，我们需要**将`target/generated-classes `文件夹添加到我们的 IDE** 的类路径中，因为默认情况下，类将只在这个文件夹中生成。

## 5.静态 JPA 元模型类

根据 JPA 规范，生成的类将与对应的实体类驻留在同一个包中，并且将具有相同的名称，并在末尾添加一个“_”(下划线)。因此，**为`Student `类生成的元模型类将是** `**Student_** `，看起来像这样:

```
@Generated(value = "org.hibernate.jpamodelgen.JPAMetaModelEntityProcessor")
@StaticMetamodel(Student.class)
public abstract class Student_ {

    public static volatile SingularAttribute<Student, String> firstName;
    public static volatile SingularAttribute<Student, String> lastName;
    public static volatile SingularAttribute<Student, Integer> id;
    public static volatile SingularAttribute<Student, Integer> gradYear;

    public static final String FIRST_NAME = "firstName";
    public static final String LAST_NAME = "lastName";
    public static final String ID = "id";
    public static final String GRAD_YEAR = "gradYear";
}
```

## 6.使用 JPA 元模型类

**我们可以像使用属性的`String `引用一样使用静态元模型类。**标准查询 API 提供了接受`String`引用和`Attribute `接口实现的重载方法。

让我们来看看将获取所有 2015 年毕业的`Students`的条件查询:

```
//session set-up code
CriteriaBuilder cb = session.getCriteriaBuilder();
CriteriaQuery<Student> criteriaQuery = cb.createQuery(Student.class);

Root<Student> root = criteriaQuery.from(Student.class);
criteriaQuery.select(root).where(cb.equal(root.get(Student_.gradYear), 2015));

Query<Student> query = session.createQuery(criteriaQuery);
List<Student> results = query.getResultList();
```

注意我们如何使用了`Student_.gradYear `引用，而不是使用传统的`grad_year `列名。

## 7.结论

在这篇简短的文章中，我们学习了如何使用静态元模型类，以及为什么它们比前面描述的使用`String `引用的传统方式更受欢迎。

本教程的源代码可以在 Github 上找到[。](https://web.archive.org/web/20220926190808/https://github.com/eugenp/tutorials/tree/master/persistence-modules/hibernate-queries)