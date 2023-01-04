# 我的春天

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-mybatis>

## 1。简介

MyBatis 是在 Java 应用程序中实现 SQL 数据库访问最常用的开源框架之一。

在这个快速教程中，我们将介绍如何将 MyBatis 与 Spring 和 Spring Boot 集成。

对于那些还不熟悉这个框架的人，一定要看看我们关于使用 MyBatis 的[文章。](/web/20220530025634/https://www.baeldung.com/mybatis)

## 2。定义模型

让我们从定义贯穿本文的简单 POJO 开始:

```java
public class Article {
    private Long id;
    private String title;
    private String author;

    // constructor, standard getters and setters
}
```

和一个等效的 SQL `schema.sql`文件:

```java
CREATE TABLE IF NOT EXISTS `ARTICLES`(
    `id`          INTEGER PRIMARY KEY,
    `title`       VARCHAR(100) NOT NULL,
    `author`      VARCHAR(100) NOT NULL
);
```

接下来，让我们创建一个`data.sql`文件，它只是将一条记录插入到我们的`articles`表中:

```java
INSERT INTO ARTICLES
VALUES (1, 'Working with MyBatis in Spring', 'Baeldung');
```

两个 SQL 文件都必须包含在[类路径](/web/20220530025634/https://www.baeldung.com/spring-classpath-file-access)中。

## 3。弹簧配置

要开始使用 MyBatis，我们必须包括两个主要的依赖项— [MyBatis](https://web.archive.org/web/20220530025634/https://search.maven.org/search?q=g:org.mybatis%20a:mybatis) 和 [MyBatis-Spring](https://web.archive.org/web/20220530025634/https://search.maven.org/search?q=g:org.mybatis%20a:%20mybatis-spring) :

```java
<dependency>
    <groupId>org.mybatis</groupId>
    <artifactId>mybatis</artifactId>
    <version>3.5.2</version>
</dependency>

<dependency>
    <groupId>org.mybatis</groupId>
    <artifactId>mybatis-spring</artifactId>
    <version>2.0.2</version>
</dependency>
```

除此之外，我们还需要基本的 [Spring 依赖关系](https://web.archive.org/web/20220530025634/https://search.maven.org/search?q=g:org.springframework):

```java
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-context</artifactId>
    <version>5.3.8</version>
</dependency>

<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-beans</artifactId>
    <version>5.3.8</version>
</dependency>
```

在我们的示例中，我们将使用[的 H2 嵌入式数据库](https://web.archive.org/web/20220530025634/https://search.maven.org/search?q=g:com.h2database%20a:h2)来简化设置，并使用 [`spring-jdbc`](https://web.archive.org/web/20220530025634/https://search.maven.org/search?q=g:org.springframework%20a:spring-jdbc) 模块中的`EmbeddedDatabaseBuilder `类进行配置:

```java
<dependency>
    <groupId>com.h2database</groupId>
    <artifactId>h2</artifactId>
    <version>1.4.199</version>
</dependency>

<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-jdbc</artifactId>
    <version>5.3.8</version>
</dependency>
```

### 3.1。基于注释的配置

Spring 简化了 MyBatis 的配置。**唯一需要的元素是`javax.sql.Datasource`、`org.apache.ibatis.session.SqlSessionFactory`，以及至少一个映射器。**

首先，让我们创建一个配置类:

```java
@Configuration
@MapperScan("com.baeldung.mybatis")
public class PersistenceConfig {

    @Bean
    public DataSource dataSource() {
        return new EmbeddedDatabaseBuilder()
          .setType(EmbeddedDatabaseType.H2)
          .addScript("schema.sql")
          .addScript("data.sql")
          .build();
    }

    @Bean
    public SqlSessionFactory sqlSessionFactory() throws Exception {
        SqlSessionFactoryBean factoryBean = new SqlSessionFactoryBean();
        factoryBean.setDataSource(dataSource());
        return factoryBean.getObject();
    }
}
```

我们还应用了 MyBatis-Spring 的`@MapperScan`注释，该注释扫描已定义的包，并使用任何映射器注释(如`@Select` 或`@Delete.`)自动选取接口

使用`@MapperScan`还可以确保每个提供的映射器被自动注册为一个 [Bean](/web/20220530025634/https://www.baeldung.com/spring-bean) ，并且以后可以与 [`@Autowired`](/web/20220530025634/https://www.baeldung.com/spring-autowire) 注释一起使用。

我们现在可以创建一个简单的`ArticleMapper`界面:

```java
public interface ArticleMapper {
    @Select("SELECT * FROM ARTICLES WHERE id = #{id}")
    Article getArticle(@Param("id") Long id);
}
```

最后，测试我们的设置:

```java
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(classes = PersistenceConfig.class)
public class ArticleMapperIntegrationTest {

    @Autowired
    ArticleMapper articleMapper;

    @Test
    public void whenRecordsInDatabase_shouldReturnArticleWithGivenId() {
        Article article = articleMapper.getArticle(1L);

        assertThat(article).isNotNull();
        assertThat(article.getId()).isEqualTo(1L);
        assertThat(article.getAuthor()).isEqualTo("Baeldung");
        assertThat(article.getTitle()).isEqualTo("Working with MyBatis in Spring");
    }
}
```

在上面的例子中，我们已经使用 MyBatis 检索了我们之前在`data.sql`文件中插入的唯一记录。

### 3.2。基于 XML 的配置

如前所述，要将 MyBatis 与 Spring 一起使用，我们需要`Datasource`、`SqlSessionFactory`和至少一个映射器。

让我们在`beans.xml`配置文件中创建所需的 bean 定义:

```java
<jdbc:embedded-database id="dataSource" type="H2">
    <jdbc:script location="schema.sql"/>
    <jdbc:script location="data.sql"/>
</jdbc:embedded-database>

<bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
    <property name="dataSource" ref="dataSource" />
</bean>

<bean id="articleMapper" class="org.mybatis.spring.mapper.MapperFactoryBean">
    <property name="mapperInterface" value="com.baeldung.mybatis.ArticleMapper" />
    <property name="sqlSessionFactory" ref="sqlSessionFactory" />
</bean>
```

在这个例子中，我们还使用了由`spring-jdbc`提供的定制 XML 模式来配置我们的 H2 数据源。

为了测试这个配置，我们可以重用之前实现的测试类。但是，我们必须调整上下文配置，这可以通过应用注释来实现:

```java
@ContextConfiguration(locations = "classpath:/beans.xml")
```

## 4。Spring Boot

Spring Boot 提供的机制进一步简化了 MyBatis 与 Spring 的配置。

首先，让我们将`[mybatis-spring-boot-starter](https://web.archive.org/web/20220530025634/https://search.maven.org/search?q=g:org.mybatis.spring.boot%20a:mybatis-spring-boot-starter)`[依赖](https://web.archive.org/web/20220530025634/https://search.maven.org/search?q=g:org.mybatis.spring.boot%20a:mybatis-spring-boot-starter)添加到我们的`pom.xml`中:

```java
<dependency>
    <groupId>org.mybatis.spring.boot</groupId>
    <artifactId>mybatis-spring-boot-starter</artifactId>
    <version>2.1.0</version>
</dependency>
```

默认情况下，如果我们使用自动配置特性， **Spring Boot 会从我们的类路径中检测 H2 依赖项，并为我们配置`Datasource`和`SqlSessionFactory`。**此外，它还在启动时执行`schema.sql `和`data.sql `。

如果我们不使用嵌入式数据库，我们可以通过一个`application.yml`或`application.properties`文件使用配置，或者定义一个指向我们数据库的`Datasource` bean。

我们剩下要做的唯一一件事就是定义一个 mapper 接口，用和以前一样的方式，并用 MyBatis 的`@Mapper`注释对它进行注释。结果，Spring Boot 扫描我们的项目，寻找那个注释，并将我们的映射器注册为 beans。

之后，我们可以通过应用来自`[spring-boot-starter-test](/web/20220530025634/https://www.baeldung.com/spring-boot-testing)`的注释，使用之前定义的测试类来测试我们的配置:

```java
@RunWith(SpringRunner.class)
@SpringBootTest
```

## 5。结论

在本文中，我们探索了用 Spring 配置 MyBatis 的多种方法。

我们查看了使用基于注释和 XML 配置的示例，并展示了 MyBatis 和 Spring Boot 的自动配置特性。

和往常一样，本文中使用的完整代码可以从 GitHub 上的[处获得。](https://web.archive.org/web/20220530025634/https://github.com/eugenp/tutorials/tree/master/persistence-modules/spring-mybatis)