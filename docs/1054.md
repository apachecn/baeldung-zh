# 具有 Spring Security OAuth 的前端应用程序–授权代码流

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-security-oauth-authorization-code-flow>

## 1。概述

在本教程中，我们将继续我们的 [Spring Security OAuth 系列](/web/20221012100324/https://www.baeldung.com/spring-security-oauth)，为授权代码流构建一个简单的前端。

请记住，这里的重点是客户端；看看[Spring REST API+oauth 2+angular js](/web/20221012100324/https://www.baeldung.com/rest-api-spring-oauth2-angular-legacy)的文章——查看授权和资源服务器的详细配置。

## 2。授权服务器

在我们到达前端之前，我们需要在我们的授权服务器配置中添加我们的客户端详细信息:

```
@Configuration
@EnableAuthorizationServer
public class OAuth2AuthorizationServerConfig extends AuthorizationServerConfigurerAdapter {

    @Override
    public void configure(ClientDetailsServiceConfigurer clients) throws Exception {
        clients.inMemory()
               .withClient("fooClientId")
               .secret(passwordEncoder().encode("secret"))
               .authorizedGrantTypes("authorization_code")
               .scopes("foo", "read", "write")
               .redirectUris("http://localhost:8089/")
...
```

请注意我们现在是如何启用授权代码授权类型的，具体如下:

*   我们的客户 id 是`fooClientId`
*   我们的范围是`foo`、`read `和`write`
*   重定向 URI 是`http://localhost:8089/`(对于我们的前端应用程序，我们将使用端口 8089)

## 3。前端

现在，让我们开始构建简单的前端应用程序。

由于我们将在这里的应用程序中使用 Angular 6，我们需要在我们的 Spring Boot 应用程序中使用`frontend-maven-plugin`插件:

```
<plugin>
    <groupId>com.github.eirslett</groupId>
    <artifactId>frontend-maven-plugin</artifactId>
    <version>1.6</version>

    <configuration>
        <nodeVersion>v8.11.3</nodeVersion>
        <npmVersion>6.1.0</npmVersion>
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
```

注意，自然地，我们需要首先在我们的机器上安装[node . js](https://web.archive.org/web/20221012100324/https://nodejs.org/en/)；我们将使用 Angular CLI 为我们的应用程序生成基础:

ng new authCode

## 4。角度模块

现在，让我们详细讨论我们的角度模块。

下面是我们简单的`AppModule`:

```
import { BrowserModule } from '@angular/platform-browser';
import { NgModule } from '@angular/core';
import { HttpClientModule } from '@angular/common/http';
import { RouterModule }   from '@angular/router';
import { AppComponent } from './app.component';
import { HomeComponent } from './home.component';
import { FooComponent } from './foo.component';

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

我们的模块由三个组件和一个服务组成，我们将在下面的部分中讨论它们

### 4.1。应用组件

让我们从我们的`AppComponent`开始，它是根组件:

```
import {Component} from '@angular/core';

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

export class AppComponent {}
```

### 4.2。家用组件

接下来是我们的主要组件，`HomeComponent`:

```
import {Component} from '@angular/core';
import {AppService} from './app.service'

@Component({
    selector: 'home-header',
    providers: [AppService],
  template: `<div class="container" >
    <button *ngIf="!isLoggedIn" class="btn btn-primary" (click)="login()" type="submit">Login</button>
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

    constructor(
        private _service:AppService){}

    ngOnInit(){
        this.isLoggedIn = this._service.checkCredentials();    
        let i = window.location.href.indexOf('code');
        if(!this.isLoggedIn && i != -1){
            this._service.retrieveToken(window.location.href.substring(i + 5));
        }
    }

    login() {
        window.location.href = 'http://localhost:8081/spring-security-oauth-server/oauth/authorize?response_type=code&client;_id=' + this._service.clientId + '&redirect;_uri='+ this._service.redirectUri;
    }

    logout() {
        this._service.logout();
    }
}
```

请注意:

*   如果用户未登录，将只显示登录按钮
*   登录按钮将用户重定向到授权 URL
*   当用户被重定向回授权代码时，我们使用该代码检索访问令牌

### 4.3。Foo 组件

我们的第三个也是最后一个组件是`FooComponent`；这将显示从资源服务器获得的`Foo`资源:

```
import { Component } from '@angular/core';
import {AppService, Foo} from './app.service'

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
    private foosUrl = 'http://localhost:8082/spring-security-oauth-resource/foos/';  

    constructor(private _service:AppService) {}

    getFoo(){
        this._service.getResource(this.foosUrl+this.foo.id)
         .subscribe(
            data => this.foo = data,
            error =>  this.foo.name = 'Error');
    }
}
```

### 4.4。应用服务

现在，让我们来看看`AppService`:

```
import {Injectable} from '@angular/core';
import { Cookie } from 'ng2-cookies';
import { HttpClient, HttpHeaders } from '@angular/common/http';
import { Observable } from 'rxjs/Observable';
import 'rxjs/add/operator/catch';
import 'rxjs/add/operator/map';

export class Foo {
  constructor(
    public id: number,
    public name: string) { }
} 

@Injectable()
export class AppService {
   public clientId = 'fooClientId';
   public redirectUri = 'http://localhost:8089/';

  constructor(
    private _http: HttpClient){}

  retrieveToken(code){
    let params = new URLSearchParams();   
    params.append('grant_type','authorization_code');
    params.append('client_id', this.clientId);
    params.append('redirect_uri', this.redirectUri);
    params.append('code',code);

    let headers = new HttpHeaders({'Content-type': 'application/x-www-form-urlencoded; charset=utf-8', 'Authorization': 'Basic '+btoa(this.clientId+":secret")});
     this._http.post('http://localhost:8081/spring-security-oauth-server/oauth/token', params.toString(), { headers: headers })
    .subscribe(
      data => this.saveToken(data),
      err => alert('Invalid Credentials')
    ); 
  }

  saveToken(token){
    var expireDate = new Date().getTime() + (1000 * token.expires_in);
    Cookie.set("access_token", token.access_token, expireDate);
    console.log('Obtained Access token');
    window.location.href = 'http://localhost:8089';
  }

  getResource(resourceUrl) : Observable<any>{
    var headers = new HttpHeaders({'Content-type': 'application/x-www-form-urlencoded; charset=utf-8', 'Authorization': 'Bearer '+Cookie.get('access_token')});
    return this._http.get(resourceUrl,{ headers: headers })
                   .catch((error:any) => Observable.throw(error.json().error || 'Server error'));
  }

  checkCredentials(){
    return Cookie.check('access_token');
  } 

  logout() {
    Cookie.delete('access_token');
    window.location.reload();
  }
}
```

让我们在这里快速概括一下我们的实现:

*   `checkCredentials()`:检查用户是否登录
*   `retrieveToken()`:使用授权码获取接入令牌
*   `saveToken()`:将访问令牌保存在 cookie 中
*   `getResource()`:使用其 ID 获取 Foo 详细信息
*   `logout()`:删除访问令牌 cookie

## 5。运行应用程序

为了运行我们的应用程序并确保一切正常运行，我们需要:

*   首先，在端口 8081 上运行授权服务器
*   然后，在端口 8082 上运行资源服务器
*   最后，运行前端

我们需要首先构建我们的应用程序:

```
mvn clean install
```

然后将目录更改为 src/main/resources:

```
cd src/main/resources
```

然后在端口 8089 上运行我们的应用程序:

```
npm start
```

## 6。结论

我们学习了如何使用 Spring 和 Angular 6 为授权代码流构建一个简单的前端客户端。

和往常一样，完整的源代码可以在 GitHub 上找到[。](https://web.archive.org/web/20221012100324/https://github.com/Baeldung/spring-security-oauth/tree/master/oauth-legacy)