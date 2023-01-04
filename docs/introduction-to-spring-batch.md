# 春季批次简介

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/introduction-to-spring-batch>

## 1。概述

在本教程中，我们将看一个实用的、以代码为中心的 Spring Batch 介绍。Spring Batch 是一个处理框架，旨在健壮地执行作业。

它目前的版本 4.3 支持 Spring 5 和 Java 8。它还支持 JSR-352，这是用于批处理的新 java 规范。

这里有几个有趣且实用的框架用例。

## 2。工作流程基础知识

Spring Batch 遵循传统的批处理架构，其中作业存储库执行调度工作并与作业交互。

一个作业可以有多个步骤。每一步通常都遵循读取数据、处理数据和写入数据的顺序。

当然，框架将在这里为我们做大部分繁重的工作——特别是当涉及到处理作业的低级持久性工作时——使用`sqlite`作为作业存储库。

### 2.1。示例用例

我们这里要处理的简单用例是将一些金融交易数据从 CSV 迁移到 XML。

输入文件的结构非常简单。

每行包含一笔交易，由用户名、用户 id、交易日期和金额组成:

```java
username, userid, transaction_date, transaction_amount
devendra, 1234, 31/10/2015, 10000
john, 2134, 3/12/2015, 12321
robin, 2134, 2/02/2015, 23411
```

## 3。Maven POM

该项目所需的依赖项是弹簧芯、弹簧批次和`sqlite` JDBC 连接器:

```java
<!-- SQLite database driver -->
<dependency>
    <groupId>org.xerial</groupId>
    <artifactId>sqlite-jdbc</artifactId>
    <version>3.15.1</version>
</dependency>
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-oxm</artifactId>
    <version>5.3.0</version>
</dependency>
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-jdbc</artifactId>
    <version>5.3.0</version>
</dependency>
<dependency>
    <groupId>org.springframework.batch</groupId>
    <artifactId>spring-batch-core</artifactId>
    <version>4.3.0</version>
</dependency>
```

## 4。春季批量配置

我们要做的第一件事是用 XML 配置 Spring Batch:

```java
<!-- connect to SQLite database -->
<bean id="dataSource"
  class="org.springframework.jdbc.datasource.DriverManagerDataSource">
    <property name="driverClassName" value="org.sqlite.JDBC" />
    <property name="url" value="jdbc:sqlite:repository.sqlite" />
    <property name="username" value="" />
    <property name="password" value="" />
</bean>

<!-- create job-meta tables automatically -->
<jdbc:initialize-database data-source="dataSource">
    <jdbc:script
      location="org/springframework/batch/core/schema-drop-sqlite.sql" />
    <jdbc:script location="org/springframework/batch/core/schema-sqlite.sql" />
</jdbc:initialize-database>

<!-- stored job-meta in memory -->
<!-- 
<bean id="jobRepository" 
  class="org.springframework.batch.core.repository.support.MapJobRepositoryFactoryBean"> 
    <property name="transactionManager" ref="transactionManager" />
</bean> 
-->

<!-- stored job-meta in database -->
<bean id="jobRepository"
  class="org.springframework.batch.core.repository.support.JobRepositoryFactoryBean">
    <property name="dataSource" ref="dataSource" />
    <property name="transactionManager" ref="transactionManager" />
    <property name="databaseType" value="sqlite" />
</bean>

<bean id="transactionManager" class=
  "org.springframework.batch.support.transaction.ResourcelessTransactionManager" />

<bean id="jobLauncher"
  class="org.springframework.batch.core.launch.support.SimpleJobLauncher">
    <property name="jobRepository" ref="jobRepository" />
</bean>
```

当然，Java 配置也是可用的:

