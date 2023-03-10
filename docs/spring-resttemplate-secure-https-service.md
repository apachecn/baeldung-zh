# 使用 Spring RestTemplate 访问 HTTPS REST 服务

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-resttemplate-secure-https-service>

## 1.概观

在本教程中，我们将看到如何使用 Spring 的 [`RestTemplate`](/web/20221122042731/https://www.baeldung.com/rest-template) 来消费 HTTPS 提供的 REST 服务。

## 2.设置

我们知道为了保证 REST 服务的安全，我们需要一个证书和一个由证书生成的密钥库。我们可以从证书颁发机构(CA)获得证书，以确保应用程序对于生产级应用程序来说是安全可信的。

出于本文的目的，我们将在示例应用程序中使用自签名证书。

我们将使用 Spring 的`RestTemplate`来消费 HTTPS REST 服务。

**首先，让我们创建一个控制器类`WelcomeController`和一个`/welcome` 端点，它返回一个简单的`String` 响应:**

```java
@RestController
public class WelcomeController {

    @GetMapping(value = "/welcome")
    public String welcome() {
       return "Welcome To Secured REST Service";
    }
} 
```

**然后，让我们将我们的密钥库添加到`src/main/resources`文件夹:**

[![](img/7319ca19b17108a7d8a8393d5cabcedf.png)](/web/20221122042731/https://www.baeldung.com/wp-content/uploads/2022/11/secure-cert-resource-folder-1.png)

**接下来，让我们将与密钥库相关的属性添加到我们的`application.properties` 文件:**

```java
server.port=8443
server.servlet.context-path=/
# The format used for the keystore
server.ssl.key-store-type=PKCS12
# The path to the keystore containing the certificate
server.ssl.key-store=classpath:keystore/baeldung.p12
# The password used to generate the certificate
server.ssl.key-store-password=password
# The alias mapped to the certificate
server.ssl.key-alias=baeldung
```

我们现在可以在这个端点访问 REST 服务:`https://localhost:8443/welcome`

## 3.消费安全休息服务

Spring 提供了一个方便的`RestTemplate`类来消费 REST 服务。

虽然消费一个简单的 REST 服务很简单，但是当消费一个安全的服务时，我们需要用服务使用的证书/密钥库定制`RestTemplate`。

接下来，让我们创建一个简单的`RestTemplate`对象，并通过添加所需的证书/密钥库来定制它。

### 3.1.创建一个`RestTemplate `客户端

**让我们编写一个简单的控制器，它使用一个`RestTemplate`来消费我们的 REST 服务:**

```java
@RestController
public class RestTemplateClientController {
    private static final String WELCOME_URL = "https://localhost:8443/welcome";

    @Autowired
    private RestTemplate restTemplate;

    @GetMapping("/welcomeclient")
    public String greetMessage() {
        String response = restTemplate.getForObject(WELCOME_URL, String.class);
        return response;
    }
}
```

**如果我们运行我们的代码并访问`/welcomeclient`端点，我们将得到一个错误，因为没有找到访问安全 REST 服务的有效证书:**

```java
`javax.net.ssl.SSLHandshakeException: sun.security.validator.ValidatorException: PKIX path building failed: 
sun.security.provider.certpath.SunCertPathBuilderException: unable to find valid certification path to requested 
target at sun.security.ssl.Alerts.getSSLException(Alerts.java:192)`
```

接下来，我们将看看如何解决这个错误。

### 3.2.为 HTTPS 访问配置 `RestTemplate`

访问安全 REST 服务的客户端应用程序应该在其`resources`文件夹中包含一个安全密钥库。此外，`RestTemplate`本身也需要配置。

**首先，让我们将前面的密钥库`baeldung.p12` 作为信任库添加到`/src/main/resources`文件夹中:**

[![Keystore in resource folder](img/1a2e12609b1b944e481cf8b436a4a39b.png)](/web/20221122042731/https://www.baeldung.com/wp-content/uploads/2022/11/secure-cert-resource-folder.png)

**接下来，我们需要在`application.properties`文件中添加信任库的详细信息:**

```java
server.port=8082
#trust store location
trust.store=classpath:keystore/baeldung.p12
#trust store password
trust.store.password=password 
```

**最后，让我们通过添加信任库来定制`RestTemplate`:**

```java
@Configuration
public class CustomRestTemplateConfiguration {

    @Value("${trust.store}")
    private Resource trustStore;

    @Value("${trust.store.password}")
    private String trustStorePassword;

    @Bean
    public RestTemplate restTemplate() throws KeyManagementException, NoSuchAlgorithmException, KeyStoreException,
      CertificateException, MalformedURLException, IOException {

        SSLContext sslContext = new SSLContextBuilder()
          .loadTrustMaterial(trustStore.getURL(), trustStorePassword.toCharArray()).build();
        SSLConnectionSocketFactory sslConFactory = new SSLConnectionSocketFactory(sslContext);

        CloseableHttpClient httpClient = HttpClients.custom().setSSLSocketFactory(sslConFactory).build();
        ClientHttpRequestFactory requestFactory = new HttpComponentsClientHttpRequestFactory(httpClient);
        return new RestTemplate(requestFactory);
    }
}
```

下面我们来详细了解一下上面`restTemplate()` 方法中的重要步骤。

首先，我们创建一个代表安全套接字协议实现的`SSLContext`对象。我们使用`SSLContextBuilder` 类的`build()`方法来创建它。

我们使用`SSLContextBuilder`的`loadTrustMaterial`()方法将密钥库文件和凭证加载到`SSLContext` 对象中。

然后，我们通过加载`SSLContext.` 为 TSL 和 SSL 连接创建 `SSLConnectionSocketFactory,`一个分层套接字工厂。此步骤的目的是验证服务器是否使用了我们在上一步中加载的可信证书列表，即验证服务器。

现在我们可以使用我们定制的`RestTemplate`在端点消费安全休息服务:`http://localhost:8082/welcomeclient:`
[![](img/5f702f4523dc4ed6a93f083799460a3d.png)](/web/20221122042731/https://www.baeldung.com/wp-content/uploads/2022/11/Secured-RestService-By-Customized-Resttemplate-Response.png)

## 4.**结论**

在本文中，我们讨论了如何使用定制的`RestTemplate`来消费安全的 REST 服务。

和往常一样，源代码可以在 GitHub 上的[处获得。](https://web.archive.org/web/20221122042731/https://github.com/eugenp/tutorials/tree/master/spring-web-modules/spring-resttemplate-3)