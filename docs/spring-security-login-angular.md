# 带角度的 Spring 安全登录页面

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-security-login-angular>

## 1。概述

在本教程中，我们将使用 Spring Security 和创建一个**登录页面**

*   安古斯
*   角度 2、4、5 和 6

我们在这里将要讨论的示例应用程序由一个与 REST 服务通信的客户端应用程序组成，它通过基本的 HTTP 身份验证进行保护。

## 2。Spring 安全配置

首先，让我们用 Spring Security 和基本 Auth 设置 REST API:

这是它的配置方式:

```java
@Configuration
@EnableWebSecurity
public class BasicAuthConfiguration 
  extends WebSecurityConfigurerAdapter {

    @Override
    protected void configure(AuthenticationManagerBuilder auth)
      throws Exception {
        auth
          .inMemoryAuthentication()
          .withUser("user")
          .password("password")
          .roles("USER");
    }

    @Override
    protected void configure(HttpSecurity http) 
      throws Exception {
        http.csrf().disable()
          .authorizeRequests()
          .antMatchers("/login").permitAll()
          .anyRequest()
          .authenticated()
          .and()
          .httpBasic();
    }
}
```

现在让我们创建端点。我们的 REST 服务将有两个——一个用于登录，另一个用于获取用户数据:

```java
@RestController
@CrossOrigin
public class UserController {

    @RequestMapping("/login")
    public boolean login(@RequestBody User user) {
        return
          user.getUserName().equals("user") && user.getPassword().equals("password");
    }

    @RequestMapping("/user")
    public Principal user(HttpServletRequest request) {
        String authToken = request.getHeader("Authorization")
          .substring("Basic".length()).trim();
        return () ->  new String(Base64.getDecoder()
          .decode(authToken)).split(":")[0];
    }
}
```

