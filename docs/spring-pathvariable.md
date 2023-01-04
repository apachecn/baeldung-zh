# Spring @PathVariable 批注

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-pathvariable>

## 1。概述

在这个快速教程中，我们将探索 Spring 的`@PathVariable`注释。

简单地说，**`@PathVariable`注释可以用来处理请求 URI 映射**中的模板变量，并将它们设置为方法参数。

我们来看看如何使用`@PathVariable`及其各种属性。

## 延伸阅读:

## [Spring @ request param vs @ path variable 注释](/web/20221113133230/https://www.baeldung.com/spring-requestparam-vs-pathvariable)

Understand the differences between Spring's @RequestParam and @PathVariable annotations.[Read more](/web/20221113133230/https://www.baeldung.com/spring-requestparam-vs-pathvariable) →

## [在 Spring 中验证请求参数和路径变量](/web/20221113133230/https://www.baeldung.com/spring-validate-requestparam-pathvariable)

Learn how to validate request parameters and path variables with Spring MVC[Read more](/web/20221113133230/https://www.baeldung.com/spring-validate-requestparam-pathvariable) →

## [Spring MVC @PathVariable 带点(。)被截断](/web/20221113133230/https://www.baeldung.com/spring-mvc-pathvariable-dot)

Learn how to handle path variables that contain a dot in Spring MVC request mappings.[Read more](/web/20221113133230/https://www.baeldung.com/spring-mvc-pathvariable-dot) →

## 2。一个简单的映射

`@PathVariable`注释的一个简单用例是用主键标识实体的端点:

```java
@GetMapping("/api/employees/{id}")
@ResponseBody
public String getEmployeesById(@PathVariable String id) {
    return "ID: " + id;
}
```

在这个例子中，我们使用`@PathVariable`注释来提取 URI 的模板化部分，由变量`{id}`表示。

一个简单的`GET request`到`/api/employees/{id}` 将使用提取的 id 值调用`getEmployeesById` :

```java
http://localhost:8080/api/employees/111 
---- 
ID: 111
```

现在让我们进一步研究这个注释，看看它的属性。

## 3。指定路径变量名

在前面的例子中，我们跳过了定义模板路径变量的名称，因为方法参数和路径变量的名称是相同的。

但是，如果路径变量名不同，我们可以在`@PathVariable`注释的参数中指定它:

```java
@GetMapping("/api/employeeswithvariable/{id}")
@ResponseBody
public String getEmployeesByIdWithVariableName(@PathVariable("id") String employeeId) {
    return "ID: " + employeeId;
}
```

```java
http://localhost:8080/api/employeeswithvariable/1 
---- 
ID: 1
```

为了清楚起见，我们也可以将路径变量名定义为`@PathVariable(value=”id”)`而不是 `PathVariable(“id”)`。

## 4.单个请求中的多个路径变量

根据用例的不同，**在控制器方法的请求 URI 中，我们可以有不止一个路径变量，它也有多个方法参数**:

```java
@GetMapping("/api/employees/{id}/{name}")
@ResponseBody
public String getEmployeesByIdAndName(@PathVariable String id, @PathVariable String name) {
    return "ID: " + id + ", name: " + name;
}
```

```java
http://localhost:8080/api/employees/1/bar 
---- 
ID: 1, name: bar
```

**我们也可以使用`java.util.Map<String, String>:`** 类型的方法参数来处理多个`@PathVariable`参数

```java
@GetMapping("/api/employeeswithmapvariable/{id}/{name}")
@ResponseBody
public String getEmployeesByIdAndNameWithMapVariable(@PathVariable Map<String, String> pathVarsMap) {
    String id = pathVarsMap.get("id");
    String name = pathVarsMap.get("name");
    if (id != null && name != null) {
        return "ID: " + id + ", name: " + name;
    } else {
        return "Missing Parameters";
    }
}
```

```java
http://localhost:8080/api/employees/1/bar 
---- 
ID: 1, name: bar
```

然而，当路径变量字符串包含一个点(.)性格。我们已经在这里详细讨论了这些极限情况。

## 5。可选路径变量

**在 Spring 中，带`@PathVariable`注释的方法参数默认是必需的:**

```java
@GetMapping(value = { "/api/employeeswithrequired", "/api/employeeswithrequired/{id}" })
@ResponseBody
public String getEmployeesByIdWithRequired(@PathVariable String id) {
    return "ID: " + id;
}
```

从表面上看，上面的控制器应该处理`/api/employeeswithrequired`和`/api/employeeswithrequired/1`请求路径。然而，由于默认情况下由`@PathVariables`注释的方法参数是强制的，它不处理发送到`/api/employeeswithrequired`路径的请求:

```java
http://localhost:8080/api/employeeswithrequired 
---- 
{"timestamp":"2020-07-08T02:20:07.349+00:00","status":404,"error":"Not Found","message":"","path":"/api/employeeswithrequired"}  http://localhost:8080/api/employeeswithrequired/1 
---- 
ID: 111
```

我们可以用两种不同的方式来处理这个问题。

### 5.1.不需要设置`@PathVariable`

**我们可以将`@PathVariable`的`required` 属性设置为`false` ，使其可选。**因此，修改我们前面的例子，我们现在可以处理有和没有路径变量的 URI 版本:

```java
@GetMapping(value = { "/api/employeeswithrequiredfalse", "/api/employeeswithrequiredfalse/{id}" })
@ResponseBody
public String getEmployeesByIdWithRequiredFalse(@PathVariable(required = false) String id) {
    if (id != null) {
        return "ID: " + id;
    } else {
        return "ID missing";
    }
}
```

```java
http://localhost:8080/api/employeeswithrequiredfalse 
---- 
ID missing
```

### 5.2.使用`java.util.Optional`

自从引入 Spring 4.1 之后，我们还可以使用[`java.util.Optional<T>`](/web/20221113133230/https://www.baeldung.com/java-optional)(Java 8+中可用)来处理一个非强制的路径变量:

```java
@GetMapping(value = { "/api/employeeswithoptional", "/api/employeeswithoptional/{id}" })
@ResponseBody
public String getEmployeesByIdWithOptional(@PathVariable Optional<String> id) {
    if (id.isPresent()) {
        return "ID: " + id.get();
    } else {
        return "ID missing";
    }
}
```

现在，如果我们不在请求中指定路径变量`id`，我们将得到默认响应:

```java
http://localhost:8080/api/employeeswithoptional 
----
ID missing 
```

### 5.3.使用`Map<String, String>`类型的方法参数

如前所述，我们可以使用类型为`java.util.Map`的单个方法参数来处理请求 URI 中的所有路径变量。**我们也可以用这个策略来处理可选路径变量的情况:**

```java
@GetMapping(value = { "/api/employeeswithmap/{id}", "/api/employeeswithmap" })
@ResponseBody
public String getEmployeesByIdWithMap(@PathVariable Map<String, String> pathVarsMap) {
    String id = pathVarsMap.get("id");
    if (id != null) {
        return "ID: " + id;
    } else {
        return "ID missing";
    }
}
```

## 6.`@PathVariable`的默认值

开箱即用，没有为用`@PathVariable`注释的方法参数定义默认值的规定。然而，我们可以使用上面讨论的相同策略来满足`@PathVariable,` 的缺省值情况，我们只需要检查路径变量上的`null`。

例如，使用`java.util.Optional<String, String>`，我们可以识别路径变量是否是`null`。如果是`null,`，那么我们可以用一个默认值来响应请求:

```java
@GetMapping(value = { "/api/defaultemployeeswithoptional", "/api/defaultemployeeswithoptional/{id}" })
@ResponseBody
public String getDefaultEmployeesByIdWithOptional(@PathVariable Optional<String> id) {
    if (id.isPresent()) {
        return "ID: " + id.get();
    } else {
        return "ID: Default Employee";
    }
}
```

## 7.结论

在本文中，我们讨论了如何使用 Spring 的`@PathVariable`注释。我们还确定了有效使用`@PathVariable`注释以适应不同用例的各种方法，比如可选参数和处理默认值。

本文中展示的代码示例也可以在 Github 的[上找到。](https://web.archive.org/web/20221113133230/https://github.com/eugenp/tutorials/tree/master/spring-web-modules/spring-mvc-java-2)