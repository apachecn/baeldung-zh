# 使用 Spring Cloud 的实例概要凭证

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-cloud-instance-profiles>

## 1。简介

在这篇简短的文章中，我们将构建一个 Spring Cloud 应用程序，它使用实例配置文件凭证来连接到一个 S3 存储桶。

## 2。调配我们的云环境

实例配置文件是一个 AWS 特性，它允许 EC2 实例使用临时凭证连接到其他 AWS 资源。这些凭证是短期的，由 AWS 自动轮换。

用户只能从 EC2 实例中请求临时凭据。但是，我们可以在任何地方使用这些凭证，直到它们过期。

要获得更多关于[实例概要配置](https://web.archive.org/web/20220707143819/https://docs.aws.amazon.com/codedeploy/latest/userguide/getting-started-create-iam-instance-profile.html)的帮助，请查阅 AWS 的文档。

### 2.1。部署

首先，我们需要一个具有适当设置的 AWS 环境。

对于下面的代码示例，我们需要建立一个 EC2 实例、一个 S3 桶和适当的 IAM 角色。要做到这一点，我们可以在代码示例中使用[cloud formation 模板](https://web.archive.org/web/20220707143819/https://github.com/eugenp/tutorials/tree/master/spring-cloud/spring-cloud-aws/src/main/resources),或者简单地自己建立这些资源。

### 2.2。验证

接下来，我们应该确保我们的 EC2 实例可以检索实例配置文件凭证。用实际的实例配置文件角色名替换`<InstanceProfileRoleName>`:

```java
curl http://169.254.169.254/latest/meta-data/iam/security-credentials/<InstanceProfileRoleName>
```

如果一切设置正确，那么 JSON 响应将包含`AccessKeyId`、`SecretAccessKey`、`Token`和`Expiration`属性。

## 3。配置 Spring Cloud

现在，对于我们的示例应用程序。我们需要将 Spring Boot 配置为使用实例配置文件，这可以在我们的 Spring Boot 配置文件中完成:

```java
cloud.aws.credentials.instanceProfile=true
```

而且，就是这样！如果这个 Spring Boot 应用程序部署在 EC2 实例中，那么每个客户机将自动尝试使用实例配置文件凭证来连接 AWS 资源。

这是因为 Spring Cloud 使用了 AWS SDK 中的`[EC2ContainerCredentialsProviderWrapper](https://web.archive.org/web/20220707143819/https://docs.aws.amazon.com/AWSJavaSDK/latest/javadoc/com/amazonaws/auth/EC2ContainerCredentialsProviderWrapper.html)`。这将按优先级顺序查找凭证，**如果在系统中找不到任何其他凭证，将自动以实例配置文件凭证结束。**

如果我们需要指定 Spring Cloud 只使用实例配置文件，那么我们可以实例化我们自己的`AmazonS3` 实例。

我们可以用一个`InstanceProfileCredentialsProvider` 来配置它，并将其发布为一个 bean:

```java
@Bean
public AmazonS3 amazonS3() {
    InstanceProfileCredentialsProvider provider
      = new InstanceProfileCredentialsProvider(true);
    return AmazonS3ClientBuilder.standard()
      .withCredentials(provider)
      .build();
}
```

**这将替换 Spring Cloud 提供的默认`AmazonS3` 实例。**

## 4。连接到我们的 S3 桶

现在，我们可以像往常一样使用 Spring Cloud 连接到我们的 S3 服务器，但不需要配置永久凭据:

```java
@Component
public class SpringCloudS3Service {

    // other declarations

    @Autowired
    AmazonS3 amazonS3;

    public void createBucket(String bucketName) {
        // log statement
        amazonS3.createBucket(bucketName);
    }
}
```

请记住，因为实例概要文件只发布给 EC2 实例，**这段代码只在 EC2 实例**上运行时有效。

当然，我们可以对 EC2 实例连接的任何 AWS 服务重复这个过程，包括 EC2、SQS 和 SNS。

## 5。结论

在本教程中，我们看到了如何在 Spring Cloud 中使用实例配置文件凭证。此外，我们创建了一个连接到 S3 桶的简单应用程序。

和往常一样，完整的源代码可以在 GitHub 上找到[。](https://web.archive.org/web/20220707143819/https://github.com/eugenp/tutorials/tree/master/spring-cloud-modules/spring-cloud-aws)