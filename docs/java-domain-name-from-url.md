# 用 Java 从给定的 URL 获取域名

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-domain-name-from-url>

## 1.概观

在这篇短文中，我们将看看用 Java 从给定的 URL 获取域名的不同方法。

## 2.什么是域名？

简单来说，域名代表一个指向 IP 地址的字符串。它是统一资源定位器(URL)的一部分。使用域名，用户可以通过客户端软件访问特定的网站。

域名通常由两个或三个部分组成，每个部分由一个点分隔。

从末尾开始，域名可能包括:

*   顶级域名(如`bealdung.com`中的`com`)，
*   二级域名(如`google.co.uk`中的`co`或`baeldung.com`中的`baeldung`)，
*   三级域名(如`google.co.uk`中的`google`

域名需要遵循[域名系统](/web/20220811171329/https://www.baeldung.com/cs/dns-intro) (DNS)规定的规则和程序。

## 3.使用 URI 类

让我们看看如何使用`[java.net.URI](https://web.archive.org/web/20220811171329/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/net/URI.html)`类从 URL 中提取域名。`URI`类提供了`getHost()`方法，该方法返回 URL 的主机部分:

```java
URI uri = new URI("https://www.baeldung.com/domain");
String host = uri.getHost();
assertEquals("www.baeldung.com", host);
```

**主机包含子域以及第三、第二和顶级域。**

此外，要获得域名，我们需要从给定的主机中删除子域:

```java
String domainName = host.startsWith("www.") ? host.substring(4) : host;
assertEquals("baeldung.com", domainName);
```

然而，在某些情况下，我们无法使用`URI`类获得域名。例如，如果我们不知道它的确切值，就不可能从 URL 中取出子域。

## 4.使用 Guava 库中的`InternetDomainName`类

现在我们将看看如何使用`Guava`库和`InternetDomainName`类来获得域名。

`InternetDomainName`类提供了`topPrivateDomain()`方法，该方法返回给定域名中比公共后缀低一级的部分。**换句话说，该方法将返回顶级、二级和三级域名。**

首先，我们需要从给定的 URL 值中提取主机。我们可以使用`URI`类:

```java
String urlString = "https://www.baeldung.com/java-tutorial";
URI uri = new URI(urlString);
String host = uri.getHost();
```

接下来，让我们使用`InternetDomainName`类及其`topPrivateDomain()`方法获得一个域名:

```java
InternetDomainName internetDomainName = InternetDomainName.from(host).topPrivateDomain(); 
String domainName = internetDomainName.toString(); 
assertEquals("baeldung.com", domainName);
```

与`URI`类相比，`InternetDomainName`将从返回值中省略子域。

最后，我们也可以从给定的 URL 中删除顶级域名:

```java
String publicSuffix = internetDomainName.publicSuffix().toString();
String name = domainName.substring(0, domainName.lastIndexOf("." + publicSuffix));
```

此外，让我们创建一个测试来检查功能:

```java
assertEquals("baeldung", domainNameClient.getName("jira.baeldung.com"));
assertEquals("google", domainNameClient.getName("www.google.co.uk"));
```

我们可以看到子域和顶级域都从结果中删除了。

## 5.使用正则表达式

使用正则表达式获取域名可能具有挑战性。例如，如果我们不知道确切的子域值，我们就不能确定应该从给定的 URL 中提取什么单词(如果有的话)。

另一方面，如果我们知道子域值，我们可以使用正则表达式将其从 URL 中删除:

```java
String url = "https://www.baeldung.com/domain";
String domainName =  url.replaceAll("http(s)?://|www\\.|/.*", "");
assertEquals("baeldung.com", domainName);
```

## 6.结论

在本文中，我们研究了如何从给定的 URL 中提取域名。与往常一样，GitHub 上的[提供了示例的源代码。](https://web.archive.org/web/20220811171329/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-networking-3)