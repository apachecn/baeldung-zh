# 带弹簧座和角度表的分页

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/pagination-with-a-spring-rest-api-and-an-angularjs-table>

## 1。概述

在本文中，我们将主要关注在一个`Spring REST API` 和一个简单的 AngularJS 前端中实现服务器端分页。

我们还将探索 Angular 中一个常用的表格网格，名为 [UI Grid](https://web.archive.org/web/20220815032736/http://ui-grid.info/) 。

## 2。依赖性

这里我们详细介绍了本文所需的各种依赖项。

### 2.1。JavaScript

为了让 Angular UI Grid 工作，我们需要在 HTML 中导入以下脚本。

*   `[Angular JS (1.5.8)](https://web.archive.org/web/20220815032736/https://ajax.googleapis.com/ajax/libs/angularjs/1.5.8/angular.min.js)`

*   `[Angular UI Grid](https://web.archive.org/web/20220815032736/https://cdn.rawgit.com/angular-ui/bower-ui-grid/master/ui-grid.min.js)`

### 2.2。肚子

对于我们的后端，我们将使用`Spring Boot`，所以我们需要以下依赖关系:

```
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-tomcat</artifactId>
    <scope>provided</scope>
</dependency>
```

**注意:**这里没有指定其他依赖项，完整列表请查看 [GitHub](https://web.archive.org/web/20220815032736/https://github.com/eugenp/tutorials/tree/master/spring-web-modules/spring-rest-angular) 项目中完整的`pom.xml`。

## 3。关于申请

该应用程序是一个简单的学生目录应用程序，允许用户在分页的表格网格中查看学生的详细信息。

该应用程序使用`Spring Boot`并在一个带有嵌入式数据库的嵌入式 Tomcat 服务器上运行。

最后，在 API 方面，有几种方法可以实现分页，在这里的 [REST Pagination in Spring 文章](/web/20220815032736/https://www.baeldung.com/rest-api-pagination-in-spring)中有所描述——强烈推荐与本文一起阅读。

我们这里的解决方案很简单——在 URI 查询中有如下的分页信息:`/student/get?page=1&size;=2`。

## 4。客户端

首先，我们需要创建客户端逻辑。

### 4.1。用户界面网格

我们的`index.html`将有我们需要的导入和表格网格的简单实现:

```
<!DOCTYPE html>
<html lang="en" ng-app="app">
    <head>
        <link rel="stylesheet" href="https://cdn.rawgit.com/angular-ui/
          bower-ui-grid/master/ui-grid.min.css">
        <script src="https://ajax.googleapis.com/ajax/libs/angularjs/
          1.5.6/angular.min.js"></script>
        <script src="https://cdn.rawgit.com/angular-ui/bower-ui-grid/
          master/ui-grid.min.js"></script>
        <script src="view/app.js"></script>
    </head>
    <body>
        <div ng-controller="StudentCtrl as vm">
            <div ui-grid="gridOptions" class="grid" ui-grid-pagination>
            </div>
        </div>
    </body>
</html>
```

让我们仔细看看代码:

*   `ng-app`–加载模块`app`的角度指令。这些下面的所有元素都将是`app` 模块的一部分
*   `ng-controller`–是加载别名为`vm.`的控制器`StudentCtrl`的角度指令，其下的所有元素都将成为`StudentCtrl`控制器的一部分
*   `ui-grid`–属于角度`ui-grid`的角度指令，使用`gridOptions`作为默认设置，`gridOptions`在`app.js`的`$scope`下声明

### 4.2。AngularJS 模块

我们先来定义一下`app.js`中的模块:

```
var app = angular.module('app', ['ui.grid','ui.grid.pagination']);
```

我们声明了`app`模块，并注入了`ui.grid`来启用 UI-Grid 功能；我们还注入了`ui.grid.pagination`来支持分页。

接下来，我们将定义控制器:

```
app.controller('StudentCtrl', ['$scope','StudentService', 
    function ($scope, StudentService) {
        var paginationOptions = {
            pageNumber: 1,
            pageSize: 5,
        sort: null
        };

    StudentService.getStudents(
      paginationOptions.pageNumber,
      paginationOptions.pageSize).success(function(data){
        $scope.gridOptions.data = data.content;
        $scope.gridOptions.totalItems = data.totalElements;
      });

    $scope.gridOptions = {
        paginationPageSizes: [5, 10, 20],
        paginationPageSize: paginationOptions.pageSize,
        enableColumnMenus:false,
    useExternalPagination: true,
        columnDefs: [
           { name: 'id' },
           { name: 'name' },
           { name: 'gender' },
           { name: 'age' }
        ],
        onRegisterApi: function(gridApi) {
           $scope.gridApi = gridApi;
           gridApi.pagination.on.paginationChanged(
             $scope, 
             function (newPage, pageSize) {
               paginationOptions.pageNumber = newPage;
               paginationOptions.pageSize = pageSize;
               StudentService.getStudents(newPage,pageSize)
                 .success(function(data){
                   $scope.gridOptions.data = data.content;
                   $scope.gridOptions.totalItems = data.totalElements;
                 });
            });
        }
    };
}]); 
```

现在让我们看看`$scope.gridOptions`中的自定义分页设置:

*   `paginationPageSizes`–定义可用的页面尺寸选项
*   `paginationPageSize`–定义默认页面尺寸
*   `enableColumnMenus`–用于启用/禁用列上的菜单
*   `useExternalPagination`–如果在服务器端分页，则需要
*   `columnDefs`–将自动映射到服务器返回的 JSON 对象的列名。从服务器返回的 JSON 对象中的字段名和定义的列名应该匹配。
*   `onRegisterApi`–在网格中注册公共方法事件的能力。这里我们注册了`gridApi.pagination.on.paginationChanged` 来告诉 UI-Grid 在页面改变时触发这个函数。

并将请求发送给 API:

```
app.service('StudentService',['$http', function ($http) {

    function getStudents(pageNumber,size) {
        pageNumber = pageNumber > 0?pageNumber - 1:0;
        return $http({
          method: 'GET',
            url: 'student/get?page='+pageNumber+'&size;='+size
        });
    }
    return {
        getStudents: getStudents
    };
}]);
```

## 5。后端和 API

### 5.1。RESTful 服务

下面是支持分页的简单 RESTful API 实现:

```
@RestController
public class StudentDirectoryRestController {

    @Autowired
    private StudentService service;

    @RequestMapping(
      value = "/student/get", 
      params = { "page", "size" }, 
      method = RequestMethod.GET
    )
    public Page<Student> findPaginated(
      @RequestParam("page") int page, @RequestParam("size") int size) {

        Page<Student> resultPage = service.findPaginated(page, size);
        if (page > resultPage.getTotalPages()) {
            throw new MyResourceNotFoundException();
        }

        return resultPage;
    }
}
```

在 Spring 4.0 中引入了 `@RestController`作为一个方便的注释，它隐式地声明了`@Controller`和`@ResponseBody.`

对于我们的 API，我们声明它接受两个参数，即`page`和 size，这两个参数也将决定返回给客户机的记录数。

我们还添加了一个简单的验证，如果页码大于总页数，将抛出一个`MyResourceNotFoundException`。

最后，我们将返回`Page` 作为响应——这是保存分页数据的 S `pring Data`的一个非常有用的组件。

### 5.2。服务实现

我们的服务将简单地根据控制器提供的页面和大小返回记录:

```
@Service
public class StudentServiceImpl implements StudentService {

    @Autowired
    private StudentRepository dao;

    @Override
    public Page<Student> findPaginated(int page, int size) {
        return dao.findAll(new PageRequest(page, size));
    }
} 
```

### 5.3。存储库实现

对于我们的持久层，我们使用嵌入式数据库和 Spring Data JPA。

首先，我们需要设置我们的持久性配置:

```
@EnableJpaRepositories("com.baeldung.web.dao")
@ComponentScan(basePackages = { "com.baeldung.web" })
@EntityScan("com.baeldung.web.entity") 
@Configuration
public class PersistenceConfig {

    @Bean
    public JdbcTemplate getJdbcTemplate() {
        return new JdbcTemplate(dataSource());
    }

    @Bean
    public DataSource dataSource() {
        EmbeddedDatabaseBuilder builder = new EmbeddedDatabaseBuilder();
        EmbeddedDatabase db = builder
          .setType(EmbeddedDatabaseType.HSQL)
          .addScript("db/sql/data.sql")
          .build();
        return db;
    }
} 
```

持久性配置很简单——我们需要`@EnableJpaRepositories` 扫描指定的包并找到我们的 Spring Data JPA 存储库接口。

我们在这里用`@ComponentScan` 来自动扫描所有的 beans，用 `have @EntityScan` (来自 Spring Boot)来扫描实体类。

我们还声明了我们的简单数据源——使用一个嵌入式数据库，该数据库将运行启动时提供的 SQL 脚本。

现在是时候创建我们的数据存储库了:

```
public interface StudentRepository extends JpaRepository<Student, Long> {} 
```

这基本上是我们在这里需要做的所有事情；如果你想更深入地了解如何设置和使用强大的 Spring 数据 JPA，请阅读这里的指南。

## 6。分页请求和响应

当调用 API `– http://localhost:8080/student/get?page=1&size;=5`时，JSON 响应看起来会像这样:

```
{
    "content":[
        {"studentId":"1","name":"Bryan","gender":"Male","age":20},
        {"studentId":"2","name":"Ben","gender":"Male","age":22},
        {"studentId":"3","name":"Lisa","gender":"Female","age":24},
        {"studentId":"4","name":"Sarah","gender":"Female","age":26},
        {"studentId":"5","name":"Jay","gender":"Male","age":20}
    ],
    "last":false,
    "totalElements":20,
    "totalPages":4,
    "size":5,
    "number":0,
    "sort":null,
    "first":true,
    "numberOfElements":5
} 
```

这里需要注意的一点是，服务器返回了一个`org.springframework.data.domain.Page` DTO，包装了我们的`Student`资源。

`Page` 对象将具有以下字段:

*   `last`–如果是最后一页，则设置为`true`，否则为假
*   `first`–如果是第一页，则设置为`true`，否则为假
*   `totalElements`–行/记录的总数。在我们的例子中，我们将它传递给`ui-grid`选项`$scope.gridOptions.totalItems` ，以确定有多少页面可用
*   `totalPages`–从(`totalElements / size`)中得出的总页数
*   `size`–每页的记录数，通过参数`size`从客户端传递
*   `number`–客户端发送的页码，在我们的响应中，数字是 0，因为在我们的后端，我们使用了一个数组`Student`，这是一个从零开始的索引，所以在我们的后端，我们将页码减 1
*   `sort`–页面的排序参数
*   `numberOfElements`–页面返回的行数/记录数

## 7。测试分页

现在让我们为我们的分页逻辑设置一个测试，使用[RestAssured](https://web.archive.org/web/20220815032736/https://github.com/rest-assured/rest-assured)；要了解更多关于 `RestAssured`的信息，你可以看看这个[教程](/web/20220815032736/https://www.baeldung.com/rest-assured-tutorial)。

### 7.1。准备测试

为了便于开发我们的测试类，我们将添加静态导入:

```
io.restassured.RestAssured.*
io.restassured.matcher.RestAssuredMatchers.*
org.hamcrest.Matchers.*
```

接下来，我们将设置启用弹簧的测试:

```
@RunWith(SpringJUnit4ClassRunner.class)
@SpringApplicationConfiguration(classes = Application.class)
@WebAppConfiguration
@IntegrationTest("server.port:8888") 
```

`@SpringApplicationConfiguration`帮助 Spring 知道如何加载`ApplicationContext,`在这种情况下，我们使用`Application.java`来配置我们的`ApplicationContext.`

定义了`@WebAppConfiguration`来告诉 Spring 要加载的`ApplicationContext`应该是一个`WebApplicationContext.`

并且`@IntegrationTest`被定义为在运行测试时触发应用程序启动，这使得我们的 REST 服务可用于测试。

### 7.2。测试

这是我们的第一个测试案例:

```
@Test
public void givenRequestForStudents_whenPageIsOne_expectContainsNames() {
    given().params("page", "0", "size", "2").get(ENDPOINT)
      .then()
      .assertThat().body("content.name", hasItems("Bryan", "Ben"));
} 
```

上面的测试用例是为了测试当页面 1 和大小 2 被传递给 REST 服务时，从服务器返回的 JSON 内容应该具有名称`Bryan`和`Ben.`

让我们分析一下测试用例:

*   `given`—`RestAssured`的一部分，用于开始构建请求，也可以使用`with()`
*   `get`—`RestAssured`的一部分，如果被使用，触发一个 get 请求，使用 post()进行 post 请求
*   `hasItems`–检查值是否匹配的 hamcrest 部分

我们增加了几个测试用例:

```
@Test
public void givenRequestForStudents_whenResourcesAreRetrievedPaged_thenExpect200() {
    given().params("page", "0", "size", "2").get(ENDPOINT)
      .then()
      .statusCode(200);
}
```

该测试断言，当实际调用该点时，收到 OK 响应:

```
@Test
public void givenRequestForStudents_whenSizeIsTwo_expectNumberOfElementsTwo() {
    given().params("page", "0", "size", "2").get(ENDPOINT)
      .then()
      .assertThat().body("numberOfElements", equalTo(2));
}
```

该测试断言，当请求页面大小为 2 时，返回的页面大小实际上是 2:

```
@Test
public void givenResourcesExist_whenFirstPageIsRetrieved_thenPageContainsResources() {
    given().params("page", "0", "size", "2").get(ENDPOINT)
      .then()
      .assertThat().body("first", equalTo(true));
} 
```

该测试断言，当第一次调用资源时，第一个页面名称值为真。

资源库中还有很多测试，所以一定要看看 [GitHub](https://web.archive.org/web/20220815032736/https://github.com/eugenp/tutorials/tree/master/spring-web-modules/spring-rest-angular) 项目。

## 8。结论

本文展示了如何使用`AngularJS`中的`UI-Grid`实现数据表网格，以及如何实现所需的服务器端分页。

这些例子和测试的实现可以在 [GitHub 项目](https://web.archive.org/web/20220815032736/https://github.com/eugenp/tutorials/tree/master/spring-web-modules/spring-rest-angular)中找到。这是一个 Maven 项目，因此应该很容易导入和运行。

要运行 Spring boot 项目，您可以简单地执行`mvn spring-boot:run`并在`http://localhost:8080/.`上本地访问它