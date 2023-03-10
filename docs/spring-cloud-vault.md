# 《春云穹》简介

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-cloud-vault>

## 1.概观

在本教程中，我们将展示如何在 Spring Boot 应用程序中使用哈希公司的 Vault 来保护敏感的配置数据。

**我们在这里假设有一些保险库知识，并且我们已经建立了一个测试设置并正在运行**。如果不是这样，让我们花一点时间阅读我们的[跳马介绍教程](/web/20221208143837/https://www.baeldung.com/vault)，这样我们就可以熟悉它的基础知识。

## 2.春天云穹

Spring Cloud Vault 是 Spring Cloud 栈的一个相对较新的成员，它允许应用程序以透明的方式访问存储在 Vault 实例中的秘密。

一般来说，迁移到 Vault 是一个非常简单的过程:只需添加所需的库，并向我们的项目添加一些额外的配置属性，我们就可以开始了。不需要修改代码！

这是可能的，因为它作为高优先级`PropertySource` 在当前`Environment`中注册。

因此，只要需要属性，Spring 就会使用它。例子有`DataSource`属性、`ConfigurationProperties,` 等等。

## 3.将 Spring Cloud Vault 添加到 Spring Boot 项目

为了在基于 Maven 的 Spring Boot 项目中包含`spring-cloud-vault`库，我们使用相关的`starter`工件，它将提取所有需要的依赖项。

除了主`starter,` 之外，我们还将包括`spring-vault-config-databases`，它增加了对动态数据库凭证的支持:

```java
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-vault-config</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-vault-config-databases</artifactId>
</dependency>
```

最新版本的 [Spring Cloud Vault starter](https://web.archive.org/web/20221208143837/https://search.maven.org/classic/#search%7Cga%7C1%7Cg%3A%22org.springframework.cloud%22%20AND%20a%3A%22spring-cloud-starter-vault-config%22) 可以从 Maven Central 下载。

### 3.1.基本配置

为了正常工作，Spring Cloud Vault 需要一种方法来确定在哪里联系 Vault 服务器以及如何根据它来验证自己。

我们通过在`application.yml`或`application.properties`中提供必要的信息来做到这一点:

```java
spring:
  cloud:
    vault:
      uri: https://localhost:8200
      ssl:
        trust-store: classpath:/vault.jks
        trust-store-password: changeit
  config:
    import: vault:// 
```

属性指向保险库的 API 地址。因为我们的测试环境使用带有自签名证书的 HTTPS，所以我们还需要提供一个包含其公钥的密钥库。

**注意，该配置没有认证数据**。对于最简单的情况，我们使用固定的令牌，我们可以通过系统属性`spring.cloud.vault.token`或环境变量来传递它。这种方法与标准的云配置机制(如 Kubernetes 的 ConfigMaps 或 Docker secrets)结合使用效果很好。

Spring Vault 还需要为我们希望在应用程序中使用的每种类型的秘密进行额外的配置。下面几节描述了我们如何添加对两种常见秘密类型的支持:键/值和数据库凭证。

## 4.使用通用机密后端

**我们使用通用秘密后端来访问在金库**中作为键值对存储的`unversioned`秘密。

假设在我们的`classpath`中已经有了`spring-cloud-starter-vault-config`依赖项，我们所要做的就是向`application.yml`文件添加一些属性:

```java
spring:
  cloud:
    vault:
      # other vault properties omitted ...
      generic:
        enabled: true
        application-name: fakebank 
```

在这种情况下，属性`application-name` 是可选的。如果没有指定，Spring 将采用标准`spring.application.name`的值。

我们现在可以将存储在`secret/fakebank` 的所有键/值对作为任何其他`Environment`属性`.` 使用。下面的代码片段显示了我们如何读取存储在该路径下的`foo`键的值:

```java
@Autowired Environment env;
public String getFoo() {
    return env.getProperty("foo");
} 
```

**正如我们看到的，代码本身对金库、**一无所知，这是好事！我们仍然可以在本地测试中使用固定属性，并通过在应用程序`.yml`中启用单个属性来随意切换到 Vault。

### 4.1.关于弹簧轮廓的注记

如果在当前的`Environment,` 中可用，Spring Cloud Vault **将使用可用的配置文件名作为后缀附加到指定的基本路径，在该路径中搜索键/值对**。

它还会在一个可配置的默认应用程序路径下查找属性(有和没有配置文件后缀)，这样我们就可以在一个单独的位置共享秘密。请谨慎使用此功能！

总而言之，如果 out `fakebank` 应用程序的`production`配置文件处于活动状态，Spring Vault 将查找存储在以下路径下的属性:

1.  `secret/` fakebank `/production`(优先级更高)
2.  `secret/` fakebank
3.  `secret/application/production`
4.  `secret/application`(低优先级)

在前面的列表中，`application`是 Spring 用作秘密的默认附加位置的名称。我们可以使用`spring.cloud.vault.generic.default-context`属性来修改它。

存储在最具体路径下的属性将优先于其他属性。例如，如果相同的属性`foo` 在上述路径下可用，则优先顺序将是:

## 5.使用数据库机密后端

**数据库后端模块允许 Spring 应用程序使用由 Vault** 创建的动态生成的数据库凭证。Spring Vault 在标准的`spring.datasource.username` 和`spring.datasource.password` 属性下注入这些凭证，这样它们就可以被常规的`DataSource`挑选

请注意，在使用这个后端之前，我们必须在 Vault 中创建一个数据库配置和角色，如我们之前的教程中的[所述。](/web/20221208143837/https://www.baeldung.com/vault)

为了在我们的 Spring 应用程序中使用 Vault 生成的数据库凭证，`spring-cloud-vault-config-databases`必须与相应的 JDBC 驱动程序一起出现在项目的类路径中。

我们还需要通过向我们的`application.yml:`添加一些属性来支持它在我们的应用程序中的使用

```java
spring:
  cloud:
    vault:
      # ... other properties omitted
      database:
        enabled: true
        role: fakebank-accounts-rw
```

这里最重要的属性是`role`属性，它保存存储在 Vault 中的数据库角色名称。在启动过程中，Spring 将联系 Vault，请求它创建具有相应权限的新凭证。

默认情况下，vault 将在配置的生存时间后撤销与这些凭据相关联的权限。

幸运的是， **Spring Vault 将自动更新与获得的凭证相关联的租约。**通过这样做，只要我们的应用程序在运行，凭证就将保持有效。

现在，让我们看看这种集成的实际效果。下面的代码片段从 Spring 管理的`DataSource`获取一个新的数据库连接:

```java
Connection c = datasource.getConnection(); 
```

**再一次，我们可以看到在我们的代码**中没有使用保险库的迹象。所有的集成都发生在`Environment` 级别，所以我们的代码可以像往常一样容易地进行单元测试。

## 6.结论

在本教程中，我们展示了如何使用 Spring Vault 库将 Vault 与 Spring Boot 集成。我们已经讨论了两个常见的用例:通用键/值对和动态数据库凭证。

GitHub 上的[提供了一个包含所有必需的依赖项、集成测试和 vault 设置脚本的示例项目。](https://web.archive.org/web/20221208143837/https://github.com/eugenp/tutorials/tree/master/spring-cloud-modules/spring-cloud-vault)