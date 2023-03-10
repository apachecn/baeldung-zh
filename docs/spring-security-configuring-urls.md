# spring Security–配置不同的 URL

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-security-configuring-urls>

 ![](img/4db576066d09309bd9091e50644f46af.png)

It’s just plain hard to get true, **real-time visibility into a running auth flow.**

Parts of the process can be completely hidden from us; if the complete authorization process requires a redirect from a remote OAuth production server, then every debugging effort must go through the production server.

It’s practically unfeasible to debug this locally. There’s no way to reproduce the exact state and no way to inspect what is actually happening under the hood. Not ideal.

Knowing these types of challenges, we built Lightrun - a real-time production debugging tool - to allow you to understand complicated flows with code-level information. Add logs, take snapshots (virtual breakpoints), and instrument metrics without a remote debugger, without stopping the running service, and, most importantly - **in real-time and without side effects**.

**Learn more with this 5-minute tutorial** focused on debugging these kinds of scenarios using Lightrun:

[>> Debugging Authentication and Authorization Using Lightrun](/web/20220524071045/https://www.baeldung.com/lightrun-n-security)

## 1.概观

在本教程中，我们将了解如何配置 Spring Security，以便针对不同的 URL 模式使用不同的安全配置。

当应用程序对某些操作要求更高的安全性，而对所有用户都允许其他操作时，这很有帮助。

## 2.设置

让我们从设置应用程序开始。

我们将需要 [Web](https://web.archive.org/web/20220524071045/https://mvnrepository.com/artifact/org.springframework.boot/spring-boot-starter-web) 和[安全](https://web.archive.org/web/20220524071045/https://mvnrepository.com/artifact/org.springframework.boot/spring-boot-starter-security)依赖来创建这个服务。让我们从向`pom.xml`文件添加以下依赖项开始:

```java
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
```

```java
<dependency> 
    <groupId>org.springframework.boot</groupId> 
    <artifactId>spring-boot-starter-security</artifactId> 
</dependency> 
```

## 3.创建 API

我们将创建一个带有两个 API 的 RESTful web 服务:一个产品 API 和一个客户 API。为此，我们将设置两个控制器。

### 3.1.产品 API

让我们创建`ProductController`。它包含一个方法`getProducts`，该方法返回产品列表:

```java
@RestController("/products")
public class ProductController {

    @GetMapping
    public List<Product> getProducts() {
        return new ArrayList<>(Arrays.asList(
          new Product("Product 1", "Description 1", 1.0),
          new Product("Product 2", "Description 2", 2.0)
        ));
    }
} 
```

### 3.2.客户 API

同样，我们来定义一下`CustomerController: `

```java
@RestController("/customers")
public class CustomerController {

    @GetMapping("/{id}")
    public Customer getCustomerById(@PathVariable("id") String id) {
        return new Customer("Customer 1", "Address 1", "Phone 1");
    }
} 
```

在典型的 web 应用程序中，所有用户，包括访客用户，都可以获得产品列表。

然而，通过 ID 获取客户的详细信息似乎是只有管理员才能做的事情。因此，我们将以能够实现这一点的方式来定义我们的安全配置。

## 4.设置安全配置

当我们将 Spring Security 添加到项目中时，它将默认禁用对所有 API 的访问。所以我们需要配置 Spring Security 来允许访问 API。

我们可以通过创建一个扩展了`WebSecurityConfigurerAdapter`类的`SecurityConfiguration`类来做到这一点。

让我们创建`SecurityConfiguration`类:

```java
@Configuration
public class SecurityConfiguration extends WebSecurityConfigurerAdapter {

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http
          .authorizeRequests()
          .antMatchers("/products/**").permitAll()
          .and()
          .authorizeRequests()
          .antMatchers("/customers/**").hasRole("ADMIN")
          .anyRequest().authenticated()
          .and()
          .httpBasic();
    }
} 
```

这里我们覆盖了`configure(HttpSecurity)`方法来配置应用程序的安全性。

此外，为了准备基本认证，我们需要[为我们的应用程序](/web/20220524071045/https://www.baeldung.com/java-config-spring-security)配置用户。

我们将阅读代码的每一部分，以便更好地理解它。

### 4.1.允许对产品 API 的请求

*   这个方法告诉 Spring 在授权请求时使用以下规则。
*   `antMatchers(“/products/**”):`这指定了安全配置适用的 URL 模式。我们用一个`permitAll() `动作将它链接起来。如果一个请求在其路径中包含了“`/products”` ，它就被允许发送到控制器。
*   我们可以使用`and()` 方法向我们的配置添加更多的规则。

这标志着一系列规则的终结。接下来的其他规则也将适用于这些请求。因此，我们需要确保我们的规则不会相互冲突。一个好的做法是在顶部定义通用规则，在底部定义更具体的规则。

### 4.2.仅允许管理员访问客户 API

现在让我们看看配置的第二部分:

*   要开始一个新的规则，我们可以再次使用`authorizeRequests()`方法。
*   `antMatchers(“/customers/**”).hasRole(“ADMIN”):`如果 URL 在路径中包含“`/customers”`,我们将检查发出请求的用户是否拥有 ADMIN 角色。

如果用户未通过身份验证，这将导致“401 未授权”错误。如果用户没有正确的角色，这将导致“403 禁止”错误。

### 4.3.缺席规则

我们已经添加了匹配来匹配某些请求。现在我们需要为其余的请求定义一些默认行为。

`anyRequest().authenticated()`–**`anyRequest()` 为任何与之前的规则**不匹配的请求定义一个规则链。在我们的例子中，这样的请求只要经过身份验证就会被传递。

请注意**在配置中只能有一个默认规则，并且需要在**的末尾。如果我们在添加一个默认规则后试图添加一个规则，我们会得到一个错误—`“Can't configure antMatchers after anyRequest”.`

## 5.测试

让我们使用 cURL 测试这两个 API。

### 5.1.测试产品 API

```java
$ curl -i http://localhost:8080/products
[
  {
    "name": "Product 1",
    "description": "Description 1",
    "price": 1.0
  },
  {
    "name": "Product 2",
    "description": "Description 2",
    "price": 2.0
  }
] 
```

我们得到了预期的两种产品的响应。

### 5.2.测试客户 API

```java
$ curl -i http://localhost:8080/customers/1 
```

响应正文为空。

如果我们检查标题，我们会看到“401 未授权”状态。这是因为只有角色为 ADMIN 的经过身份验证的用户才能访问客户 API。

现在，让我们在向请求添加身份验证信息后再试一次:

```java
$ curl -u admin:password -i http://localhost:8080/customers/1 
{
  "name": "Customer 1",
  "address": "Address 1",
  "phone": "Phone 1"
} 
```

太好了！我们现在可以访问客户 API 了。

## 6.结论

在本教程中，我们学习了如何在 Spring Boot 应用程序中设置 Spring 安全性。我们还介绍了使用`antMatchers()`方法配置特定于 URL 模式的访问。

像往常一样，本教程的代码可以在 GitHub 上找到[。](https://web.archive.org/web/20220524071045/https://github.com/eugenp/tutorials/tree/master/spring-boot-modules/spring-boot-security)