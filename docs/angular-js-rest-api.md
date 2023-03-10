# REST API 的简单 AngularJS 前端

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/angular-js-rest-api>

## 1。概述

在这个快速教程中，我们将学习如何从一个简单的 AngularJS 前端使用 RESTful API。

我们将在一个表中显示数据，创建一个资源，更新它，最后删除它。

## 2。REST API

首先，让我们快速看一下我们的简单 API——通过分页公开一个`Feed`资源:

*   **获取分页-** `**GET** /api/myFeeds?page={page}&size;={size}&sortDir;={dir}&sort;={propertyName}`
*   **创建-** `**POST** /api/myFeeds`
*   **更新—** `**PUT** /api/myFeeds/{id}`
*   **删除-** `**DELETE** /api/myFeeds/{id}`

这里需要注意的是，分页使用了以下 4 个参数:

*   `page`:请求页面的索引
*   `size`:每页最大记录数
*   `sort`:排序时使用的属性名称
*   `sortDir`:分拣方向

下面是一个关于`Feed`资源的例子:

```java
{
    "id":1,
    "name":"baeldung feed",
    "url":"/feed"
}
```

## 3。Feeds 页面

现在，让我们来看看我们的 feeds 页面:

```java
<script 
  src="http://ajax.googleapis.com/ajax/libs/angularjs/1.3.14/angular.min.js">
</script>
<script 
  src="http://ajax.googleapis.com/ajax/libs/angularjs/1.3.14/angular-resource.min.js">
</script>
<script th:src="@{/resources/ng-table.min.js}"></script>
<script th:src="@{/resources/mainCtrl.js}"></script>

<a href="#" ng-click="addNewFeed()">Add New RSS Feed</a>
<table ng-table="tableParams">
    <tr ng-repeat="row in $data track by row.id">
        <td data-title="'Name'" sortable="'name'">{{row.name}}</td>
        <td data-title="'Feed URL'" sortable="'url'">{{row.url}}</td>
        <td data-title="'Actions'">
            <a href="#" ng-click="editFeed(row) ">Edit</a>
            <a href="#" ng-click="confirmDelete(row.id) ">Delete</a>
        </td>
    </tr>
</table>
</div>
</body>
</html>
```

