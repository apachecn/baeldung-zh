# 弹簧座 API + OAuth2 + Angular

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/rest-api-spring-oauth2-angular>

## 1。概述

在本教程中，**我们将使用 OAuth2 保护 REST API，并从一个简单的 Angular 客户端使用它。**

我们将要构建的应用程序将由三个独立的模块组成:

*   授权服务器
*   资源服务器
*   UI 授权代码:使用授权代码流的前端应用程序

我们将在 Spring Security 5 中使用 OAuth 堆栈。如果你想使用 Spring Security OAuth legacy stack，可以看看之前的这篇文章:[Spring REST API+OAuth 2+Angular(使用 Spring Security OAuth Legacy Stack)](/web/20220707143825/https://www.baeldung.com/rest-api-spring-oauth2-angular-legacy)。

## 延伸阅读:

## [使用带有 Spring 安全 OAuth 的 JWT](/web/20220707143825/https://www.baeldung.com/spring-security-oauth-jwt)

A guide to using JWT tokens with Spring Security 5.[Read more](/web/20220707143825/https://www.baeldung.com/spring-security-oauth-jwt) →

## [OAuth2.0 和动态客户端注册(使用 Spring Security OAuth 遗留堆栈)](/web/20220707143825/https://www.baeldung.com/spring-security-oauth-dynamic-client-registration)

Learn how to define clients dynamically with Spring Security and OAuth2.[Read more](/web/20220707143825/https://www.baeldung.com/spring-security-oauth-dynamic-client-registration) →

让我们直接开始吧。

## 2。OAuth2 授权服务器(AS)

简单地说，**授权服务器是一个发布授权令牌的应用程序。**

以前，Spring Security OAuth 堆栈提供了将授权服务器设置为 Spring 应用程序的可能性。但是这个项目已经被否决了，主要是因为 OAuth 是一个开放标准，有许多成熟的提供者，比如 Okta、Keycloak 和 ForgeRock 等等。

其中，我们将使用[键盘锁](/web/20220707143825/https://www.baeldung.com/spring-boot-keycloak)。它是一个开源的身份和访问管理服务器，由 JBoss 用 Java 开发，由 Red Hat 管理。它不仅支持 OAuth2，还支持其他标准协议，如 OpenID Connect 和 SAML。

在本教程中，**我们将在一个 Spring Boot 应用中设置一个[嵌入式 Keycloak 服务器。](/web/20220707143825/https://www.baeldung.com/keycloak-embedded-in-a-spring-boot-application/)**

## 3.资源服务器(RS)

现在我们来讨论资源服务器；这本质上是 REST API，我们最终希望能够使用它。

### 3.1。Maven 配置

我们的资源服务器的 pom 与之前的授权服务器 pom 非常相似，除了 Keycloak 部分和**一个额外的 [`spring-boot-starter-oauth2-resource-server`](https://web.archive.org/web/20220707143825/https://search.maven.org/search?q=a:spring-boot-starter-oauth2-resource-server) 依赖**:

```java
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-oauth2-resource-server</artifactId>
</dependency>
```

### 3.2.安全配置

由于我们使用的是 Spring Boot，**我们可以使用引导属性定义最低要求的配置。**

我们将在一个`application.yml`文件中这样做:

```java
server: 
  port: 8081
  servlet: 
    context-path: /resource-server

spring:
  security:
    oauth2:
      resourceserver:
        jwt:
          issuer-uri: http://localhost:8083/auth/realms/baeldung
          jwk-set-uri: http://localhost:8083/auth/realms/baeldung/protocol/openid-connect/certs
```

在这里，我们指定将使用 JWT 令牌进行授权。

**`jwk-set-uri`属性指向包含公钥的 URI，这样我们的资源服务器可以验证令牌的完整性。**

`issuer-uri`属性表示验证令牌发行者(即授权服务器)的附加安全措施。但是，添加该属性还要求授权服务器应该在我们可以启动资源服务器应用程序之前运行。

接下来，让我们为 API 设置一个**安全配置来保护端点**:

```java
@Configuration
public class SecurityConfig extends WebSecurityConfigurerAdapter {

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http.cors()
            .and()
              .authorizeRequests()
                .antMatchers(HttpMethod.GET, "/user/info", "/api/foos/**")
                  .hasAuthority("SCOPE_read")
                .antMatchers(HttpMethod.POST, "/api/foos")
                  .hasAuthority("SCOPE_write")
                .anyRequest()
                  .authenticated()
            .and()
              .oauth2ResourceServer()
                .jwt();
    }
}
```

正如我们所看到的，对于我们的 GET 方法，我们只允许具有`read`范围的请求。对于 POST 方法，除了`read`之外，请求者还需要有一个`write`权限。然而，对于任何其他端点，请求应该由任何用户进行身份验证。

同样，**`oauth2ResourceServer()`**方法指定这是一个资源服务器，带有`jwt()-`格式的令牌。

这里要注意的另一点是使用方法`cors()`来允许请求上的访问控制头。这一点尤其重要，因为我们正在处理一个 Angular 客户端，我们的请求将来自另一个原始 URL。

### 3.4.模型和存储库

接下来，让我们为我们的模型`Foo`定义一个`javax.persistence.Entity`:

```java
@Entity
public class Foo {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String name;

    // constructor, getters and setters
}
```

然后我们需要一个`Foo`的存储库。我们将使用 Spring 的`PagingAndSortingRepository`:

```java
public interface IFooRepository extends PagingAndSortingRepository<Foo, Long> {
} 
```

### 3.4.服务和实施

之后，我们将为我们的 API 定义并实现一个简单的服务:

```java
public interface IFooService {
    Optional<Foo> findById(Long id);

    Foo save(Foo foo);

    Iterable<Foo> findAll();

}

@Service
public class FooServiceImpl implements IFooService {

    private IFooRepository fooRepository;

    public FooServiceImpl(IFooRepository fooRepository) {
        this.fooRepository = fooRepository;
    }

    @Override
    public Optional<Foo> findById(Long id) {
        return fooRepository.findById(id);
    }

    @Override
    public Foo save(Foo foo) {
        return fooRepository.save(foo);
    }

    @Override
    public Iterable<Foo> findAll() {
        return fooRepository.findAll();
    }
} 
```

### 3.5.样本控制器

现在让我们实现一个简单的控制器，通过 DTO 公开我们的`Foo`资源:

```java
@RestController
@RequestMapping(value = "/api/foos")
public class FooController {

    private IFooService fooService;

    public FooController(IFooService fooService) {
        this.fooService = fooService;
    }

    @CrossOrigin(origins = "http://localhost:8089")    
    @GetMapping(value = "/{id}")
    public FooDto findOne(@PathVariable Long id) {
        Foo entity = fooService.findById(id)
            .orElseThrow(() -> new ResponseStatusException(HttpStatus.NOT_FOUND));
        return convertToDto(entity);
    }

    @GetMapping
    public Collection<FooDto> findAll() {
        Iterable<Foo> foos = this.fooService.findAll();
        List<FooDto> fooDtos = new ArrayList<>();
        foos.forEach(p -> fooDtos.add(convertToDto(p)));
        return fooDtos;
    }

    protected FooDto convertToDto(Foo entity) {
        FooDto dto = new FooDto(entity.getId(), entity.getName());

        return dto;
    }
}
```

**注意上面`@CrossOrigin`的用法；这是控制器级别的配置，我们需要允许 CORS 从我们的 Angular 应用程序运行在指定的网址。**

这是我们的`FooDto`:

```java
public class FooDto {
    private long id;
    private String name;
}
```

## 4。前端—设置

我们现在来看一个简单的客户端前端角度实现，它将访问我们的 REST API。

我们将首先使用 [Angular CLI](https://web.archive.org/web/20220707143825/https://cli.angular.io/) 来生成和管理我们的前端模块。

首先，我们安装[节点和 npm](https://web.archive.org/web/20220707143825/https://nodejs.org/en/download/)T3，因为 Angular CLI 是一个 NPM 工具。

然后我们需要使用 [`frontend-maven-plugin`](https://web.archive.org/web/20220707143825/https://github.com/eirslett/frontend-maven-plugin) 来使用 Maven 构建我们的 Angular 项目:

```java
<build>
    <plugins>
        <plugin>
            <groupId>com.github.eirslett</groupId>
            <artifactId>frontend-maven-plugin</artifactId>
            <version>1.3</version>
            <configuration>
                <nodeVersion>v6.10.2</nodeVersion>
                <npmVersion>3.10.10</npmVersion>
                <workingDirectory>src/main/resources</workingDirectory>
            </configuration>
            <executions>
                <execution>
                    <id>install node and npm</id>
                    <goals>
                        <goal>install-node-and-npm</goal>
                    </goals>
                </execution>
                <execution>
                    <id>npm install</id>
                    <goals>
                        <goal>npm</goal>
                    </goals>
                </execution>
                <execution>
                    <id>npm run build</id>
                    <goals>
                        <goal>npm</goal>
                    </goals>
                    <configuration>
                        <arguments>run build</arguments>
                    </configuration>
                </execution>
            </executions>
        </plugin>
    </plugins>
</build>
```

最后，**使用 Angular CLI 生成新模块:**

```java
ng new oauthApp
```

在下一节中，我们将讨论 Angular 应用程序逻辑。

## 5.使用角度的授权代码流

我们将在这里使用 OAuth2 授权代码流。

我们的用例:客户端应用程序向授权服务器请求一个代码，并显示一个登录页面。一旦用户提供他们的有效凭证并提交，授权服务器就给我们代码。然后前端客户端使用它来获取访问令牌。

### 5.1.家用部件

让我们从我们的主要组件`HomeComponent`开始，所有的动作都从这里开始:

```java
@Component({
  selector: 'home-header',
  providers: [AppService],
  template: `<div class="container" >
    <button *ngIf="!isLoggedIn" class="btn btn-primary" (click)="login()" type="submit">
      Login</button>
    <div *ngIf="isLoggedIn" class="content">
      <span>Welcome !!</span>
      <a class="btn btn-default pull-right"(click)="logout()" href="#">Logout</a>
      <br/>
      <foo-details></foo-details>
    </div>
  </div>`
})

export class HomeComponent {
  public isLoggedIn = false;

  constructor(private _service: AppService) { }

  ngOnInit() {
    this.isLoggedIn = this._service.checkCredentials();    
    let i = window.location.href.indexOf('code');
    if(!this.isLoggedIn && i != -1) {
      this._service.retrieveToken(window.location.href.substring(i + 5));
    }
  }

  login() {
    window.location.href = 
      'http://localhost:8083/auth/realms/baeldung/protocol/openid-connect/auth?
         response_type=code&scope;=openid%20write%20read&client;_id=' + 
         this._service.clientId + '&redirect;_uri='+ this._service.redirectUri;
    }

  logout() {
    this._service.logout();
  }
} 
```

一开始，当用户没有登录时，只显示登录按钮。单击此按钮后，用户将被导航到 AS 的授权 URL，并在其中键入用户名和密码。成功登录后，用户被重定向回授权代码，然后我们使用该代码检索访问令牌。

### 5.2.应用服务

现在让我们看看位于`app.service.ts`的`AppService`，它包含了服务器交互的逻辑:

*   `retrieveToken()`:使用授权码获取接入令牌
*   使用 ng2-cookies 库将我们的访问令牌保存在 cookie 中
*   `getResource()`:使用其 ID 从服务器获取 Foo 对象
*   `checkCredentials()`:检查用户是否登录
*   `logout()`:删除访问令牌 cookie 并注销用户

```java
export class Foo {
  constructor(public id: number, public name: string) { }
} 

@Injectable()
export class AppService {
  public clientId = 'newClient';
  public redirectUri = 'http://localhost:8089/';

  constructor(private _http: HttpClient) { }

  retrieveToken(code) {
    let params = new URLSearchParams();   
    params.append('grant_type','authorization_code');
    params.append('client_id', this.clientId);
    params.append('client_secret', 'newClientSecret');
    params.append('redirect_uri', this.redirectUri);
    params.append('code',code);

    let headers = 
      new HttpHeaders({'Content-type': 'application/x-www-form-urlencoded; charset=utf-8'});

      this._http.post('http://localhost:8083/auth/realms/baeldung/protocol/openid-connect/token', 
        params.toString(), { headers: headers })
        .subscribe(
          data => this.saveToken(data),
          err => alert('Invalid Credentials')); 
  }

  saveToken(token) {
    var expireDate = new Date().getTime() + (1000 * token.expires_in);
    Cookie.set("access_token", token.access_token, expireDate);
    console.log('Obtained Access token');
    window.location.href = 'http://localhost:8089';
  }

  getResource(resourceUrl) : Observable<any> {
    var headers = new HttpHeaders({
      'Content-type': 'application/x-www-form-urlencoded; charset=utf-8', 
      'Authorization': 'Bearer '+Cookie.get('access_token')});
    return this._http.get(resourceUrl, { headers: headers })
                   .catch((error:any) => Observable.throw(error.json().error || 'Server error'));
  }

  checkCredentials() {
    return Cookie.check('access_token');
  } 

  logout() {
    Cookie.delete('access_token');
    window.location.reload();
  }
}
```

在`retrieveToken`方法中，我们使用我们的客户端凭证和基本 Auth 向`/openid-connect/token`端点发送一个`POST`来获取访问令牌。参数以 URL 编码的格式发送。在我们获得访问令牌之后，**我们将它存储在一个 cookie 中。**

cookie 存储在这里特别重要，因为我们只是将 cookie 用于存储目的，而不是直接驱动身份验证过程。这有助于防范跨站点请求伪造(CSRF)攻击和漏洞。

### 5.3.Foo 组件

最后，我们的`FooComponent`显示我们的 Foo 细节:

```java
@Component({
  selector: 'foo-details',
  providers: [AppService],  
  template: `<div class="container">
    <h1 class="col-sm-12">Foo Details</h1>
    <div class="col-sm-12">
        <label class="col-sm-3">ID</label> <span>{{foo.id}}</span>
    </div>
    <div class="col-sm-12">
        <label class="col-sm-3">Name</label> <span>{{foo.name}}</span>
    </div>
    <div class="col-sm-12">
        <button class="btn btn-primary" (click)="getFoo()" type="submit">New Foo</button>        
    </div>
  </div>`
})

export class FooComponent {
  public foo = new Foo(1,'sample foo');
  private foosUrl = 'http://localhost:8081/resource-server/api/foos/';  

  constructor(private _service:AppService) {}

  getFoo() {
    this._service.getResource(this.foosUrl+this.foo.id)
      .subscribe(
         data => this.foo = data,
         error =>  this.foo.name = 'Error');
    }
}
```

### 5.5。应用组件

我们简单的`AppComponent`充当根组件:

```java
@Component({
  selector: 'app-root',
  template: `<nav class="navbar navbar-default">
    <div class="container-fluid">
      <div class="navbar-header">
        <a class="navbar-brand" href="/">Spring Security Oauth - Authorization Code</a>
      </div>
    </div>
  </nav>
  <router-outlet></router-outlet>`
})

export class AppComponent { } 
```

我们包装所有组件、服务和路线的地方:

```java
@NgModule({
  declarations: [
    AppComponent,
    HomeComponent,
    FooComponent    
  ],
  imports: [
    BrowserModule,
    HttpClientModule,
    RouterModule.forRoot([
     { path: '', component: HomeComponent, pathMatch: 'full' }], {onSameUrlNavigation: 'reload'})
  ],
  providers: [],
  bootstrap: [AppComponent]
})
export class AppModule { } 
```

## 7。运行前端

1.要运行我们的任何前端模块，我们需要首先构建应用程序:

```java
mvn clean install
```

2.然后我们需要导航到我们的 Angular 应用程序目录:

```java
cd src/main/resources
```

3.最后，我们将启动我们的应用:

```java
npm start
```

默认情况下，服务器将在端口 4200 上启动；要更改任何模块的端口，请更改:

```java
"start": "ng serve"
```

例如，在`package.json;` 中，要让它在端口 8089 上运行，添加:

```java
"start": "ng serve --port 8089"
```

## 8。结论

在本文中，我们学习了如何使用 OAuth2 授权我们的应用程序。

本教程的完整实现可以在 GitHub 项目中找到。