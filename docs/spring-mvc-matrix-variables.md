# Spring MVC 矩阵变量快速指南

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-mvc-matrix-variables>

## 1。概述

URI 规范 [RFC 3986](https://web.archive.org/web/20220827073606/https://tools.ietf.org/html/rfc3986#section-3.3) 将 URI 路径参数定义为名称-值对。矩阵变量是 Spring 创造的术语，也是传递和解析 URI 路径参数的替代实现。

矩阵变量支持在 Spring MVC 3.2 中变得可用，这意味着用大量参数来**简化请求。**

在本文中，我们将展示如何简化复杂的 GET 请求，这些请求在 URI 的不同路径段中使用变量或可选路径参数。

## 2。配置

要启用 Spring MVC 矩阵变量，让我们从配置开始:

```java
@Configuration
public class WebConfig implements WebMvcConfigurer {

    @Override
    public void configurePathMatch(PathMatchConfigurer configurer) {
        UrlPathHelper urlPathHelper = new UrlPathHelper();
        urlPathHelper.setRemoveSemicolonContent(false);
        configurer.setUrlPathHelper(urlPathHelper);
    }
}
```

否则，默认情况下它们是禁用的。

## 3。如何使用矩阵变量

这些变量可以出现在路径的任何部分，字符等号(" = ")用于给出值，分号('；'))来限定每个矩阵变量。在同一路径上，我们还可以重复相同的变量名，或者使用逗号('，')字符分隔不同的值。

我们的例子有一个控制器，它提供关于雇员的信息。每个员工都有一个工作区，我们可以根据该属性进行搜索。以下请求可用于搜索:

```java
http://localhost:8080/spring-mvc-java-2/employeeArea/workingArea=rh,informatics,admin
```

或者像这样:

```java
http://localhost:8080/spring-mvc-java-2
  /employeeArea/workingArea=rh;workingArea=informatics;workingArea=admin
```

当我们想在 Spring MVC 中引用这些变量时，我们应该使用注释`@MatrixVariable`。

在我们的例子中，我们将使用`Employee` 类:

```java
public class Employee {

    private long id;
    private String name;
    private String contactNumber;

    // standard setters and getters 
}
```

还有`Company`类:

```java
public class Company {

    private long id;
    private String name;

    // standard setters and getters
}
```

这两个类将绑定请求参数。

## 4。定义矩阵变量属性

我们可以为变量指定必需的或默认的属性。在下面的例子中，`contactNumber`是必需的，所以它必须包含在我们的路径中，就像这样:

```java
http://localhost:8080/spring-mvc-java-2/employeesContacts/contactNumber=223334411
```

该请求将通过以下方法处理:

```java
@RequestMapping(value = "/employeesContacts/{contactNumber}", 
  method = RequestMethod.GET)
@ResponseBody
public ResponseEntity<List<Employee>> getEmployeeByContactNumber(
  @MatrixVariable(required = true) String contactNumber) {
    List<Employee> employeesList = new ArrayList<Employee>();
    ...
    return new ResponseEntity<List<Employee>>(employeesList, HttpStatus.OK);
}
```

因此，我们将获得所有联系号码为`223334411`的员工。

## 5。补充参数

**矩阵变量可以补充路径变量。**

例如，我们在搜索一个员工的姓名，但是我们也可以包括他/她的联系号码的起始号码。

该搜索的请求应该是这样的:

```java
http://localhost:8080/spring-mvc-java-2/employees/John;beginContactNumber=22001
```

该请求将通过以下方法处理:

```java
@RequestMapping(value = "/employees/{name}", method = RequestMethod.GET)
@ResponseBody
public ResponseEntity<List<Employee>> getEmployeeByNameAndBeginContactNumber(
  @PathVariable String name, @MatrixVariable String beginContactNumber) {
    List<Employee> employeesList = new ArrayList<Employee>();
    ...
    return new ResponseEntity<>(employeesList, HttpStatus.OK);
}
```

因此，我们将获得联系号码为`22001`或姓名为`John`的所有员工。

## 6。绑定所有矩阵变量

如果出于某种原因，我们想要获得路径上所有可用的变量，我们可以将它们绑定到一个`Map`:

```java
http://localhost:8080/spring-mvc-java-2/employeeData/id=1;name=John;contactNumber=2200112334
```

该请求将通过以下方法处理:

```java
@GetMapping("employeeData/{employee}")
@ResponseBody
public ResponseEntity<Map<String, String>> getEmployeeData(
  @MatrixVariable Map<String, String> matrixVars) {
    return new ResponseEntity<>(matrixVars, HttpStatus.OK);
}
```

当然，我们可以限制绑定到路径特定部分的矩阵变量。例如，如果我们有这样一个请求:

```java
http://localhost:8080/spring-mvc-java-2/
  companyEmployee/id=2;name=Xpto/employeeData/id=1;name=John;
  contactNumber=2200112334
```

而我们只想得到属于`employeeData`的所有变量；那么我们应该使用以下内容作为输入参数:

```java
@RequestMapping(
 value = "/companyEmployee/{company}/employeeData/{employee}",
 method = RequestMethod.GET)
@ResponseBody
public ResponseEntity<Map<String, String>> getEmployeeDataFromCompany(
  @MatrixVariable(pathVar = "employee") Map<String, String> matrixVars) {
  ...
}
```

## 7 .**。部分绑定**

除了简单之外，灵活性是另一个好处，矩阵变量可以以各种不同的方式使用。例如，我们可以从每个路径段获得每个变量。考虑以下请求:

```java
http://localhost:8080/spring-mvc-java-2/
  companyData/id=2;name=Xpto/employeeData/id=1;name=John;
  contactNumber=2200112334
```

如果我们只想知道`companyData`段的矩阵变量`name`，那么，我们应该使用以下作为输入参数:

```java
@MatrixVariable(value="name", pathVar="company") String name 
```

## 8.防火墙设置

如果应用程序使用 Spring 安全，那么默认使用 `StrictHttpFirewall` 。这会阻止看似恶意的请求，包括带有分号分隔符的矩阵变量。

我们可以 [在应用配置中定制](/web/20220827073606/https://www.baeldung.com/spring-security-request-rejected-exception#2-stricthttpfirewall) 这种实现，允许这样的变量，同时拒绝其他可能的恶意请求。

然而，通过这种方式，我们可能会使应用程序遭受攻击。因此，我们应该在仔细分析应用程序和安全需求之后才实现它。

## 9。结论

本文举例说明了矩阵变量的各种用法。

了解这个新工具如何处理过于复杂的请求或帮助我们添加更多参数来界定我们的搜索是非常重要的。

所有这些例子和代码片段的实现都可以在一个 GitHub 项目中找到——这是一个基于 Maven 的项目，所以它应该很容易导入和运行。