请注意，我们使用了 [ng-table](https://web.archive.org/web/20210927012325/http://ng-table.com/#/) 来显示数据——在接下来的章节中会详细介绍。

## 4。一个角度控制器

接下来，让我们看看 AngularJS 控制器:

```java
var app = angular.module('myApp', ["ngTable", "ngResource"]);
app.controller('mainCtrl', function($scope, NgTableParams, $resource) {
    ...   
});
```

请注意:

*   我们注入了`ngTable`模块，用它在一个用户友好的表格中显示我们的数据，并处理分页/排序操作
*   我们还注入了`ngResource`模块来使用它访问我们的 REST API 资源

## 5。一个 AngularJS 数据表

现在让我们快速看一下`ng-table`模块——配置如下:

```java
$scope.feed = $resource("api/myFeeds/:feedId",{feedId:'@id'});
$scope.tableParams = new NgTableParams({}, {
    getData: function(params) {
        var queryParams = {
            page:params.page() - 1, 
            size:params.count()
        };
        var sortingProp = Object.keys(params.sorting());
        if(sortingProp.length == 1){
    	    queryParams["sort"] = sortingProp[0];
    	    queryParams["sortDir"] = params.sorting()[sortingProp[0]];
        }
        return $scope.feed.query(queryParams, function(data, headers) {
            var totalRecords = headers("PAGING_INFO").split(",")[0].split("=")[1];
            params.total(totalRecords);
            return data;
        }).$promise;
    }
});
```

API 需要某种分页样式，所以我们需要在表中定制它来匹配它。我们正在使用`ng-module`中的`params`，并在这里创建我们自己的`queryParams`。

关于分页的一些附加说明:

*   `params.page()`从 1 开始，所以我们还需要确保当与 API 通信时，它变成零索引
*   `params.sorting()`返回一个对象——例如`{“name”: “asc”}`,所以我们需要将键和值分离为两个不同的参数——`sort`、`sortDir`
*   我们从响应的 HTTP 头中提取元素总数

## 6。更多操作

最后，我们可以使用`[ngResource](https://web.archive.org/web/20210927012325/https://docs.angularjs.org/api/ngResource/service/$resource)`模块执行许多操作——`$resource`确实涵盖了可用操作方面的完整 HTTP 语义。我们还可以定义我们的自定义功能。

我们已经在前一节中使用了`query`来获取提要列表。注意，`get`和`query`都做`GET`——但是`query`用于处理数组响应。

### 6.1。添加新的`Feed`

为了添加新进给，我们将使用`$resource`方法`save`，如下所示:

```java
$scope.feed = {name:"New feed", url: "http://www.example.com/feed"};

$scope.createFeed = function(){
    $scope.feeds.save($scope.feed, function(){
        $scope.tableParams.reload();
    });
}
```

### 6.2。`Feed`更新一个

我们可以通过`$resource`使用我们自己的定制方法，如下所示:

```java
$scope.feeds = $resource("api/myFeeds/:feedId",{feedId:'@id'},{
    'update': { method:'PUT' }
});

$scope.updateFeed = function(){
    $scope.feeds.update($scope.feed, function(){
        $scope.tableParams.reload();
    });
} 
```

注意我们是如何配置我们自己的`update`方法来发出`PUT`请求的。

### 6.3。删除一个`Feed`

最后，我们可以使用`delete`方法删除一个提要:

```java
$scope.confirmDelete = function(id){
    $scope.feeds.delete({feedId:id}, function(){
        $scope.tableParams.reload();
    });
}
```

## 7 号。安圭拉对话〔t1〕

现在，让我们看看如何使用`[ngDialog](https://web.archive.org/web/20210927012325/https://github.com/likeastore/ngDialog)`模块显示简单的表单来添加/更新我们的提要。

这是我们的模板，我们可以在单独的 HTML 页面或同一页面中定义它:

```java
<script type="text/ng-template" id="templateId">
<div class="ngdialog-message">
    <h2>{{feed.name}}</h2>
    <input ng-model="feed.name"/>
    <input ng-model="feed.url"/>
</div>
<div class="ngdialog-buttons mt">
    <button ng-click="save()">Save</button>
</div>
</script>
```

然后我们将打开对话框来添加/编辑提要:

```java
$scope.addNewFeed = function(){
    $scope.feed = {name:"New Feed", url: ""};
    ngDialog.open({ template: 'templateId', scope: $scope});
}
$scope.editFeed = function(row){
    $scope.feed.id = row.id;
    $scope.feed.name = row.name;
    $scope.feed.url = row.url;
    ngDialog.open({ template: 'templateId', scope: $scope});
}
$scope.save = function(){
    ngDialog.close('ngdialog1');
    if(! $scope.feed.id){
        $scope.createFeed();
    } else{
        $scope.updateFeed();
    }
}
```

请注意:

*   当用户点击对话框中的**保存**按钮时，调用`$scope.save()`
*   当用户点击 feeds 页面中的**添加新的 Feed** 按钮时，`$scope.addNewFeed()`被调用——它初始化一个新的`Feed`对象(没有`id`
*   当用户想要编辑 Feeds 表中的特定行时，调用`$scope.editFeed()`

## 8。错误处理

最后，让我们看看**如何使用 AngularJS 处理响应错误消息**。

为了全局处理服务器错误响应——而不是每个请求——我们将向`[$httpProvider](https://web.archive.org/web/20210927012325/https://docs.angularjs.org/api/ng/service/$http)`注册一个拦截器:

```java
app.config(['$httpProvider', function ($httpProvider) {
    $httpProvider.interceptors.push(function ($q,$rootScope) {
        return {
            'responseError': function (responseError) {
                $rootScope.message = responseError.data.message;
                return $q.reject(responseError);
            }
        };
    });
}]);
```

这是我们用 HTML 表示的消息:

```java
<div ng-show="message" class="alert alert-danger">
    {{message}}
</div>
```

## 9。结论

这是一篇关于使用 AngularJS 的 REST API 的快速文章。

本教程的**完整实现**可以在[的 github 项目](https://web.archive.org/web/20210927012325/https://github.com/Baeldung/reddit-app)中找到——这是一个基于 Eclipse 的项目，所以它应该很容易导入和运行。