```java
@Configuration
@EnableBatchProcessing
@Profile("spring")
public class SpringConfig {

    @Value("org/springframework/batch/core/schema-drop-sqlite.sql")
    private Resource dropReopsitoryTables;

    @Value("org/springframework/batch/core/schema-sqlite.sql")
    private Resource dataReopsitorySchema;

    @Bean
    public DataSource dataSource() {
        DriverManagerDataSource dataSource = new DriverManagerDataSource();
        dataSource.setDriverClassName("org.sqlite.JDBC");
        dataSource.setUrl("jdbc:sqlite:repository.sqlite");
        return dataSource;
    }

    @Bean
    public DataSourceInitializer dataSourceInitializer(DataSource dataSource)
      throws MalformedURLException {
        ResourceDatabasePopulator databasePopulator = 
          new ResourceDatabasePopulator();

        databasePopulator.addScript(dropReopsitoryTables);
        databasePopulator.addScript(dataReopsitorySchema);
        databasePopulator.setIgnoreFailedDrops(true);

        DataSourceInitializer initializer = new DataSourceInitializer();
        initializer.setDataSource(dataSource);
        initializer.setDatabasePopulator(databasePopulator);

        return initializer;
    }

    private JobRepository getJobRepository() throws Exception {
        JobRepositoryFactoryBean factory = new JobRepositoryFactoryBean();
        factory.setDataSource(dataSource());
        factory.setTransactionManager(getTransactionManager());
        factory.afterPropertiesSet();
        return (JobRepository) factory.getObject();
    }

    private PlatformTransactionManager getTransactionManager() {
        return new ResourcelessTransactionManager();
    }

    public JobLauncher getJobLauncher() throws Exception {
        SimpleJobLauncher jobLauncher = new SimpleJobLauncher();
        jobLauncher.setJobRepository(getJobRepository());
        jobLauncher.afterPropertiesSet();
        return jobLauncher;
    }
}
```

## 5。春季批处理作业配置

现在让我们为 CSV 到 XML 的工作编写工作描述:

```java
<import resource="spring.xml" />

<bean id="record" class="com.baeldung.spring_batch_intro.model.Transaction"></bean>
<bean id="itemReader"
  class="org.springframework.batch.item.file.FlatFileItemReader">

    <property name="resource" value="input/record.csv" />

    <property name="lineMapper">
        <bean class="org.springframework.batch.item.file.mapping.DefaultLineMapper">
            <property name="lineTokenizer">
                <bean class=
                  "org.springframework.batch.item.file.transform.DelimitedLineTokenizer">
                    <property name="names" value="username,userid,transactiondate,amount" />
                </bean>
            </property>
            <property name="fieldSetMapper">
                <bean class="com.baeldung.spring_batch_intro.service.RecordFieldSetMapper" />
            </property>
        </bean>
    </property>
</bean>

<bean id="itemProcessor"
  class="com.baeldung.spring_batch_intro.service.CustomItemProcessor" />

<bean id="itemWriter"
  class="org.springframework.batch.item.xml.StaxEventItemWriter">
    <property name="resource" value="file:xml/output.xml" />
    <property name="marshaller" ref="recordMarshaller" />
    <property name="rootTagName" value="transactionRecord" />
</bean>

<bean id="recordMarshaller" class="org.springframework.oxm.jaxb.Jaxb2Marshaller">
    <property name="classesToBeBound">
        <list>
            <value>com.baeldung.spring_batch_intro.model.Transaction</value>
        </list>
    </property>
</bean>
<batch:job id="firstBatchJob">
    <batch:step id="step1">
        <batch:tasklet>
            <batch:chunk reader="itemReader" writer="itemWriter"
              processor="itemProcessor" commit-interval="10">
            </batch:chunk>
        </batch:tasklet>
    </batch:step>
</batch:job>
```

下面是类似的基于 Java 的作业配置:

