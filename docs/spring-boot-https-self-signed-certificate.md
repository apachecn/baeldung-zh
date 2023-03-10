# HTTPS 在 Spring Boot 使用自签名证书

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-boot-https-self-signed-certificate>

## 1。概述

在本教程中，我们将学习如何在 Spring Boot 启用 HTTPS。为此，我们还将生成一个自签名证书，并配置一个简单的应用程序。

关于 Spring Boot 项目的更多细节，我们可以参考这里的一些资源。

## 延伸阅读:

## [Spring Boot 安全自动配置](/web/20221006230231/https://www.baeldung.com/spring-boot-security-autoconfiguration)

A quick and practical guide to Spring Boot's default Spring Security configuration.[Read more](/web/20221006230231/https://www.baeldung.com/spring-boot-security-autoconfiguration) →

## [Spring Security Java 配置简介](/web/20221006230231/https://www.baeldung.com/java-config-spring-security)

A quick and practical guide to Java Config for Spring Security[Read more](/web/20221006230231/https://www.baeldung.com/java-config-spring-security) →

## 2。生成自签名证书

在开始之前，我们将创建一个自签名证书。我们将使用以下证书格式之一:

*   PKCS12: [公钥加密标准](https://web.archive.org/web/20221006230231/https://en.wikipedia.org/wiki/PKCS_12)是一种受密码保护的格式，可以包含多个证书和密钥；这是一种全行业使用的格式。
*   JKS: [Java KeyStore](https://web.archive.org/web/20221006230231/https://en.wikipedia.org/wiki/Keystore) 类似于 PKCS12 这是一种专有格式，仅限于 Java 环境。

我们可以使用 keytool 或 OpenSSL 工具从命令行生成证书。 [Keytool](https://web.archive.org/web/20221006230231/https://docs.oracle.com/en/java/javase/11/tools/keytool.html) 自带 Java 运行时环境，OpenSSL 可以从[这里](https://web.archive.org/web/20221006230231/https://www.openssl.org/)下载。

在我们的演示中，让我们使用 keytool。

### 2.1。生成密钥库

现在，我们将创建一组加密密钥，并将它们存储在密钥库中。

我们可以使用以下命令来生成 PKCS12 密钥库格式:

```java
keytool -genkeypair -alias baeldung -keyalg RSA -keysize 2048 -storetype PKCS12 -keystore baeldung.p12 -validity 3650
```

我们可以在同一个密钥库中存储任意数量的密钥对，每个密钥对都有一个惟一的别名。

为了以 JKS 格式生成我们的密钥库，我们可以使用以下命令:

```java
keytool -genkeypair -alias baeldung -keyalg RSA -keysize 2048 -keystore baeldung.jks -validity 3650
```

我们建议使用 PKCS12 格式，这是一种行业标准格式。因此，如果我们已经有一个 JKS 密钥库，我们可以使用以下命令将其转换为 PKCS12 格式:

```java
keytool -importkeystore -srckeystore baeldung.jks -destkeystore baeldung.p12 -deststoretype pkcs12
```

我们必须提供源密钥库密码，并设置一个新的密钥库密码。稍后将需要别名和密钥库密码。

## 3。在 Spring Boot 启用 HTTPS

Spring Boot 提供了一组声明性的`[server.ssl.* properties](https://web.archive.org/web/20221006230231/https://docs.spring.io/spring-boot/docs/current/reference/htmlsingle/#howto-configure-ssl).` 我们将在示例应用程序中使用这些属性来配置 HTTPS。

我们将从一个简单的带有 Spring Security 的 Spring Boot 应用程序[开始，它包含一个由端点`/welcome`处理的欢迎页面。](/web/20221006230231/https://www.baeldung.com/spring-boot-security-autoconfiguration)

然后，我们将把上一步中生成的名为“`baeldung.p12,”`的文件复制到“`src/main/resources/keystore`目录中。

### 3.1。配置 SSL 属性

现在我们将配置 SSL 相关属性:

```java
# The format used for the keystore. It could be set to JKS in case it is a JKS file
server.ssl.key-store-type=PKCS12
# The path to the keystore containing the certificate
server.ssl.key-store=classpath:keystore/baeldung.p12
# The password used to generate the certificate
server.ssl.key-store-password=password
# The alias mapped to the certificate
server.ssl.key-alias=baeldung
```

因为我们使用的是支持 Spring 安全的应用程序，所以让我们将其配置为只接受 HTTPS 请求:

```java
server.ssl.enabled=true
```

## 4。调用 HTTPS 网址

现在，我们已经在应用程序中启用了 HTTPS，让我们转到客户端，探索如何使用自签名证书调用 HTTPS 端点。

首先，我们需要创建一个信任存储。因为我们已经生成了 PKCS12 文件，所以我们可以将它用作信任存储。让我们为信任存储详细信息定义新的属性:

```java
#trust store location
trust.store=classpath:keystore/baeldung.p12
#trust store password
trust.store.password=password
```

然后我们需要为信任存储准备一个`SSLContext`并创建一个定制的`RestTemplate:`

```java
RestTemplate restTemplate() throws Exception {
    SSLContext sslContext = new SSLContextBuilder()
      .loadTrustMaterial(trustStore.getURL(), trustStorePassword.toCharArray())
      .build();
    SSLConnectionSocketFactory socketFactory = new SSLConnectionSocketFactory(sslContext);
    HttpClient httpClient = HttpClients.custom()
      .setSSLSocketFactory(socketFactory)
      .build();
    HttpComponentsClientHttpRequestFactory factory = 
      new HttpComponentsClientHttpRequestFactory(httpClient);
    return new RestTemplate(factory);
}
```

为了便于演示，让我们确保`Spring Security` 允许任何传入的请求:

```java
protected void configure(HttpSecurity http) throws Exception {
    http.authorizeRequests()
      .antMatchers("/**")
      .permitAll();
}
```

最后，我们可以呼叫 HTTPS 端点:

```java
@Test
public void whenGETanHTTPSResource_thenCorrectResponse() throws Exception {
    ResponseEntity<String> response = 
      restTemplate().getForEntity(WELCOME_URL, String.class, Collections.emptyMap());

    assertEquals("<h1>Welcome to Secured Site</h1>", response.getBody());
    assertEquals(HttpStatus.OK, response.getStatusCode());
}
```

## 5。结论

在本文中，我们首先学习了如何生成自签名证书，以便在 Spring Boot 应用程序中启用 HTTPS。然后我们讨论了如何调用支持 HTTPS 的端点。

和往常一样，我们可以在 [GitHub 库](https://web.archive.org/web/20221006230231/https://github.com/eugenp/tutorials/tree/master/spring-security-modules/spring-security-web-boot-2)上找到完整的源代码。

最后，为了运行代码示例，我们需要在`pom.xml`中取消注释下面的 start-class 属性:

```java
<start-class>com.baeldung.ssl.HttpsEnabledApplication</start-class>
```