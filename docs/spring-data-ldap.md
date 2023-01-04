# Spring 数据 LDAP 指南

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-data-ldap>

## 1。简介

在本文中，**我们将关注 Spring 数据 LDAP 集成和配置。**要逐步了解 Spring LDAP，请快速浏览[这篇文章](/web/20220707143821/https://www.baeldung.com/spring-ldap)。

另外，你可以在这里找到 Spring Data JPA guide [的概述。](/web/20220707143821/https://www.baeldung.com/the-persistence-layer-with-spring-data-jpa)

## 2。 **美芬** **属地**

让我们从添加所需的 Maven 依赖项开始:

```java
<dependency>
    <groupId>org.springframework.data</groupId>
    <artifactId>spring-data-ldap</artifactId>
    <version>1.0.6.RELEASE</version>
</dependency> 
```

这里可以找到 [spring-data-ldap](https://web.archive.org/web/20220707143821/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22org.springframework.data%22%20AND%20a%3A%22spring-data-ldap%22) 的最新版本。

## 3。域条目

Spring LDAP 项目通过使用[对象-目录映射(ODM)](https://web.archive.org/web/20220707143821/https://docs.spring.io/spring-ldap/docs/current-SNAPSHOT/reference/#odm) 提供了将 LDAP 条目映射到 Java 对象的能力。

让我们定义将用于映射 LDAP 目录的实体，这些目录已经在 [Spring LDAP 文章](/web/20220707143821/https://www.baeldung.com/spring-ldap)中进行了配置。

```java
@Entry(
  base = "ou=users", 
  objectClasses = { "person", "inetOrgPerson", "top" })
public class User {
    @Id
    private Name id;

    private @Attribute(name = "cn") String username;
    private @Attribute(name = "sn") String password;

    // standard getters/setters
}
```

`@Entry`类似于`@Entity`(JPA/ORM 的)用于指定哪个实体映射到 LDAP 条目的根目录。

一个`Entry`类必须在代表实体`DN`的 javax `.naming.Name` 类型的字段上声明`@Id` 注释。`@Attribute`注释用于将对象类字段映射到实体字段。

## 4。Spring 数据仓库

Spring Data Repository 是一个抽象，它为各种持久性存储提供了基本的开箱即用的数据访问层实现。

**Spring 框架在内部为数据仓库中的给定类提供 CRUD 操作**的实现。我们可以在[Spring Data JPA](/web/20220707143821/https://www.baeldung.com/the-persistence-layer-with-spring-data-jpa#springdatadao)文章中找到完整的细节。

Spring Data LDAP 提供了类似的抽象，它提供了包括 LDAP 目录的基本 CRUD 操作的`[Repository](https://web.archive.org/web/20220707143821/https://docs.spring.io/spring-data/commons/docs/current/api/org/springframework/data/repository/CrudRepository.html)`接口的自动实现。

此外，Spring Data Framework 可以基于方法名创建一个定制查询。

让我们定义我们的存储库接口，它将用于管理`User Entry:`

```java
@Repository
public interface UserRepository extends LdapRepository<User> {
    User findByUsername(String username);
    User findByUsernameAndPassword(String username, String password);
    List<User> findByUsernameLikeIgnoreCase(String username);
}
```

我们可以看到，我们通过扩展`LdapRepository` 为入口`User.` 声明了一个接口，Spring Data Framework 将自动提供基本的 CRUD 方法实现，如`find()`、`findAll()`、`save(),`、`delete(),`等。

此外，我们还声明了一些自定义方法。Spring Data Framework 将通过使用一种称为[查询构建器机制](https://web.archive.org/web/20220707143821/https://docs.spring.io/spring-data/data-commons/docs/1.6.1.RELEASE/reference/html/repositories.html#repositories.query-methods.query-creation)的策略探测方法名来提供实现。

## 5。配置

我们可以使用基于 Java 的`@Configuration`类或 XML 名称空间来配置 Spring 数据 LDAP。让我们使用基于 Java 的方法来配置存储库:

```java
@Configuration
@EnableLdapRepositories(basePackages = "com.baeldung.ldap.**")
public class AppConfig {
}
```

`@EnableLdapRepositories` 提示 Spring 扫描给定包中标记为`@Repository.`的接口

## 6.使用 Spring Boot

**在 Spring Boot 项目中工作时，我们可以使用 [Spring Boot 启动器数据 Ldap](https://web.archive.org/web/20220707143821/https://search.maven.org/search?q=a:spring-boot-starter-data-ldap) 依赖，它会自动为我们装备`LdapContextSource `和`LdapTemplate `。**

要启用自动配置，我们需要确保在 pom.xml 中将`spring-boot-starter-data-ldap` Starter 或`spring-ldap-core`定义为依赖项:

```java
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-ldap</artifactId>
</dependency>
```

要连接到 LDAP，我们需要在应用程序中提供连接设置。属性:

```java
spring.ldap.url=ldap://localhost:18889
spring.ldap.base=dc=example,dc=com
spring.ldap.username=uid=admin,ou=system
spring.ldap.password=secret
```

关于 Spring 数据 LDAP 自动配置的更多细节可以在[官方文档](https://web.archive.org/web/20220707143821/https://docs.spring.io/spring-boot/docs/current/reference/htmlsingle/#features.nosql.ldap)中找到。Spring Boot 引入了 [LdapAutoConfiguration](https://web.archive.org/web/20220707143821/https://docs.spring.io/spring-boot/docs/current/api/org/springframework/boot/autoconfigure/ldap/LdapAutoConfiguration.html) ，它负责`LdapTemplate` 的检测，然后可以将这些检测注入到所需的服务类中:

```java
@Autowired
private LdapTemplate ldapTemplate;
```

## 7 .**。业务逻辑**

让我们定义我们的服务类，它将使用`UserRepository` 来操作 LDAP 目录:

```java
@Service
public class UserService {
    @Autowired
    private UserRepository userRepository;

    // business methods
}
```

现在，我们将一次研究一个动作，看看使用 Spring Data Repository 执行这些动作有多容易

### 7.1。用户认证

现在让我们实现一个简单的逻辑来验证现有用户:

```java
public Boolean authenticate(String u, String p) {
    return userRepository.findByUsernameAndPassword(u, p) != null;
}
```

### 7.2。用户创建

接下来，让我们创建一个新用户并存储一个密码的哈希:

```java
public void create(String username, String password) {
    User newUser = new User(username,digestSHA(password));
    newUser.setId(LdapUtils.emptyLdapName());
    userRepository.save(newUser);
}
```

### 7.3。用户修改

我们可以使用以下方法修改现有用户或条目:

```java
public void modify(String u, String p) {
    User user = userRepository.findByUsername(u);
    user.setPassword(p);
    userRepository.save(user);
}
```

### 7.4。用户搜索

我们可以使用自定义方法搜索现有用户:

```java
public List<String> search(String u) {
    List<User> userList = userRepository
      .findByUsernameLikeIgnoreCase(u);

    if (userList == null) {
        return Collections.emptyList();
    }

    return userList.stream()
      .map(User::getUsername)
      .collect(Collectors.toList());  
}
```

## 8。行动范例

最后，我们可以快速测试一个简单的身份验证场景:

```java
@Test
public void givenLdapClient_whenCorrectCredentials_thenSuccessfulLogin() {
    Boolean isValid = userService.authenticate(USER3, USER3_PWD);

    assertEquals(true, isValid);
}
```

## 9。结论

这个快速教程演示了 Spring LDAP 存储库配置和 CRUD 操作的基础。

本文中使用的例子可以在 GitHub 上找到[。](https://web.archive.org/web/20220707143821/https://github.com/eugenp/tutorials/tree/master/spring-security-modules/spring-ldap)