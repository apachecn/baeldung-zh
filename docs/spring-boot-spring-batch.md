# Spring Boot 春季批次

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-boot-spring-batch>

## 1。概述

Spring Batch 是一个强大的框架，用于开发健壮的批处理应用程序。在我们之前的教程中，我们[介绍了 Spring Batch](/web/20220626090639/https://www.baeldung.com/introduction-to-spring-batch) 。

在本教程中，我们将建立在前一个的基础上，学习如何使用 [Spring Boot](/web/20220626090639/https://www.baeldung.com/category/spring/spring-boot/) 设置和创建一个基本的批处理驱动应用程序。

## 2。Maven 依赖关系

首先，让我们将 [`spring-boot-starter-batch`](https://web.archive.org/web/20220626090639/https://search.maven.org/classic/#search%7Cga%7C1%7Cg%3Aorg.springframework.boot%20a%3Aspring-boot-starter-batch) 添加到我们的`pom.xml`中:

```java
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-batch</artifactId>
    <version>2.4.0</version>
</dependency>
```

我们还将添加`org.hsqldb` 依赖项，它也可以从 [Maven Central](https://web.archive.org/web/20220626090639/https://search.maven.org/classic/#search%7Cga%7C1%7Cg%3A%22org.hsqldb%22%20AND%20a%3A%22hsqldb%22) 获得:

```java
<dependency>
    <groupId>org.hsqldb</groupId>
    <artifactId>hsqldb</artifactId>
    <version>2.5.1</version>
    <scope>runtime</scope>
</dependency>
```

## 3。定义一个简单的 Spring 批处理作业

**我们将构建一个作业，从 CSV 文件导入咖啡清单，使用定制的处理器对其进行转换，并将最终结果存储在内存数据库中**。

### 3.1.入门指南

让我们从定义应用程序入口点开始:

```java
@SpringBootApplication
public class SpringBootBatchProcessingApplication {

    public static void main(String[] args) {
        SpringApplication.run(SpringBootBatchProcessingApplication.class, args);
    }
}
```

正如我们所看到的，这是一个标准的 Spring Boot 应用程序。因为我们希望尽可能使用默认配置值，所以我们将使用一组非常简单的应用程序配置属性。

我们将在我们的`src/main/resources/application.properties`文件中定义这些属性:

```java
file.input=coffee-list.csv
```

该属性包含我们输入咖啡列表的位置。每一行都包含品牌、产地和我们咖啡的一些特征:

```java
Blue Mountain,Jamaica,Fruity
Lavazza,Colombia,Strong
Folgers,America,Smokey
```

正如我们将要看到的，这是一个平面 CSV 文件，这意味着 Spring 无需任何特殊定制就可以处理它。

接下来，我们将添加一个 SQL 脚本`schema-all.sql`来创建我们的`coffee`表来存储数据:

```java
DROP TABLE coffee IF EXISTS;

CREATE TABLE coffee  (
    coffee_id BIGINT IDENTITY NOT NULL PRIMARY KEY,
    brand VARCHAR(20),
    origin VARCHAR(20),
    characteristics VARCHAR(30)
);
```

为了方便起见，Spring Boot 会在启动时自动运行这个脚本。

### 3.2.咖啡领域类

随后，我们需要一个简单的域类来保存我们的咖啡项目:

```java
public class Coffee {

    private String brand;
    private String origin;
    private String characteristics;

    public Coffee(String brand, String origin, String characteristics) {
        this.brand = brand;
        this.origin = origin;
        this.characteristics = characteristics;
    }

    // getters and setters
}
```

如前所述，我们的`Coffee`对象包含三个属性:

*   一个品牌
*   一个起源
*   一些附加特征

## 4。作业配置

现在，我们来看看关键部分，我们的工作配置。我们将一步一步来，逐步构建我们的配置并解释每一部分:

```java
@Configuration
@EnableBatchProcessing
public class BatchConfiguration {

    @Autowired
    public JobBuilderFactory jobBuilderFactory;

    @Autowired
    public StepBuilderFactory stepBuilderFactory;

    @Value("${file.input}")
    private String fileInput;

    // ...
}
```

首先，我们从一个标准的 Spring `[@Configuration](/web/20220626090639/https://www.baeldung.com/configuration-properties-in-spring-boot)`类开始。接下来，我们向我们的类添加一个`@EnableBatchProcessing`注释。**值得注意的是，这让我们可以获得许多支持就业的有用的 beans，并为我们节省大量的跑腿工作。**

此外，使用该注释还为我们提供了对两个有用工厂的访问，我们稍后将在构建作业配置和作业步骤时使用这两个工厂。

对于初始配置的最后一部分，我们包含了对之前声明的`file.input`属性的引用。

### 4.1.我们工作的读者和作者

现在，我们可以在配置中定义一个阅读器 bean:

```java
@Bean
public FlatFileItemReader reader() {
    return new FlatFileItemReaderBuilder().name("coffeeItemReader")
      .resource(new ClassPathResource(fileInput))
      .delimited()
      .names(new String[] { "brand", "origin", "characteristics" })
      .fieldSetMapper(new BeanWrapperFieldSetMapper() {{
          setTargetType(Coffee.class);
      }})
      .build();
}
```

简而言之，**我们上面定义的 reader bean 寻找一个名为`coffee-list.csv`的文件，并将每个行项目解析成一个`Coffee`对象**。

同样，我们定义了一个编写器 bean:

```java
@Bean
public JdbcBatchItemWriter writer(DataSource dataSource) {
    return new JdbcBatchItemWriterBuilder()
      .itemSqlParameterSourceProvider(new BeanPropertyItemSqlParameterSourceProvider<>())
      .sql("INSERT INTO coffee (brand, origin, characteristics) VALUES (:brand, :origin, :characteristics)")
      .dataSource(dataSource)
      .build();
}
```

这一次，我们包含了将一个咖啡项目插入数据库所需的 SQL 语句，由我们的`Coffee`对象的 Java bean 属性驱动。**很方便的是`dataSource`是由`@EnableBatchProcessing`标注**自动生成的。

### 4.2.整合我们的工作

最后，我们需要添加实际的作业步骤和配置:

```java
@Bean
public Job importUserJob(JobCompletionNotificationListener listener, Step step1) {
    return jobBuilderFactory.get("importUserJob")
      .incrementer(new RunIdIncrementer())
      .listener(listener)
      .flow(step1)
      .end()
      .build();
}

@Bean
public Step step1(JdbcBatchItemWriter writer) {
    return stepBuilderFactory.get("step1")
      .<Coffee, Coffee> chunk(10)
      .reader(reader())
      .processor(processor())
      .writer(writer)
      .build();
}

@Bean
public CoffeeItemProcessor processor() {
    return new CoffeeItemProcessor();
}
```

正如我们所看到的，我们的工作相对简单，由`step1`方法中定义的一个步骤组成。

让我们来看看这一步在做什么:

*   首先，我们配置我们的步骤，这样它将使用`chunk(10)`声明一次写入多达 10 条记录
*   然后，我们使用 reader bean 读入咖啡数据，这是使用`reader`方法设置的
*   接下来，我们将每个咖啡项目传递给一个定制的处理器，在那里我们应用一些定制的业务逻辑
*   最后，我们使用前面看到的编写器将每个咖啡项目写入数据库

另一方面，我们的`importUserJob`包含我们的作业定义，它包含一个使用内置`RunIdIncrementer`类的 id。**我们还设置了一个`JobCompletionNotificationListener,`，用于在任务完成时得到通知**。

为了完成我们的作业配置，我们列出了每个步骤(尽管这个作业只有一个步骤)。我们现在有了一个完美配置的作业！

## 5。定制咖啡处理器

让我们详细了解一下我们之前在作业配置中定义的自定义处理器:

```java
public class CoffeeItemProcessor implements ItemProcessor<Coffee, Coffee> {

    private static final Logger LOGGER = LoggerFactory.getLogger(CoffeeItemProcessor.class);

    @Override
    public Coffee process(final Coffee coffee) throws Exception {
        String brand = coffee.getBrand().toUpperCase();
        String origin = coffee.getOrigin().toUpperCase();
        String chracteristics = coffee.getCharacteristics().toUpperCase();

        Coffee transformedCoffee = new Coffee(brand, origin, chracteristics);
        LOGGER.info("Converting ( {} ) into ( {} )", coffee, transformedCoffee);

        return transformedCoffee;
    }
}
```

特别有趣的是，`ItemProcessor`接口为我们提供了一种在工作执行期间应用一些特定业务逻辑的机制。

为了简单起见，**我们定义了我们的`CoffeeItemProcessor`，它接受一个输入`Coffee`对象并将每个属性转换成大写的**。

## 6。工作完成

此外，我们还将编写一个`JobCompletionNotificationListener `来在我们的工作完成时提供一些反馈:

```java
@Override
public void afterJob(JobExecution jobExecution) {
    if (jobExecution.getStatus() == BatchStatus.COMPLETED) {
        LOGGER.info("!!! JOB FINISHED! Time to verify the results");

        String query = "SELECT brand, origin, characteristics FROM coffee";
        jdbcTemplate.query(query, (rs, row) -> new Coffee(rs.getString(1), rs.getString(2), rs.getString(3)))
          .forEach(coffee -> LOGGER.info("Found < {} > in the database.", coffee));
    }
}
```

在上面的例子中，我们覆盖了`afterJob`方法，并检查作业是否成功完成。**此外，我们运行一个简单的查询来检查每个咖啡项目是否成功地存储在数据库中**。

## 7.管理我们的工作

现在我们已经准备好了运行我们工作的一切，有趣的部分来了。让我们继续运行我们的作业:

```java
...
17:41:16.336 [main] INFO  c.b.b.JobCompletionNotificationListener -
  !!! JOB FINISHED! Time to verify the results
17:41:16.336 [main] INFO  c.b.b.JobCompletionNotificationListener -
  Found < Coffee [brand=BLUE MOUNTAIN, origin=JAMAICA, characteristics=FRUITY] > in the database.
17:41:16.337 [main] INFO  c.b.b.JobCompletionNotificationListener -
  Found < Coffee [brand=LAVAZZA, origin=COLOMBIA, characteristics=STRONG] > in the database.
17:41:16.337 [main] INFO  c.b.b.JobCompletionNotificationListener -
  Found < Coffee [brand=FOLGERS, origin=AMERICA, characteristics=SMOKEY] > in the database.
... 
```

**如我们所见，我们的作业运行成功，每一种咖啡都如预期的那样存储在数据库中**。

## 8.结论

在本文中，我们学习了如何使用 Spring Boot 创建一个简单的 Spring 批处理作业。首先，我们从定义一些基本配置开始。

然后，我们看到了如何添加文件读取器和数据库写入器。最后，我们看了看如何应用一些自定义处理，并检查我们的作业是否成功执行。

和往常一样，GitHub 上的[提供了这篇文章的完整源代码。](https://web.archive.org/web/20220626090639/https://github.com/eugenp/tutorials/tree/master/spring-batch-2)