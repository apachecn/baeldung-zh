# spring Cloud AWS–RDS

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-cloud-aws-rds>

在[之前的文章](/web/20220627083636/https://www.baeldung.com/spring-cloud-aws-ec2)中，我们关注的是 EC2 现在，让我们转到关系数据库服务。

Content Series:[This article is part of a series:](javascript:void(0);)[• Spring Cloud AWS – S3](/web/20220627083636/https://www.baeldung.com/spring-cloud-aws-s3)
[• Spring Cloud AWS – EC2](/web/20220627083636/https://www.baeldung.com/spring-cloud-aws-ec2)
• Spring Cloud AWS – RDS (current article)[• Spring Cloud AWS – Messaging Support](/web/20220627083636/https://www.baeldung.com/spring-cloud-aws-messaging)

## 1。RDS 支持

## 1.1。简单配置

**Spring Cloud AWS 只需指定 RDS 数据库标识符和主密码，就可以自动创建一个`DataSource`。**用户名、JDBC 驱动和完整的 URL 都由 Spring 解析。

如果一个 AWS 帐户有一个 RDS 实例，其 DB 实例标识符为`spring-cloud-test-db`，主密码为`se3retpass`，那么创建一个`DataSource`所需要的就是`application.properties`中的下面一行:

```java
cloud.aws.rds.spring-cloud-test-db.password=se3retpass
```

如果您希望使用 RDS 默认值以外的值，可以添加其他三个属性:

```java
cloud.aws.rds.spring-cloud-test-db.username=testuser
cloud.aws.rds.spring-cloud-test-db.readReplicaSupport=true
cloud.aws.rds.spring-cloud-test-db.databaseName=test
```

## 1.2。自定义数据源

在没有 Spring Boot 的应用程序中，或者在需要定制配置的情况下，**我们也可以使用基于 Java 的配置**来创建`DataSource`:

```java
@Configuration
@EnableRdsInstance(
  dbInstanceIdentifier = "spring-cloud-test-db", 
  password = "se3retpass")
public class SpringRDSSupport {

    @Bean
    public RdsInstanceConfigurer instanceConfigurer() {
        return () -> {
            TomcatJdbcDataSourceFactory dataSourceFactory
             = new TomcatJdbcDataSourceFactory();
            dataSourceFactory.setInitialSize(10);
            dataSourceFactory.setValidationQuery("SELECT 1");
            return dataSourceFactory;
        };
    }
}
```

另外，请注意，我们需要添加正确的 JDBC 驱动程序依赖关系。

## 2。结论

在本文中，我们了解了访问 AWS RDS 服务的各种方法；在系列文章的下一篇也是最后一篇文章[中，我们将了解 AWS 消息支持。](/web/20220627083636/https://www.baeldung.com/spring-cloud-aws-messaging)

像往常一样，这些例子可以在 GitHub 的[上找到。](https://web.archive.org/web/20220627083636/https://github.com/eugenp/tutorials/tree/master/spring-cloud-modules/spring-cloud-aws)

Next **»**[Spring Cloud AWS – Messaging Support](/web/20220627083636/https://www.baeldung.com/spring-cloud-aws-messaging)**«** Previous[Spring Cloud AWS – EC2](/web/20220627083636/https://www.baeldung.com/spring-cloud-aws-ec2)