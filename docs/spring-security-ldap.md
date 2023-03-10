# Spring 安全 LDAP 简介

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-security-ldap>

## 1。概述

在这个快速教程中，我们将学习如何设置 Spring 安全 LDAP。

在我们开始之前，先说明一下 LDAP 是什么——它代表轻量级目录访问协议，是一个开放的、供应商中立的协议，用于通过网络访问目录服务。

## 延伸阅读:

## [Spring LDAP 概述](/web/20220902041127/https://www.baeldung.com/spring-ldap)

Learn how to use the Spring LDAP APIs to authenticate and search for users, as well as to create and modify users in the directory server.[Read more](/web/20220902041127/https://www.baeldung.com/spring-ldap) →

## [Spring 数据 LDAP 指南](/web/20220902041127/https://www.baeldung.com/spring-data-ldap)

Learn how to use Spring Data with LDAP.[Read more](/web/20220902041127/https://www.baeldung.com/spring-data-ldap) →

## [带弹簧安全的弹簧数据](/web/20220902041127/https://www.baeldung.com/spring-data-security)

See how to integrate Spring Data with Spring Security.[Read more](/web/20220902041127/https://www.baeldung.com/spring-data-security) →

## 2。Maven 依赖关系

首先，让我们看看我们需要的 maven 依赖项:

```java
<dependency>
    <groupId>org.springframework.security</groupId>
    <artifactId>spring-security-ldap</artifactId>
</dependency>

<dependency>
    <groupId>org.apache.directory.server</groupId>
    <artifactId>apacheds-server-jndi</artifactId>
    <version>1.5.5</version>
</dependency>
```

注意:我们使用 [ApacheDS](https://web.archive.org/web/20220902041127/https://directory.apache.org/apacheds/) 作为我们的 LDAP 服务器，它是一个可扩展和可嵌入的目录服务器。

## 3。Java 配置

接下来，让我们讨论我们的 Spring Security Java 配置:

```java
public class SecurityConfig extends WebSecurityConfigurerAdapter {

    @Override
    protected void configure(AuthenticationManagerBuilder auth) throws Exception {
        auth.ldapAuthentication()
            .userSearchBase("ou=people")
            .userSearchFilter("(uid={0})")
            .groupSearchBase("ou=groups")
            .groupSearchFilter("member={0}")
            .contextSource()
            .root("dc=baeldung,dc=com")
            .ldif("classpath:users.ldif");
    }
}
```

当然，这只是配置中与 LDAP 相关的部分——完整的 Java 配置可以在[这里](https://web.archive.org/web/20220902041127/https://github.com/eugenp/tutorials/blob/master/spring-security-modules/spring-security-ldap/src/main/java/com/baeldung/config/SecurityConfig.java)找到。

## 4。XML 配置

现在，让我们看看相应的 XML 配置:

```java
<authentication-manager>
    <ldap-authentication-provider
      user-search-base="ou=people"
      user-search-filter="(uid={0})"
      group-search-base="ou=groups"
      group-search-filter="(member={0})">
    </ldap-authentication-provider>
</authentication-manager>

<ldap-server root="dc=baeldung,dc=com" ldif="users.ldif"/>
```

同样，这只是配置的一部分——与 LDAP 相关的部分；完整的 XML 配置可以在这里找到。

## 5。LDAP 数据交换格式

LDAP 数据可以用 LDAP 数据交换格式(LDIF)来表示——下面是我们的用户数据的一个例子:

```java
dn: ou=groups,dc=baeldung,dc=com
objectclass: top
objectclass: organizationalUnit
ou: groups

dn: ou=people,dc=baeldung,dc=com
objectclass: top
objectclass: organizationalUnit
ou: people

dn: uid=baeldung,ou=people,dc=baeldung,dc=com
objectclass: top
objectclass: person
objectclass: organizationalPerson
objectclass: inetOrgPerson
cn: Jim Beam
sn: Beam
uid: baeldung
userPassword: password

dn: cn=admin,ou=groups,dc=baeldung,dc=com
objectclass: top
objectclass: groupOfNames
cn: admin
member: uid=baeldung,ou=people,dc=baeldung,dc=com

dn: cn=user,ou=groups,dc=baeldung,dc=com
objectclass: top
objectclass: groupOfNames
cn: user
member: uid=baeldung,ou=people,dc=baeldung,dc=com
```

## 6.使用 Spring Boot

**当在 Spring Boot 项目上工作时，我们也可以使用 [Spring Boot 启动器数据 Ldap](https://web.archive.org/web/20220902041127/https://search.maven.org/search?q=a:spring-boot-starter-data-ldap) 依赖，它将自动为我们装备`LdapContextSource `和`LdapTemplate `。**

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

关于 Spring 数据 LDAP 自动配置的更多细节可以在[官方文档](https://web.archive.org/web/20220902041127/https://docs.spring.io/spring-boot/docs/current/reference/htmlsingle/#features.nosql.ldap)中找到。Spring Boot 引入了 [LdapAutoConfiguration](https://web.archive.org/web/20220902041127/https://docs.spring.io/spring-boot/docs/current/api/org/springframework/boot/autoconfigure/ldap/LdapAutoConfiguration.html) ，它负责`LdapTemplate `的检测，然后可以将这些检测注入到所需的服务类中:

```java
@Autowired
private LdapTemplate ldapTemplate;
```

## 7。应用程序

最后，这是我们的简单应用程序:

```java
@Controller
public class MyController {

    @RequestMapping("/secure")
    public String secure(Map<String, Object> model, Principal principal) {
        model.put("title", "SECURE AREA");
        model.put("message", "Only Authorized Users Can See This Page");
        return "home";
    }
}
```

## 8。结论

在这篇关于使用 LDAP 实现 Spring 安全性的快速指南中，我们学习了如何使用 LDIF 配置一个基本系统并配置该系统的安全性。

本教程的**完整实现**可以在[的 GitHub 项目](https://web.archive.org/web/20220902041127/https://github.com/eugenp/tutorials/tree/master/spring-security-modules/spring-security-ldap "The Full Registration/Authentication Example Project on Github ")中找到——这是一个基于 Eclipse 的项目，所以它应该很容易导入和运行。