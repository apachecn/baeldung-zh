# 信任 OkHttp 中的所有证书

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/okhttp-client-trust-all-certificates>

## 1.概观

在本教程中，我们将看到如何**创建和配置一个`OkHttpClient`来信任所有的证书**。

看看我们关于 OkHttp 的[文章，了解更多关于这个库的细节。](/web/20220529025657/https://www.baeldung.com/tag/okhttp/)

## 2.Maven 依赖性

让我们首先将 [OkHttp](https://web.archive.org/web/20220529025657/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22com.squareup.okhttp3%22%20AND%20a%3A%22okhttp%22) 依赖项添加到我们的`pom.xml`文件中:

```java
<dependency>
    <groupId>com.squareup.okhttp3</groupId>
    <artifactId>okhttp</artifactId>
    <version>4.9.2</version>
</dependency>
```

## 3.使用普通的`OkHttpClient`

首先，让我们用一个标准的`OkHttpClient`对象调用一个带有过期证书的网页:

```java
OkHttpClient client = new OkHttpClient.Builder().build();
client.newCall(new Request.Builder().url("https://expired.badssl.com/").build()).execute();
```

堆栈跟踪输出将如下所示:

```java
sun.security.validator.ValidatorException: PKIX path validation failed: java.security.cert.CertPathValidatorException: validity check failed
```

现在，让我们看看当我们使用自签名证书尝试另一个网站时收到的错误:

```java
sun.security.validator.ValidatorException: PKIX path building failed: sun.security.provider.certpath.SunCertPathBuilderException: unable to find valid certification path to requested target
```

让我们尝试一个拥有错误主机证书的网站:

```java
Hostname wrong.host.badssl.com not verified
```

正如我们看到的，默认情况下， **`OkHttpClient`如果调用有坏证书的站点**就会抛出错误。接下来，我们将看到如何创建和配置一个`OkHttpClient`来信任所有证书。

## 4.设置一个`OkHttpClient`来信任所有证书

让我们创建包含单个`X509TrustManager`的`TrustManager`数组，**通过覆盖它们的方法**来禁用默认证书验证:

```java
TrustManager[] trustAllCerts = new TrustManager[]{
    new X509TrustManager() {
        @Override
        public void checkClientTrusted(java.security.cert.X509Certificate[] chain, String authType) {
        }

        @Override
        public void checkServerTrusted(java.security.cert.X509Certificate[] chain, String authType) {
        }

        @Override
        public java.security.cert.X509Certificate[] getAcceptedIssuers() {
            return new java.security.cert.X509Certificate[]{};
        }
    }
};
```

我们将使用这个`TrustManager`到**的数组创建一个`SSLContext`** :

```java
SSLContext sslContext = SSLContext.getInstance("SSL");
sslContext.init(null, trustAllCerts, new java.security.SecureRandom());
```

然后，我们将用这个`SSLContext`到**设置`OkHttpClient`生成器的`SSLSocketFactory`** :

```java
OkHttpClient.Builder newBuilder = new OkHttpClient.Builder();
newBuilder.sslSocketFactory(sslContext.getSocketFactory(), (X509TrustManager) trustAllCerts[0]);
newBuilder.hostnameVerifier((hostname, session) -> true);
```

我们还将新的`Builder`的`HostnameVerifier`设置为一个**新的`HostnameVerifier` 对象，其验证方法总是返回`true`** 。

最后，我们可以获得一个新的`OkHttpClient`对象，并再次调用带有错误证书的站点，而不会出现任何错误:

```java
OkHttpClient newClient = newBuilder.build();
newClient.newCall(new Request.Builder().url("https://expired.badssl.com/").build()).execute();
```

## 5.结论

在这篇短文中，我们看到了如何**创建和配置一个`OkHttpClient`来信任所有证书**。当然，不建议信任所有证书。但是，在某些情况下，我们可能会需要它。

这篇文章的完整代码可以在 GitHub 的[上找到。](https://web.archive.org/web/20220529025657/https://github.com/eugenp/tutorials/tree/master/libraries-http-2)