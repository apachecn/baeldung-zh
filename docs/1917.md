# 弹簧跳马

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-vault>

## 1。概述

**[HashiCorp 的金库](https://web.archive.org/web/20220627175105/https://www.vaultproject.io/)** 是一个储存和保障机密的工具。总的来说，Vault 解决了如何管理机密的软件开发安全问题。要了解更多，请点击查看[我们的文章。](/web/20220627175105/https://www.baeldung.com/vault)

[**春天金库**](https://web.archive.org/web/20220627175105/https://spring.io/projects/spring-vault) 为哈希公司的金库提供春天抽象。

在本教程中，我们将通过一个例子来说明如何从保险库中存储和检索机密。

## 2。Maven 依赖关系

首先，让我们看看开始使用 Spring Vault 所需的依赖项:

```
<dependencies>
    <dependency>
        <groupId>org.springframework.vault</groupId>
        <artifactId>spring-vault-core</artifactId>
        <version>2.3.2</version>
    </dependency>
</dependencies> 
```

最新版本的`spring-vault-core`可以在 [Maven Central](https://web.archive.org/web/20220627175105/https://search.maven.org/search?q=g:org.springframework.vault%20AND%20a:spring-vault-core&core=gav) 上找到。

## 3。配置保险库

现在，我们来看一下配置 Vault 所需的步骤。

### 3.1。创造一个`VaultTemplate`

为了保护我们的秘密，我们必须实例化一个`VaultTemplate`，为此我们需要`VaultEndpoint`和`TokenAuthentication`实例:

```
VaultTemplate vaultTemplate = new VaultTemplate(new VaultEndpoint(), 
  new TokenAuthentication("00000000-0000-0000-0000-000000000000"));
```

### 3.2。创造一个`VaultEndpoint`

有几种方法可以实例化`VaultEndpoint`。让我们来看看其中的一些。

第一种是使用默认构造函数简单地实例化它，这将创建一个指向`http://localhost:8200:`的默认端点

```
VaultEndpoint endpoint = new VaultEndpoint();
```

另一种方法是通过指定存储库的主机和端口来创建`VaultEndpoint`:

```
VaultEndpoint endpoint = VaultEndpoint.create("host", port);
```

最后，我们还可以从存储库 URL 创建它:

```
VaultEndpoint endpoint = VaultEndpoint.from(new URI("vault uri"));
```

这里需要注意一些事情——Vault 将配置一个根令牌`00000000-0000-0000-0000-000000000000`来运行这个应用程序。

在我们的例子中，我们使用了`TokenAuthentication`，但是也支持其他的[认证方法](https://web.archive.org/web/20220627175105/https://docs.spring.io/spring-vault/docs/current/reference/html/index.html#vault.core.authentication)。

## 4。使用 Spring 配置 Vault Beans】

使用 Spring，我们可以用几种方式配置 Vault。一个是通过扩展`AbstractVaultConfiguration,`，另一个是通过使用`EnvironmentVaultConfiguration `，它利用了 Spring 的环境属性。

我们现在将两种方式都过一遍。

### 4.1。使用`AbstractVaultConfiguration`

让我们创建一个扩展`AbstractVaultConfiguration,`来配置 Spring Vault 的类:

```
@Configuration
public class VaultConfig extends AbstractVaultConfiguration {

    @Override
    public ClientAuthentication clientAuthentication() {
        return new TokenAuthentication("00000000-0000-0000-0000-000000000000");
    }

    @Override
    public VaultEndpoint vaultEndpoint() {
        return VaultEndpoint.create("host", 8020);
    }
}
```

这种方法类似于我们在上一节中看到的。不同的是，我们使用 Spring Vault 通过扩展抽象类`AbstractVaultConfiguration.`来配置 Vault beans

我们只需要提供配置`VaultEndpoint`和`ClientAuthentication`的实现。

### 4.2。使用`EnvironmentVaultConfiguration`

我们还可以使用`EnviromentVaultConfiguration`来配置 Spring Vault:

```
@Configuration
@PropertySource(value = { "vault-config.properties" })
@Import(value = EnvironmentVaultConfiguration.class)
public class VaultEnvironmentConfig {
}
```

`EnvironmentVaultConfiguration`利用 Spring 的`PropertySource`来配置 Vault beans。我们只需要为属性文件提供一些可接受的条目。

关于所有预定义属性的更多信息可以在官方文档中找到。

要配置存储库，我们至少需要几个属性:

```
vault.uri=https://localhost:8200
vault.token=00000000-0000-0000-0000-000000000000
```

## 5。保护机密

我们将创建一个简单的`Credentials`类，它映射到用户名和密码:

```
public class Credentials {

    private String username;
    private String password;

    // standard constructors, getters, setters
}
```

现在，让我们看看如何使用`VaultTemplate:`来保护我们的`Credentials`对象

```
Credentials credentials = new Credentials("username", "password");
vaultTemplate.write("secret/myapp", credentials);
```

随着这些线条的完成，我们的秘密被储存起来了。

接下来，我们将看到如何访问它们。

## 6。访问机密

我们可以使用`VaultTemplate,` 中的`read() `方法访问受保护的秘密，该方法返回` VaultResponseSupport`作为响应:

```
VaultResponseSupport<Credentials> response = vaultTemplate
  .read("secret/myapp", Credentials.class);
String username = response.getData().getUsername();
String password = response.getData().getPassword();
```

我们的秘密值现在准备好了。

## 7.保管库仓库

Vault repository 是 Spring Vault 2.0 附带的一个便利特性。**它在保险库**之上应用了 [Spring 数据仓库](/web/20220627175105/https://www.baeldung.com/the-persistence-layer-with-spring-data-jpa)的概念。

让我们深入研究一下如何在实践中使用这个新特性。

### 7.1.`@Secret`和`@Id`注释

Spring 提供了这两个注释来标记我们想要在 Vault 中持久化的对象。

所以首先，我们需要修饰我们的域类型`Credentials`:

```
@Secret(backend = "credentials", value = "myapp")
public class Credentials {

    @Id
    private String username;
    // Same code
]
```

`@Secret`注释的 `value` 属性用于区分域类型。`backend`属性表示秘密的后端挂载。

另一方面，`@Id`只是简单地划定了我们对象的标识符。

### 7.2.保管库储存库

现在，让我们定义一个使用我们的域对象`Credentials`的存储库接口:

```
public interface CredentialsRepository extends CrudRepository<Credentials, String> {
}
```

正如我们所见，**我们的存储库扩展了`CrudRepository`，它提供了基本的 CRUD 和查询方法**。

接下来，让我们将`CredentialsRepository`注入到`CredentialsService`中，并实现一些 CRUD 方法:

```
public class CredentialsService {

    @Autowired
    private CredentialsRepository credentialsRepository;

    public Credentials saveCredentials(Credentials credentials) {
        return credentialsRepository.save(credentials);
    }

    public Optional<Credentials> findById(String username) {
        return credentialsRepository.findById(username);
    }
}
```

现在我们已经添加了所有缺失的部分，让我们使用测试用例来确认一切都正常工作。

首先，让我们从`save()`方法的测试用例开始:

```
@Test
public void givenCredentials_whenSave_thenReturnCredentials() {
    // Given
    Credentials credentials = new Credentials("login", "password");
    Mockito.when(credentialsRepository.save(credentials))
      .thenReturn(credentials);

    // When
    Credentials savedCredentials = credentialsService.saveCredentials(credentials);

    // Then
    assertNotNull(savedCredentials);
    assertEquals(savedCredentials.getUsername(), credentials.getUsername());
    assertEquals(savedCredentials.getPassword(), credentials.getPassword());
}
```

最后，让我们用一个测试案例来确认`findById()`方法:

```
@Test
public void givenId_whenFindById_thenReturnCredentials() {
    // Given
    Credentials credentials = new Credentials("login", "[[email protected]](/web/20220627175105/https://www.baeldung.com/cdn-cgi/l/email-protection)@rd");
    Mockito.when(credentialsRepository.findById("login"))
      .thenReturn(Optional.of(credentials));

    // When
    Optional<Credentials> returnedCredentials = credentialsService.findById("login");

    // Then
    assertNotNull(returnedCredentials);
    assertNotNull(returnedCredentials.get());
    assertEquals(returnedCredentials.get().getUsername(), credentials.getUsername());
    assertEquals(returnedCredentials.get().getPassword(), credentials.getPassword());
}
```

## 8。结论

在本文中，我们学习了 Spring Vault 的基础知识，并通过一个示例展示了 Vault 在典型场景**中的工作方式。**

像往常一样，这里展示的源代码可以在 GitHub 上找到[。](https://web.archive.org/web/20220627175105/https://github.com/eugenp/tutorials/tree/master/spring-vault)