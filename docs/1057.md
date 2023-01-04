# spring Cloud AWS–EC2

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-cloud-aws-ec2>

在[上一篇文章](/web/20220625233808/https://www.baeldung.com/spring-cloud-aws-s3)中，我们关注的是 S3；现在，我们将重点关注弹性计算云，通常称为 EC2。

Content Series:[This article is part of a series:](javascript:void(0);)[• Spring Cloud AWS – S3](/web/20220625233808/https://www.baeldung.com/spring-cloud-aws-s3)
• Spring Cloud AWS – EC2 (current article)[• Spring Cloud AWS – RDS](/web/20220625233808/https://www.baeldung.com/spring-cloud-aws-rds)
[• Spring Cloud AWS – Messaging Support](/web/20220625233808/https://www.baeldung.com/spring-cloud-aws-messaging)

## 1。EC2 元数据访问

AWS [`EC2MetadataUtils`](https://web.archive.org/web/20220625233808/https://docs.aws.amazon.com/AWSJavaSDK/latest/javadoc/com/amazonaws/util/EC2MetadataUtils.html) 类提供静态方法来访问实例元数据，如 AMI Id 和实例类型。**使用 Spring Cloud AWS，我们可以使用`@Value`注释**直接注入这些元数据。

这可以通过在任何配置类上添加`@EnableContextInstanceData`注释来实现:

```
@Configuration
@EnableContextInstanceData
public class EC2EnableMetadata {
    //
}
```

**在 Spring Boot 环境中，默认情况下启用实例元数据，这意味着不需要此配置**。

然后，我们可以注入这些值:

```
@Value("${ami-id}")
private String amiId;

@Value("${hostname}")
private String hostname;

@Value("${instance-type}")
private String instanceType;

@Value("${services/domain}")
private String serviceDomain;
```

### 1.1。自定义标签

此外，Spring 还支持注入用户自定义的[标签](https://web.archive.org/web/20220625233808/https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/Using_Tags.html)。我们可以通过使用以下 XML 配置在`context-instance-data`中定义属性`user-tags-map`来实现这一点:

```
<beans...>
    <aws-context:context-instance-data user-tags-map="instanceData"/>
</beans>
```

现在，让我们借助 Spring 表达式语法注入用户定义的标签:

```
@Value("#{instanceData.myTagKey}")
private String myTagValue;
```

## 2。EC2 客户端

此外，如果为实例配置了用户标签，Spring 将创建一个`AmazonEC2`客户端，我们可以使用`@Autowired`将它注入到我们的代码中:

```
@Autowired
private AmazonEC2 amazonEc2;
```

请注意，只有当应用程序运行在 EC2 实例上时，这些功能才起作用。

## 3。结论

这是关于使用 Spring Cloud AWS 访问 EC2d 数据的快速而中肯的介绍。

在本系列的下一篇文章中，我们将探索 RDS 支持。

像往常一样，这些例子可以在 GitHub 的[上找到。](https://web.archive.org/web/20220625233808/https://github.com/eugenp/tutorials/tree/master/spring-cloud-modules/spring-cloud-aws)

Next **»**[Spring Cloud AWS – RDS](/web/20220625233808/https://www.baeldung.com/spring-cloud-aws-rds)**«** Previous[Spring Cloud AWS – S3](/web/20220625233808/https://www.baeldung.com/spring-cloud-aws-s3)