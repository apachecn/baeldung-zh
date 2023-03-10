# 使用纯 Java 的 LDAP 认证

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-ldap-auth>

## 1。简介

在本文中，我们将介绍如何使用纯 Java 通过 [LDAP](https://web.archive.org/web/20221206221623/https://ldap.com/learn-about-ldap/) 认证用户。此外，我们将探索如何搜索用户的[识别名](https://web.archive.org/web/20221206221623/https://ldap.com/ldap-dns-and-rdns/) (DN)。这很重要，因为 LDAP 需要 DN 来认证用户。

为了进行搜索和用户认证，我们将使用 Java 命名和目录接口( [JNDI](/web/20221206221623/https://www.baeldung.com/jndi) )的目录服务访问功能。

首先，我们将简要讨论什么是 LDAP 和 JNDI。然后我们将讨论如何通过 JNDI API 使用 LDAP 进行认证。

## 2。什么是 LDAP？

**轻量级目录访问协议(LDAP)为客户端定义了一种发送请求和从目录服务接收响应的方式**。我们称使用这个协议的目录服务为 LDAP 服务器。

**LDAP 服务器提供的数据存储在基于[x . 500](https://web.archive.org/web/20221206221623/https://docs.oracle.com/javase/jndi/tutorial/ldap/models/x500.html)的信息模型中。这是一组用于电子目录服务的计算机网络标准。**

## 3。什么是 JNDI？

JNDI 为应用程序发现和访问命名和目录服务提供了一个标准 API。它的根本目的是为应用程序提供一种访问组件和资源的方式。这包括本地和网络。

命名服务是这种能力的基础，因为它们在分层名称空间中通过名称提供对服务、数据或对象的单点访问。为每个本地或网络可访问资源指定的名称是在托管命名服务的服务器上配置的。

我们可以通过使用 JNDI 的命名服务接口来访问目录服务，比如 LDAP。这是因为目录服务只是一种特殊类型的命名服务，它使每个命名条目都有一个与之关联的属性列表。

除了属性之外，每个目录条目可以有一个或多个子条目。这使得条目可以分层链接。在 JNDI 中，目录条目的子条目被表示为其父上下文的子上下文。

JNDI API 的一个主要优点是它独立于任何底层服务提供者的实现，例如 LDAP。因此，我们可以使用 JNDI 来访问 LDAP 目录服务，而不需要使用特定于协议的 API。

使用 JNDI 不需要外部库，因为它是 Java SE 平台的一部分。此外，作为 Java EE 中的核心技术，它被广泛用于实现企业应用程序。

## 4。用 LDAP 认证的 JNDI API 概念

在讨论示例代码之前，让我们先了解一下使用 JNDI API 进行基于 LDAP 的身份验证的一些基础知识。

为了连接到 LDAP 服务器，我们首先需要创建一个 JNDI `[InitialDirContext](https://web.archive.org/web/20221206221623/https://docs.oracle.com/en/java/javase/11/docs/api/java.naming/javax/naming/directory/InitialDirContext.html)`对象。这样做时，我们需要将环境属性作为 [`Hashtable`](https://web.archive.org/web/20221206221623/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/Hashtable.html) 传递给它的构造函数来配置它。

其中，我们需要为这个`Hashtable`添加属性，以获得我们希望用来进行身份验证的用户名和密码。为此，我们必须将用户的 DN 和密码分别设置为`Context.SECURITY_PRINCIPAL`和`Context.SECURITY_CREDENTIALS`属性。

`InitialDirContext`实现了 [`DirContext`](https://web.archive.org/web/20221206221623/https://docs.oracle.com/en/java/javase/11/docs/api/java.naming/javax/naming/directory/DirContext.html) ，主 JNDI 目录服务接口。通过这个接口，我们可以使用新的上下文在 LDAP 服务器上执行各种目录服务操作。这些包括将名称和属性绑定到对象，以及搜索目录条目。

值得注意的是，JNDI 返回的对象将与它们的底层 LDAP 条目具有相同的名称和属性。因此，要搜索一个条目，我们可以使用它的名称和属性作为查找的标准。

一旦我们通过 JNDI 检索到一个目录条目，我们就可以使用`[Attributes](https://web.archive.org/web/20221206221623/https://docs.oracle.com/en/java/javase/11/docs/api/java.naming/javax/naming/directory/Attributes.html)`接口查看它的属性。此外，我们可以使用`Attribute`接口来检查它们。

## 5。如果我们没有用户的 DN 怎么办？

有时我们没有用户的 DN 来立即进行身份验证。为了解决这个问题，我们首先需要使用管理员凭证创建一个`InitialDirContext`。之后，我们可以使用它从目录服务器中搜索相关用户，并获得他的 DN。

然后，一旦我们有了用户的 DN，我们就可以通过创建一个新的`InitialDirContext`来认证他，这次是用他的凭证。为此，我们首先需要在环境属性中设置用户的 DN 和密码。之后，我们需要在创建时将这些属性传递给`InitDirContext`的构造函数。

既然我们已经讨论了使用 JNDI API 通过 LDAP 认证用户，那么让我们来看一下示例代码。

## 6。示例代码

在我们的例子中，我们将使用 [ApacheDS](https://web.archive.org/web/20221206221623/https://directory.apache.org/apacheds/) 目录服务器的[嵌入式版本](https://web.archive.org/web/20221206221623/https://directory.apache.org/apacheds/advanced-ug/7-embedding-apacheds.html)。这是一个使用 Java 构建的 LDAP 服务器，设计为在单元测试中以嵌入式模式运行。

让我们看看如何设置它。

### 6.1。设置嵌入式 ApacheDS 服务器

为了使用嵌入式 ApacheDS 服务器，我们需要定义 Maven [依赖关系](https://web.archive.org/web/20221206221623/https://search.maven.org/classic/#search%7Cga%7C1%7Cg%3A%22org.apache.directory.server%22%20AND%20a%3A%22apacheds-test-framework%22):

```java
<dependency>
    <groupId>org.apache.directory.server</groupId>
    <artifactId>apacheds-test-framework</artifactId>
    <version>2.0.0.AM26</version>
    <scope>test</scope>
</dependency> 
```

接下来，我们需要使用 JUnit 4 创建一个单元测试类。为了在这个类中使用嵌入式 ApacheDS 服务器，我们必须声明它从 ApacheDS 库扩展了`AbstractLdapTestUnit`。由于这个库与 JUnit 5 还不兼容，我们需要使用 JUnit 4。

此外，我们需要在我们的单元测试类声明上面包含 [Java 注释](https://web.archive.org/web/20221206221623/https://directory.apache.org/apacheds/advanced-ug/7-embedding-apacheds.html#basic-test)来配置服务器。我们可以从[的完整代码示例](https://web.archive.org/web/20221206221623/https://github.com/eugenp/tutorials/blob/master/core-java-modules/core-java-jndi/src/test/java/com/baeldung/jndi/ldap/auth/JndiLdapAuthManualTest.java)中看到给它们取什么值，我们将在后面探讨。

最后，我们还需要将 [`users.ldif`](https://web.archive.org/web/20221206221623/https://github.com/eugenp/tutorials/blob/master/core-java-modules/core-java-jndi/src/test/resources/users.ldif) 添加到类路径中。这样，当我们运行我们的代码示例时，ApacheDS 服务器就可以从这个文件中加载 [LDIF](https://web.archive.org/web/20221206221623/https://datatracker.ietf.org/doc/html/rfc2849) 格式的目录条目。这样做时，服务器将为用户`Joe Simms`加载条目。

接下来，我们将讨论验证用户身份的示例代码。为了在 LDAP 服务器上运行它，我们需要将代码添加到单元测试类的方法中。这将使用文件中定义的 DN 和密码通过 LDAP 对 Joe 进行身份验证。

### 6.2。认证用户

为了认证用户`Joe Simms`，我们需要创建一个新的`InitialDirContext`对象。这将创建到目录服务器的连接，并使用用户的 DN 和密码通过 LDAP 对用户进行身份验证。

为此，我们首先需要将这些环境属性添加到一个`Hashtable`中:

```java
Hashtable<String, String> environment = new Hashtable<String, String>();

environment.put(Context.INITIAL_CONTEXT_FACTORY, "com.sun.jndi.ldap.LdapCtxFactory");
environment.put(Context.PROVIDER_URL, "ldap://localhost:10389");
environment.put(Context.SECURITY_AUTHENTICATION, "simple");
environment.put(Context.SECURITY_PRINCIPAL, "cn=Joe Simms,ou=Users,dc=baeldung,dc=com");
environment.put(Context.SECURITY_CREDENTIALS, "12345");
```

接下来，在一个名为`authenticateUser`的新方法中，我们将通过将环境属性传递给它的构造函数来创建`InitialDirContext`对象。然后，我们将关闭上下文以释放资源:

```java
DirContext context = new InitialDirContext(environment);
context.close();
```

最后，我们将验证用户:

```java
assertThatCode(() -> authenticateUser(environment)).doesNotThrowAnyException();
```

既然我们已经介绍了用户身份验证成功的情况，那么让我们来看看它失败的情况。

### 6.3。处理用户认证失败

应用与前面相同的环境属性，让我们通过使用错误的密码使身份验证失败:

```java
environment.put(Context.SECURITY_CREDENTIALS, "wrongpassword");
```

然后，我们将检查使用该密码验证用户是否如预期的那样失败:

```java
assertThatExceptionOfType(AuthenticationException.class).isThrownBy(() -> authenticateUser(environment));
```

接下来，我们来讨论在没有用户 DN 的情况下，如何对用户进行认证。

### 6.4。作为管理员查找用户的 DN

有时，当我们想要认证一个用户时，我们手头没有他的 DN。在这种情况下，我们首先需要创建一个带有管理员凭证的目录上下文来查找用户的 DN，然后用它来验证用户。

和以前一样，我们首先需要在一个`Hashtable`中添加一些环境属性。但是这一次，我们将使用管理员的 DN 作为`Context.SECURITY_PRINCIPAL`，以及他的默认管理员密码作为`Context.SECURITY_CREDENTIALS`属性:

```java
Hashtable<String, String> environment = new Hashtable<String, String>();

environment.put(Context.INITIAL_CONTEXT_FACTORY, "com.sun.jndi.ldap.LdapCtxFactory");
environment.put(Context.PROVIDER_URL, "ldap://localhost:10389");
environment.put(Context.SECURITY_AUTHENTICATION, "simple");
environment.put(Context.SECURITY_PRINCIPAL, "uid=admin,ou=system");
environment.put(Context.SECURITY_CREDENTIALS, "secret");
```

接下来，我们将用这些属性创建一个`InitialDirContext`对象:

```java
DirContext adminContext = new InitialDirContext(environment);
```

这将创建一个目录上下文，以管理员身份验证到服务器的连接。这赋予了我们搜索用户 DN 的安全权限。

现在，我们将根据用户的 CN，即他的[常用名](https://web.archive.org/web/20221206221623/https://ldapwiki.com/wiki/CommonName)，为我们的搜索定义过滤器。

```java
String filter = "(&(objectClass=person)(cn=Joe Simms))";
```

然后，使用这个`filter`来搜索用户，我们将创建一个 [`SearchControls`](https://web.archive.org/web/20221206221623/https://docs.oracle.com/en/java/javase/11/docs/api/java.naming/javax/naming/directory/SearchControls.html) 对象:

```java
String[] attrIDs = { "cn" };
SearchControls searchControls = new SearchControls();
searchControls.setReturningAttributes(attrIDs);
searchControls.setSearchScope(SearchControls.SUBTREE_SCOPE);
```

接下来，我们将使用我们的`filter`和`SearchControls`搜索用户:

```java
NamingEnumeration<SearchResult> searchResults
  = adminContext.search("dc=baeldung,dc=com", filter, searchControls);

String commonName = null;
String distinguishedName = null;
if (searchResults.hasMore()) {

    SearchResult result = (SearchResult) searchResults.next();
    Attributes attrs = result.getAttributes();

    distinguishedName = result.getNameInNamespace();
    assertThat(distinguishedName, isEqualTo("cn=Joe Simms,ou=Users,dc=baeldung,dc=com")));

    commonName = attrs.get("cn").toString();
    assertThat(commonName, isEqualTo("cn: Joe Simms")));
} 
```

现在我们已经有了用户的 DN，让我们对用户进行身份验证。

### 6.5。使用用户的查找 DN 进行认证

现在有了用户的 DN 来进行身份验证，我们将用用户的 DN 和密码替换现有环境属性中的管理员 DN 和密码:

```java
environment.put(Context.SECURITY_PRINCIPAL, distinguishedName);
environment.put(Context.SECURITY_CREDENTIALS, "12345");
```

然后，有了这些，让我们验证用户:

```java
assertThatCode(() -> authenticateUser(environment)).doesNotThrowAnyException();
```

最后，我们将关闭管理员的上下文来释放资源:

```java
adminContext.close();
```

## 7。结论

在本文中，我们讨论了如何使用 JNDI 通过 LDAP 利用用户的 DN 和密码对用户进行身份验证。

此外，我们还研究了如果没有 DN，如何查找它。

像往常一样，例子的完整源代码可以在 GitHub 的[中找到。](https://web.archive.org/web/20221206221623/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-jndi)