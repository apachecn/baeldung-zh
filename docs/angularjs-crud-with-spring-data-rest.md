# AngularJS CRUD 应用程序与 Spring 数据休息

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/angularjs-crud-with-spring-data-rest>

## 1。概述

在本教程中，我们将创建一个简单的 CRUD 应用程序示例，前端使用 AngularJS，后端使用 Spring Data REST。

## 2。创建 REST 数据服务

为了创建对持久性的支持，我们将利用 Spring Data REST 规范，它将使我们能够在数据模型上执行 CRUD 操作。

您可以在[Spring Data REST 简介](/web/20220628094913/https://www.baeldung.com/spring-data-rest-intro)中找到关于如何设置 REST 端点的所有必要信息。在本文中，我们将重用我们为入门教程设置的现有项目。

对于持久性，我们将使用内存数据库中的`H2`。

作为数据模型，前一篇文章定义了一个`WebsiteUser`类，具有`id`、`name`和`email`属性以及一个名为`UserRepository`的存储库接口。

定义这个接口指示 Spring 创建对公开 REST 集合资源和项目资源的支持。让我们更仔细地看看我们可用的端点，因为我们稍后将从`AngularJS`调用。

### 2.1。收藏资源

所有用户的列表将在端点`/users`对我们可用。可以使用 GET 方法调用此 URL，它将返回以下形式的 JSON 对象:

```java
{
  "_embedded" : {
    "users" : [ {
      "name" : "Bryan",
      "age" : 20,
      "_links" : {
        "self" : {
          "href" : "http://localhost:8080/users/1"
        },
        "User" : {
          "href" : "http://localhost:8080/users/1"
        }
      }
    }, 
...
    ]
  }
}
```

### 2.2。物品资源

通过使用不同的 HTTP 方法和请求有效负载访问形式为`/users/{userID}`的 URL，可以操作单个`WebsiteUser` 对象。

为了检索一个`WebsiteUser`对象，我们可以用 GET 方法访问`/users/{userID}`。这将返回以下形式的 JSON 对象:

```java
{
  "name" : "Bryan",
  "email" : "[[email protected]](/web/20220628094913/https://www.baeldung.com/cdn-cgi/l/email-protection)",
  "_links" : {
    "self" : {
      "href" : "http://localhost:8080/users/1"
    },
    "User" : {
      "href" : "http://localhost:8080/users/1"
    }
  }
}
```

要添加一个新的`WebsiteUser`，我们需要用 POST 方法调用`/users`。新的`WebsiteUser`记录的属性将作为一个 JSON 对象添加到请求体中:

```java
{name: "Bryan", email: "[[email protected]](/web/20220628094913/https://www.baeldung.com/cdn-cgi/l/email-protection)"}
```

如果没有错误，这个 URL 返回一个创建的状态代码 201。

如果我们想要更新`WebsiteUser` 记录的属性，我们需要使用 PATCH 方法和包含新值的请求体来调用 URL `/users/{UserID}`:

```java
{name: "Bryan", email: "[[email protected]](/web/20220628094913/https://www.baeldung.com/cdn-cgi/l/email-protection)"}
```

要删除一个`WebsiteUser`记录，我们可以用 delete 方法调用 URL `/users/{UserID}`。如果没有错误，将返回状态代码 204 NO CONTENT。

### 2.3。MVC 配置

我们还将添加一个基本的 MVC 配置来显示应用程序中的 html 文件:

```java
@Configuration
@EnableWebMvc
public class MvcConfig implements WebMvcConfigurer {

    public MvcConfig(){
        super();
    }

    @Override
    public void configureDefaultServletHandling(
      DefaultServletHandlerConfigurer configurer) {
        configurer.enable();
    }

   @Bean
   WebServerFactoryCustomizer<ConfigurableServletWebServerFactory> enableDefaultServlet() {
     return (factory) -> factory.setRegisterDefaultServlet(true);
   }
}
```

### 2.4。允许跨来源请求

如果我们想要独立于 REST API 部署`AngularJS`前端应用程序，那么我们需要启用跨来源请求。

从版本 1.5.0.RELEASE 开始，Spring Data REST 增加了对此的支持。

```java
@CrossOrigin
@RepositoryRestResource(collectionResourceRel = "users", path = "users")
public interface UserRepository extends CrudRepository<WebsiteUser, Long> {}
```

因此，在来自 REST 端点的每个响应上，将添加一个标题`Access-Control-Allow-Origin`。

## 3。创建 AngularJS 客户端

为了创建 CRUD 应用程序的前端，我们将使用`AngularJS`——一个众所周知的 JavaScript 框架，它简化了前端应用程序的创建。

为了使用`AngularJS`，我们首先需要在我们的 html 页面中包含 `angular.min.js`文件，它将被称为`users.html`:

```java
<script 
  src="https://ajax.googleapis.com/ajax/libs/angularjs/1.5.6/angular.min.js">
</script>
```

接下来，我们需要创建一个角度模块、控制器和服务来调用 REST 端点并显示返回的数据。

这些将被放在一个名为 `app.js`的 JavaScript 文件中，该文件也需要包含在 `users.html`页面中:

```java
<script src="view/app.js"></script>
```

### 3.1。角度服务

首先，让我们创建一个名为`UserCRUDService`的角度服务，它将利用注入的`AngularJS` `$http`服务来调用服务器。每个调用都将放在一个单独的方法中。

让我们来看一下使用`/users/{userID}`端点定义通过 id 检索用户的方法:

```java
app.service('UserCRUDService', [ '$http', function($http) {

    this.getUser = function getUser(userId) {
        return $http({
            method : 'GET',
            url : 'users/' + userId
        });
    }
} ]);
```

接下来，让我们定义`addUser`方法，该方法向`/users` URL 发出 POST 请求，并发送`data`属性中的用户值:

```java
this.addUser = function addUser(name, email) {
    return $http({
        method : 'POST',
        url : 'users',
        data : {
            name : name,
            email: email
        }
    });
}
```

`updateUser`方法类似于上面的方法，除了它将有一个`id` 参数并发出一个补丁请求:

```java
this.updateUser = function updateUser(id, name, email) {
    return $http({
        method : 'PATCH',
        url : 'users/' + id,
        data : {
            name : name,
            email: email
        }
    });
}
```

删除`WebsiteUser` 记录的方法将发出删除请求:

```java
this.deleteUser = function deleteUser(id) {
    return $http({
        method : 'DELETE',
        url : 'users/' + id
    })
}
```

最后，让我们看看检索整个用户列表的方法:

```java
this.getAllUsers = function getAllUsers() {
    return $http({
        method : 'GET',
        url : 'users'
    });
}
```

所有这些服务方法都将被一个`AngularJS`控制器调用。

### 3.2。角度控制器

我们将创建一个`UserCRUDCtrl` `AngularJS`控制器，它将注入一个`UserCRUDService`，并使用服务方法从服务器获得响应，处理`success`和`error`案例，并设置包含响应数据的`$scope`变量，以便在 HTML 页面中显示响应数据。

我们来看看调用`getUser(userId)`服务函数的`getUser()`函数，定义了两个回调方法，分别在成功和错误的情况下使用。如果服务器请求成功，那么响应被保存在一个`user`变量中；否则，将处理错误消息:

```java
app.controller('UserCRUDCtrl', ['$scope','UserCRUDService', 
  function ($scope,UserCRUDService) {
      $scope.getUser = function () {
          var id = $scope.user.id;
          UserCRUDService.getUser($scope.user.id)
            .then(function success(response) {
                $scope.user = response.data;
                $scope.user.id = id;
                $scope.message='';
                $scope.errorMessage = '';
            },
    	    function error (response) {
                $scope.message = '';
                if (response.status === 404){
                    $scope.errorMessage = 'User not found!';
                }
                else {
                    $scope.errorMessage = "Error getting user!";
                }
            });
      };
}]);
```

`addUser()`函数将调用相应的服务函数并处理响应:

```java
$scope.addUser = function () {
    if ($scope.user != null && $scope.user.name) {
        UserCRUDService.addUser($scope.user.name, $scope.user.email)
          .then (function success(response){
              $scope.message = 'User added!';
              $scope.errorMessage = '';
          },
          function error(response){
              $scope.errorMessage = 'Error adding user!';
              $scope.message = '';
        });
    }
    else {
        $scope.errorMessage = 'Please enter a name!';
        $scope.message = '';
    }
}
```

`updateUser()`和`deleteUser()`功能与上图类似:

```java
$scope.updateUser = function () {
    UserCRUDService.updateUser($scope.user.id, 
      $scope.user.name, $scope.user.email)
      .then(function success(response) {
          $scope.message = 'User data updated!';
          $scope.errorMessage = '';
      },
      function error(response) {
          $scope.errorMessage = 'Error updating user!';
          $scope.message = '';
      });
}

$scope.deleteUser = function () {
    UserCRUDService.deleteUser($scope.user.id)
      .then (function success(response) {
          $scope.message = 'User deleted!';
          $scope.User = null;
          $scope.errorMessage='';
      },
      function error(response) {
          $scope.errorMessage = 'Error deleting user!';
          $scope.message='';
      });
}
```

最后，让我们定义检索用户列表的函数，并将其存储在`users` 变量中:

```java
$scope.getAllUsers = function () {
    UserCRUDService.getAllUsers()
      .then(function success(response) {
          $scope.users = response.data._embedded.users;
          $scope.message='';
          $scope.errorMessage = '';
      },
      function error (response) {
          $scope.message='';
          $scope.errorMessage = 'Error getting users!';
      });
}
```

### 3.3。HTML 页面

`users.html`页面将利用上一节定义的控制器功能和存储的变量。

首先，为了使用 Angular 模块，我们需要设置 `ng-app`属性:

```java
<html ng-app="app">
```

然后，为了避免每次使用控制器的函数时都键入`UserCRUDCtrl.getUser()`，我们可以将 HTML 元素包装在一个带有`ng-controller`属性集的`div`中:

```java
<div ng-controller="UserCRUDCtrl">
```

让我们创建一个表单，它将输入并显示我们想要操作的`WebiteUser` 对象的值。其中每一个都有一个`ng-model`属性集，将它绑定到属性值:

```java
<table>
    <tr>
        <td width="100">ID:</td>
        <td><input type="text" id="id" ng-model="user.id" /></td>
    </tr>
    <tr>
        <td width="100">Name:</td>
        <td><input type="text" id="name" ng-model="user.name" /></td>
    </tr>
    <tr>
        <td width="100">Age:</td>
        <td><input type="text" id="age" ng-model="user.email" /></td>
    </tr>
</table>
```

例如，将`id`输入绑定到`user.id`变量意味着每当输入的值改变时，这个值就被设置在`user.id`变量中，反之亦然。

接下来，让我们使用`ng-click`属性来定义将触发每个 CRUD 控制器函数调用的链接:

```java
<a ng-click="getUser(user.id)">Get User</a>
<a ng-click="updateUser(user.id,user.name,user.email)">Update User</a>
<a ng-click="addUser(user.name,user.email)">Add User</a>
<a ng-click="deleteUser(user.id)">Delete User</a>
```

最后，让我们按名称显示完整的用户列表:

```java
<a ng-click="getAllUsers()">Get all Users</a><br/><br/>
<div ng-repeat="usr in users">
{{usr.name}} {{usr.email}}
```

## 4。结论

在本教程中，我们展示了如何使用`AngularJS`和`Spring Data REST`规范创建 CRUD 应用程序。

上面例子的完整代码可以在 [GitHub 项目](https://web.archive.org/web/20220628094913/https://github.com/eugenp/tutorials/tree/master/persistence-modules/spring-data-rest)中找到。

要运行应用程序，您可以使用命令`mvn spring-boot:run`并访问 URL `/users.html`。