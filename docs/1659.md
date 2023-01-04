# Spring LDAP 概述

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-ldap>

## 1。概述

LDAP 目录服务器是读取优化的分层数据存储。通常，它们用于存储用户身份验证和授权所需的用户相关信息。

在本文中，我们将探索 Spring LDAP APIs 来验证和搜索用户，以及在目录服务器中创建和修改用户。同一组 API 可用于管理 LDAP 中任何其他类型的条目。

## 2。Maven 依赖关系

让我们从添加所需的 Maven 依赖项开始:

```
<dependency>
    <groupId>org.springframework.ldap</groupId>
    <artifactId>spring-ldap-core</artifactId>
    <version>2.3.6.RELEASE</version>
</dependency>
```

这个依赖关系的最新版本可以在 spring-ldap-core 中找到。

## 3。数据准备

出于本文的目的，让我们首先创建以下 LDAP 条目:

```
ou=users,dc=example,dc=com (objectClass=organizationalUnit)
```

在此节点下，我们将创建新用户、修改现有用户、验证现有用户和搜索信息。

## 4。spring LDAP API

### 4.1。`ContextSource` & `LdapTemplate` Bean 定义

`ContextSource`用于创建`LdapTemplate`。在下一节中，我们将看到在用户认证过程中使用`ContextSource`:

```
@Bean
public LdapContextSource contextSource() {
    LdapContextSource contextSource = new LdapContextSource();

    contextSource.setUrl(env.getRequiredProperty("ldap.url"));
    contextSource.setBase(
      env.getRequiredProperty("ldap.partitionSuffix"));
    contextSource.setUserDn(
      env.getRequiredProperty("ldap.principal"));
    contextSource.setPassword(
      env.getRequiredProperty("ldap.password"));

    return contextSource;
}
```

`LdapTemplate`用于创建和修改 LDAP 条目:

```
@Bean
public LdapTemplate ldapTemplate() {
    return new LdapTemplate(contextSource());
}
```

### 4.2。使用 Spring Boot

**当我们在 Spring Boot 项目上工作时，我们可以使用 [Spring Boot 启动器数据 Ldap](https://web.archive.org/web/20220921015126/https://search.maven.org/search?q=a:spring-boot-starter-data-ldap) 依赖，它将自动为我们装备`LdapContextSource `和`LdapTemplate `。**

要启用自动配置，我们需要确保在 pom.xml 中将`spring-boot-starter-data-ldap` Starter 或`spring-ldap-core`定义为依赖项:

```
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-ldap</artifactId>
</dependency>
```

要连接到 LDAP，我们需要在应用程序中提供连接设置。属性:

```
spring.ldap.url=ldap://localhost:18889
spring.ldap.base=dc=example,dc=com
spring.ldap.username=uid=admin,ou=system
spring.ldap.password=secret
```

然后我们就可以将自动配置的`LdapTemplate`注入到所需的服务类中了。

```
@Autowired
private LdapTemplate ldapTemplate;
```

### 4.3。用户认证

现在让我们实现一个简单的逻辑来验证现有用户:

```
public void authenticate(String username, String password) {
    contextSource
      .getContext(
        "cn=" + 
         username + 
         ",ou=users," + 
         env.getRequiredProperty("ldap.partitionSuffix"), password);
}
```

### 4.4。用户创建

接下来，让我们创建一个新用户，并将密码的 SHA 散列存储在 LDAP 中。

在身份验证时，LDAP 服务器会生成所提供密码的 SHA 哈希，并将其与存储的密码进行比较:

```
public void create(String username, String password) {
    Name dn = LdapNameBuilder
      .newInstance()
      .add("ou", "users")
      .add("cn", username)
      .build();
    DirContextAdapter context = new DirContextAdapter(dn);

    context.setAttributeValues(
      "objectclass", 
      new String[] 
        { "top", 
          "person", 
          "organizationalPerson", 
          "inetOrgPerson" });
    context.setAttributeValue("cn", username);
    context.setAttributeValue("sn", username);
    context.setAttributeValue
      ("userPassword", digestSHA(password));

    ldapTemplate.bind(context);
}
```

`digestSHA()`是一个自定义方法，它返回所提供密码的 SHA 哈希的 Base64 编码字符串。

最后，使用`LdapTemplate`的`bind()`方法在 LDAP 服务器中创建一个条目。

### 4.5。用户修改

我们可以使用以下方法修改现有用户或条目:

```
public void modify(String username, String password) {
    Name dn = LdapNameBuilder.newInstance()
      .add("ou", "users")
      .add("cn", username)
      .build();
    DirContextOperations context 
      = ldapTemplate.lookupContext(dn);

    context.setAttributeValues
      ("objectclass", 
          new String[] 
            { "top", 
              "person", 
              "organizationalPerson", 
              "inetOrgPerson" });
    context.setAttributeValue("cn", username);
    context.setAttributeValue("sn", username);
    context.setAttributeValue("userPassword", 
      digestSHA(password));

    ldapTemplate.modifyAttributes(context);
}
```

`lookupContext()`方法用于查找所提供的用户。

### 4.6。用户搜索

我们可以使用搜索过滤器搜索现有用户:

```
public List<String> search(String username) {
    return ldapTemplate
      .search(
        "ou=users", 
        "cn=" + username, 
        (AttributesMapper<String>) attrs -> (String) attrs.get("cn").get());
}
```

`AttributesMapper`用于从找到的条目中获取所需的属性值。在内部，Spring `LdapTemplate`为找到的所有条目调用`AttributesMapper`，并创建一个属性值列表。

## 5。测试

`spring-ldap-test`提供基于 ApacheDS 1.5.5 的嵌入式 LDAP 服务器。为了设置嵌入式 LDAP 服务器进行测试，我们需要配置以下 Spring bean:

```
@Bean
public TestContextSourceFactoryBean testContextSource() {
    TestContextSourceFactoryBean contextSource 
      = new TestContextSourceFactoryBean();

    contextSource.setDefaultPartitionName(
      env.getRequiredProperty("ldap.partition"));
    contextSource.setDefaultPartitionSuffix(
      env.getRequiredProperty("ldap.partitionSuffix"));
    contextSource.setPrincipal(
      env.getRequiredProperty("ldap.principal"));
    contextSource.setPassword(
      env.getRequiredProperty("ldap.password"));
    contextSource.setLdifFile(
      resourceLoader.getResource(
        env.getRequiredProperty("ldap.ldiffile")));
    contextSource.setPort(
      Integer.valueOf(
        env.getRequiredProperty("ldap.port")));
    return contextSource;
}
```

让我们用 JUnit 测试我们的用户搜索方法:

```
@Test
public void 
  givenLdapClient_whenCorrectSearchFilter_thenEntriesReturned() {
    List<String> users = ldapClient
      .search(SEARCH_STRING);

    assertThat(users, Matchers.containsInAnyOrder(USER2, USER3));
}
```

## 6。结论

在本文中，我们介绍了 Spring LDAP APIs，并开发了在 LDAP 服务器中进行用户认证、用户搜索、用户创建和修改的简单方法。

与往常一样，Github 项目的[中提供了完整的源代码。测试是在 Maven 概要文件“live”下创建的，因此可以使用选项“-P live”运行。](https://web.archive.org/web/20220921015126/https://github.com/eugenp/tutorials/tree/master/spring-security-modules/spring-security-ldap)