```java
@Profile("spring")
public class SpringBatchConfig {

    @Autowired
    private JobBuilderFactory jobs;

    @Autowired
    private StepBuilderFactory steps;

    @Value("input/record.csv")
    private Resource inputCsv;

    @Value("file:xml/output.xml")
    private Resource outputXml;

    @Bean
    public ItemReader<Transaction> itemReader()
      throws UnexpectedInputException, ParseException {
        FlatFileItemReader<Transaction> reader = new FlatFileItemReader<Transaction>();
        DelimitedLineTokenizer tokenizer = new DelimitedLineTokenizer();
        String[] tokens = { "username", "userid", "transactiondate", "amount" };
        tokenizer.setNames(tokens);
        reader.setResource(inputCsv);
        DefaultLineMapper<Transaction> lineMapper = 
          new DefaultLineMapper<Transaction>();
        lineMapper.setLineTokenizer(tokenizer);
        lineMapper.setFieldSetMapper(new RecordFieldSetMapper());
        reader.setLineMapper(lineMapper);
        return reader;
    }

    @Bean
    public ItemProcessor<Transaction, Transaction> itemProcessor() {
        return new CustomItemProcessor();
    }

    @Bean
    public ItemWriter<Transaction> itemWriter(Marshaller marshaller)
      throws MalformedURLException {
        StaxEventItemWriter<Transaction> itemWriter = 
          new StaxEventItemWriter<Transaction>();
        itemWriter.setMarshaller(marshaller);
        itemWriter.setRootTagName("transactionRecord");
        itemWriter.setResource(outputXml);
        return itemWriter;
    }

    @Bean
    public Marshaller marshaller() {
        Jaxb2Marshaller marshaller = new Jaxb2Marshaller();
        marshaller.setClassesToBeBound(new Class[] { Transaction.class });
        return marshaller;
    }

    @Bean
    protected Step step1(ItemReader<Transaction> reader,
      ItemProcessor<Transaction, Transaction> processor,
      ItemWriter<Transaction> writer) {
        return steps.get("step1").<Transaction, Transaction> chunk(10)
          .reader(reader).processor(processor).writer(writer).build();
    }

    @Bean(name = "firstBatchJob")
    public Job job(@Qualifier("step1") Step step1) {
        return jobs.get("firstBatchJob").start(step1).build();
    }
}
```

现在我们有了完整的配置，让我们把它分解并开始讨论。

### 5.1。用`ItemReader` 读取数据和创建对象

首先，我们配置了从`record.csv`中读取数据并将其转换成`Transaction`对象的`cvsFileItemReader`:

```java
@SuppressWarnings("restriction")
@XmlRootElement(name = "transactionRecord")
public class Transaction {
    private String username;
    private int userId;
    private LocalDateTime transactionDate;
    private double amount;

    /* getters and setters for the attributes */

    @Override
    public String toString() {
        return "Transaction [username=" + username + ", userId=" + userId
          + ", transactionDate=" + transactionDate + ", amount=" + amount
          + "]";
    }
}
```

为此，它使用自定义映射器:

```java
public class RecordFieldSetMapper implements FieldSetMapper<Transaction> {

    public Transaction mapFieldSet(FieldSet fieldSet) throws BindException {
        DateTimeFormatter formatter = DateTimeFormatter.ofPattern("d/M/yyy");
        Transaction transaction = new Transaction();

        transaction.setUsername(fieldSet.readString("username"));
        transaction.setUserId(fieldSet.readInt(1));
        transaction.setAmount(fieldSet.readDouble(3));
        String dateString = fieldSet.readString(2);
        transaction.setTransactionDate(LocalDate.parse(dateString, formatter).atStartOfDay());
        return transaction;
    }
}
```

### 5.2。用`ItemProcessor` 处理数据

我们已经创建了自己的项目处理器`CustomItemProcessor`。这不处理任何与事务对象相关的事情。

它所做的只是将来自阅读器的原始对象传递给编写器:

```java
public class CustomItemProcessor implements ItemProcessor<Transaction, Transaction> {

    public Transaction process(Transaction item) {
        return item;
    }
}
```

### 5.3。用`ItemWriter` 向 FS 中写入对象

最后，我们将把这个`transaction`存储到位于`xml/output.xml`的 XML 文件中:

```java
<bean id="itemWriter"
  class="org.springframework.batch.item.xml.StaxEventItemWriter">
    <property name="resource" value="file:xml/output.xml" />
    <property name="marshaller" ref="recordMarshaller" />
    <property name="rootTagName" value="transactionRecord" />
</bean>
```

### 5.4。配置批处理作业

因此，我们所要做的就是使用`batch:job` 语法将这些点与作业联系起来。

注意`commit-interval`。这是在将批处理提交给`itemWriter`之前要保存在内存中的事务数量。

它会将事务保存在内存中，直到该点(或者直到遇到输入数据的结尾):