同样，如果您对实现 OAuth2 服务器进行授权感兴趣，也可以查看我们的另一个关于 [Spring Security OAuth2](/web/20220924195726/https://www.baeldung.com/sso-spring-security-oauth2) 的教程。

## 3。设置角度客户端

现在我们已经创建了 REST 服务，让我们用不同版本的 Angular 客户机来设置登录页面。

我们将在这里看到的例子使用`npm`进行依赖管理，使用`nodejs`运行应用程序。

Angular 使用单页面架构，其中所有子组件(在我们的例子中是登录和主页组件)都被注入到一个公共的父 DOM 中。

与使用 JavaScript 的 AngularJS 不同，Angular version 2 以后使用 TypeScript 作为其主要语言。因此，应用程序还需要某些支持文件，这些文件是它正常工作所必需的。

由于 Angular 的增量增强，每个版本所需的文件都不同。

让我们熟悉一下其中的每一项:

*   `systemjs.config.js`–系统配置(版本 2)
*   `package.json`–节点模块依赖性(版本 2 及以上)
*   `tsconfig.json`–根级打字稿配置(版本 2 及以上)
*   `tsconfig.app.json`–应用程序级类型脚本配置(版本 4 及以上)
*   cli`.json`–角度 CLI 配置(版本 4 和 5)
*   `angular.json`–角度 CLI 配置(版本 6 及以上)

## 4。登录页面

### 4.1。使用角尺

让我们创建`index.html`文件，并向其中添加相关的依赖项:

```java
<html ng-app="app">
<body>
    <div ng-view></div>

    <script src="//code.jquery.com/jquery-3.1.1.min.js"></script>
    <script src="//code.angularjs.org/1.6.0/angular.min.js"></script>
    <script src="//code.angularjs.org/1.6.0/angular-route.min.js"></script>
    <script src="app.js"></script>
    <script src="home/home.controller.js"></script>
    <script src="login/login.controller.js"></script>
</body>
</html>
```

因为这是一个单页面应用程序，所以所有子组件都将基于路由逻辑添加到具有`ng-view`属性的 div 元素中。

现在让我们创建定义 URL 到组件映射的`app.js`:

```java
(function () {
    'use strict';

    angular
        .module('app', ['ngRoute'])
        .config(config)
        .run(run);

    config.$inject = ['$routeProvider', '$locationProvider'];
    function config($routeProvider, $locationProvider) {
        $routeProvider.when('/', {
            controller: 'HomeController',
            templateUrl: 'home/home.view.html',
            controllerAs: 'vm'
        }).when('/login', {
            controller: 'LoginController',
            templateUrl: 'login/login.view.html',
            controllerAs: 'vm'
        }).otherwise({ redirectTo: '/login' });
    }

    run.$inject = ['$rootScope', '$location', '$http', '$window'];
    function run($rootScope, $location, $http, $window) {
        var userData = $window.sessionStorage.getItem('userData');
        if (userData) {
            $http.defaults.headers.common['Authorization']
              = 'Basic ' + JSON.parse(userData).authData;
        }

        $rootScope
        .$on('$locationChangeStart', function (event, next, current) {
            var restrictedPage
              = $.inArray($location.path(), ['/login']) === -1;
            var loggedIn
              = $window.sessionStorage.getItem('userData');
            if (restrictedPage && !loggedIn) {
                $location.path('/login');
            }
        });
    }
})();
```

登录组件由两个文件组成，`login.controller.js`和`login.view.html.`

我们来看第一个:

```java
<h2>Login</h2>
<form name="form" ng-submit="vm.login()" role="form">
    <div>
        <label for="username">Username</label>
        <input type="text" name="username"
          id="username" ng-model="vm.username" required />
        <span ng-show="form.username.$dirty
          && form.username.$error.required">Username is required</span>
    </div>
    <div>
        <label for="password">Password</label>
        <input type="password"
          name="password" id="password" ng-model="vm.password" required />
        <span ng-show="form.password.$dirty
          && form.password.$error.required">Password is required</span>
    </div>
    <div class="form-actions">
        <button type="submit"
          ng-disabled="form.$invalid || vm.dataLoading">Login</button>
    </div>
</form>
```

第二个是:

```java
(function () {
    'use strict';
    angular
        .module('app')
        .controller('LoginController', LoginController);

    LoginController.$inject = ['$location', '$window', '$http'];
    function LoginController($location, $window, $http) {
        var vm = this;
        vm.login = login;

        (function initController() {
            $window.localStorage.setItem('token', '');
        })();

        function login() {
            $http({
                url: 'http://localhost:8082/login',
                method: "POST",
                data: { 
                    'userName': vm.username,
                    'password': vm.password
                }
            }).then(function (response) {
                if (response.data) {
                    var token
                      = $window.btoa(vm.username + ':' + vm.password);
                    var userData = {
                        userName: vm.username,
                        authData: token
                    }
                    $window.sessionStorage.setItem(
                      'userData', JSON.stringify(userData)
                    );
                    $http.defaults.headers.common['Authorization']
                      = 'Basic ' + token;
                    $location.path('/');
                } else {
                    alert("Authentication failed.")
                }
            });
        };
    }
})();
```

控制器将通过传递用户名和密码来调用 REST 服务。身份验证成功后，它将对用户名和密码进行编码，并将编码后的令牌存储在会话存储中以备将来使用。

与登录组件类似，主页组件也由两个文件组成，`home.view.html` **:**

```java
<h1>Hi {{vm.user}}!</h1>
<p>You're logged in!!</p>
<p><a href="#!/login" class="btn btn-primary" ng-click="logout()">Logout</a></p>
```

而`home.controller.js:`

```java
(function () {
    'use strict';
    angular
        .module('app')
        .controller('HomeController', HomeController);

    HomeController.$inject = ['$window', '$http', '$scope'];
    function HomeController($window, $http, $scope) {
        var vm = this;
        vm.user = null;

        initController();

        function initController() {
            $http({
                url: 'http://localhost:8082/user',
                method: "GET"
            }).then(function (response) {
                vm.user = response.data.name;
            }, function (error) {
                console.log(error);
            });
        };

        $scope.logout = function () {
            $window.sessionStorage.setItem('userData', '');
            $http.defaults.headers.common['Authorization'] = 'Basic';
        }
    }
})();
```

家庭控制器将通过传递`Authorization`报头来请求用户数据。只有当令牌有效时，我们的 REST 服务才会返回用户数据。

现在让我们安装`http-server`来运行 Angular 应用程序:

```java
npm install http-server --save
```

安装完成后，我们可以在命令提示符下打开项目根文件夹并执行命令:

```java
http-server -o
```

### 4.2。使用角度版本 2、4、5

版本 2 中的`index.html`与 AngularJS 版本略有不同:

```java
<!DOCTYPE html>
<html>
<head>
    <base href="/" />
    <script src="node_modules/core-js/client/shim.min.js"></script>
    <script src="node_modules/zone.js/dist/zone.js"></script>
    <script src="node_modules/systemjs/dist/system.src.js"></script>

    <script src="systemjs.config.js"></script>
    <script>
        System.import('app').catch(function (err) { console.error(err); });
    </script>
</head>
<body>
    <app>Loading...</app>
</body>
</html>
```

`main.ts`是应用程序的主入口点。它引导应用程序模块，因此浏览器加载登录页面:

```java
platformBrowserDynamic().bootstrapModule(AppModule);
```

`app.routing.ts`负责应用程序路由:

```java
const appRoutes: Routes = [
    { path: '', component: HomeComponent },
    { path: 'login', component: LoginComponent },
    { path: '**', redirectTo: '' }
];

export const routing = RouterModule.forRoot(appRoutes);
```

`app.module.ts`声明组件并导入相关模块:

```java
@NgModule({
    imports: [
        BrowserModule,
        FormsModule,
        HttpModule,
        routing
    ],
    declarations: [
        AppComponent,
        HomeComponent,
        LoginComponent
    ],
    bootstrap: [AppComponent]
})

export class AppModule { }
```

因为我们正在创建一个单页面应用程序，所以让我们创建一个根组件，将所有子组件添加到其中:

```java
@Component({
    selector: 'app-root',
    templateUrl: './app.component.html'
})

export class AppComponent { }
```

`app.component.html`将只有一个`<router-outlet>`标签。Angular 使用此标签作为其位置路由机制。

现在让我们在`login.component.ts:`中创建登录组件及其相应的模板

```java
@Component({
    selector: 'login',
    templateUrl: './app/login/login.component.html'
})

export class LoginComponent implements OnInit {
    model: any = {};

    constructor(
        private route: ActivatedRoute,
        private router: Router,
        private http: Http
    ) { }

    ngOnInit() {
        sessionStorage.setItem('token', '');
    }

    login() {
        let url = 'http://localhost:8082/login';
        let result = this.http.post(url, {
            userName: this.model.username,
            password: this.model.password
        }).map(res => res.json()).subscribe(isValid => {
            if (isValid) {
                sessionStorage.setItem(
                  'token',
                  btoa(this.model.username + ':' + this.model.password)
                );
                this.router.navigate(['']);
            } else {
                alert("Authentication failed.");
            }
        });
    }
}
```

最后，我们来看看`login.component.html`:

```java
<form name="form" (ngSubmit)="f.form.valid && login()" #f="ngForm" novalidate>
    <div [ngClass]="{ 'has-error': f.submitted && !username.valid }">
        <label for="username">Username</label>
        <input type="text"
          name="username" [(ngModel)]="model.username"
            #username="ngModel" required />
        <div *ngIf="f.submitted
          && !username.valid">Username is required</div>
    </div>
    <div [ngClass]="{ 'has-error': f.submitted && !password.valid }">
        <label for="password">Password</label>
        <input type="password"
          name="password" [(ngModel)]="model.password"
            #password="ngModel" required />
        <div *ngIf="f.submitted
          && !password.valid">Password is required</div>
    </div>
    <div>
        <button [disabled]="loading">Login</button>
    </div>
</form>
```

### 4.3。使用角度 6

Angular team 在版本 6 中做了一些增强。由于这些变化，我们的示例与其他版本相比也会有一些不同。关于版本 6，我们的例子中唯一的变化是在服务调用部分。

**代替`HttpModule`，版本 6 从** **`@angular/common/http.`** 导入`HttpClientModule`

服务调用部分也将与旧版本略有不同:

```java
this.http.post<Observable<boolean>>(url, {
    userName: this.model.username,
    password: this.model.password
}).subscribe(isValid => {
    if (isValid) {
        sessionStorage.setItem(
          'token', 
          btoa(this.model.username + ':' + this.model.password)
        );
	this.router.navigate(['']);
    } else {
        alert("Authentication failed.")
    }
});
```

## 5。结论

我们已经学习了如何用 Angular 实现一个 Spring 安全登录页面。从版本 4 开始，我们可以利用 Angular CLI 项目来简化开发和测试。

像往常一样，我们在这里讨论的所有例子都可以在 GitHub 项目中找到。