```java
<batch:job id="firstBatchJob">
    <batch:step id="step1">
        <batch:tasklet>
            <batch:chunk reader="itemReader" writer="itemWriter"
              processor="itemProcessor" commit-interval="10">
            </batch:chunk>
        </batch:tasklet>
    </batch:step>
</batch:job>
```

### 5.5。运行批处理作业

现在让我们设置并运行一切:

```java
@Profile("spring")
public class App {
    public static void main(String[] args) {
        // Spring Java config
        AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext();
        context.register(SpringConfig.class);
        context.register(SpringBatchConfig.class);
        context.refresh();

        JobLauncher jobLauncher = (JobLauncher) context.getBean("jobLauncher");
        Job job = (Job) context.getBean("firstBatchJob");
        System.out.println("Starting the batch job");
        try {
            JobExecution execution = jobLauncher.run(job, new JobParameters());
            System.out.println("Job Status : " + execution.getStatus());
            System.out.println("Job completed");
        } catch (Exception e) {
            e.printStackTrace();
            System.out.println("Job failed");
        }
    }
}
```

我们使用`-Dspring.profiles.active=spring` profile 运行我们的 Spring 应用程序。

在下一节中，我们将在一个 Spring Boot 应用程序中配置我们的示例。

## 6。Spring Boot 配置

在这一节中，我们将创建一个 Spring Boot 应用程序，并将之前的 Spring 批处理配置转换为在 Spring Boot 环境中运行。实际上，这大致相当于前面的春批示例。

### 6.1。Maven 依赖关系

让我们从在 Spring Boot 应用程序的`pom.xml`中声明 [`spring-boot-starter-batch`](https://web.archive.org/web/20221118053208/https://search.maven.org/search?q=g:org.springframework.boot%20a:spring-boot-starter-batch) 依赖关系开始:

```java
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-batch</artifactId>
</dependency>
```

**我们需要一个数据库来存储春季批量作业信息**。在本教程中，我们使用一个内存中的 [HSQLDB](/web/20221118053208/https://www.baeldung.com/spring-boot-hsqldb) 数据库。因此，我们需要用 [`hsqldb`](https://web.archive.org/web/20221118053208/https://search.maven.org/search?q=g:org.hsqldb%20AND%20a:hsqldb) 搭配 Spring Boot:

```java
<dependency>
    <groupId>org.hsqldb</groupId>
    <artifactId>hsqldb</artifactId>
    <version>2.7.0</version>
    <scope>runtime</scope>
</dependency>
```

### 6.2。Spring Boot 配置

我们使用 [`@Profile`](/web/20221118053208/https://www.baeldung.com/spring-profiles) 注释来区分弹簧和 Spring Boot 配置。我们在应用程序中设置了`spring-boot`配置文件:

```java
@SpringBootApplication
public class SpringBatchApplication {

    public static void main(String[] args) {
        SpringApplication springApp = new SpringApplication(SpringBatchApplication.class);
        springApp.setAdditionalProfiles("spring-boot");
        springApp.run(args);
    }

}
```

### 6.3。春季批处理作业配置

我们使用与前面的`SpringBatchConfig`类相同的批处理作业配置:

```java
@Configuration
@EnableBatchProcessing
@Profile("spring-boot")
public class SpringBootBatchConfig {
    @Autowired
    private JobBuilderFactory jobBuilderFactory;

    @Autowired
    private StepBuilderFactory stepBuilderFactory;

    @Value("input/record.csv")
    private Resource inputCsv;

    @Value("input/recordWithInvalidData.csv")
    private Resource invalidInputCsv;

    @Value("file:xml/output.xml")
    private Resource outputXml;

    // ...
}
```

我们从一个 Spring `@Configuration`注释开始。然后，我们将`@EnableBatchProcessing`注释添加到类中。**`@EnableBatchProcessing`注释自动创建`dataSource` 对象并将其提供给我们的作业**。

## 7 .**。结论**

在本文中，我们学习了如何使用 Spring Batch 以及如何在一个简单的用例中使用它。

我们看到了如何轻松开发批处理管道，以及如何定制读取、处理和写入的不同阶段。

一如既往，本文的完整实现可以在 GitHub 上的[中找到。](https://web.archive.org/web/20221118053208/https://github.com/eugenp/tutorials/tree/master/spring-batch)