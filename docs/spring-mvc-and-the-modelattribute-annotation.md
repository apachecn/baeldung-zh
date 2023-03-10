# Spring MVC 和@ModelAttribute 注释

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-mvc-and-the-modelattribute-annotation>

## 1。概述

最重要的 [Spring MVC](https://web.archive.org/web/20220625222500/https://docs.spring.io/spring/docs/current/spring-framework-reference/html/mvc.html) 注释之一是 [@ModelAttribute](https://web.archive.org/web/20220625222500/https://docs.spring.io/spring/docs/3.1.x/javadoc-api/org/springframework/web/bind/annotation/ModelAttribute.html) 注释。

`@ModelAttribute`是一个注释，它将方法参数或方法返回值绑定到一个命名的模型属性，然后将其公开给 web 视图。

在本教程中，我们将通过一个常见的概念，一个公司员工提交的表单，来演示这个注释的可用性和功能性。

## 延伸阅读:

## [Spring MVC 中的模型、模型图和模型视图](/web/20220625222500/https://www.baeldung.com/spring-mvc-model-model-map-model-view)

Learn about the interfaces `Model`**,** `ModelMap` and `ModelAndView` provided by Spring MVC.[Read more](/web/20220625222500/https://www.baeldung.com/spring-mvc-model-model-map-model-view) →

## [Spring @RequestParam 注释](/web/20220625222500/https://www.baeldung.com/spring-request-param)

A detailed guide to Spring's @RequestParam annotation[Read more](/web/20220625222500/https://www.baeldung.com/spring-request-param) →

## 2。`@ModelAttribute`深度

正如介绍性段落所揭示的，我们可以使用`@ModelAttribute`作为方法参数或者在方法级别使用。

### 2.1。在方法级别

当我们在方法级别使用注释时，它表明方法的目的是添加一个或多个模型属性。这样的方法支持与 [@RequestMapping](https://web.archive.org/web/20220625222500/https://docs.spring.io/spring/docs/current/javadoc-api/org/springframework/web/bind/annotation/RequestMapping.html) 方法相同的参数类型，但是它们不能直接映射到请求。

让我们看一个简单的例子来理解这是如何工作的:

```java
@ModelAttribute
public void addAttributes(Model model) {
    model.addAttribute("msg", "Welcome to the Netherlands!");
} 
```

在上面的例子中，我们看到一个方法将名为`msg`的属性添加到控制器类中定义的所有`model`中。

当然，我们将在本文的后面看到这一点。

一般来说，Spring MVC 在调用任何请求处理程序方法之前，总是先调用那个方法。基本上， **`@ModelAttribute`方法是在用`@RequestMapping`标注的控制器方法被调用之前被调用的。**这是因为模型对象必须在控制器方法内的任何处理开始之前创建。

同样重要的是，我们将各自的类标注为 [@ControllerAdvice](https://web.archive.org/web/20220625222500/https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/web/bind/annotation/ControllerAdvice.html) 。因此，我们可以在`Model`中添加将被标识为全局的值。这实际上意味着对于每个请求，响应中的每个方法都有一个默认值。

### 2.2。作为方法参数

当我们使用注释作为方法参数时，它表示从模型中检索参数。当注释不存在时，它应该首先被实例化，然后被添加到模型中。一旦出现在模型中，arguments 字段应该由具有匹配名称的所有请求参数填充。

在下面的代码片段中，我们将用提交给`addEmployee`端点的表单中的数据填充`employee`模型属性。Spring MVC 在调用 submit 方法之前在幕后完成这项工作:

```java
@RequestMapping(value = "/addEmployee", method = RequestMethod.POST)
public String submit(@ModelAttribute("employee") Employee employee) {
    // Code that uses the employee object

    return "employeeView";
}
```

在本文的后面，我们将看到如何使用`employee`对象填充`employeeView`模板的完整示例。

它用 bean 绑定表单数据。**用`@RequestMapping`标注的控制器可以有用`@ModelAttribute`标注的自定义类参数。**

在 Spring MVC 中，我们称之为数据绑定，这是一种通用的机制，使我们不必单独解析每个表单字段。

## 3。表单示例

在本节中，我们将查看概述部分中概述的示例，这是一个非常基本的表单，提示用户(特别是公司员工)输入一些个人信息(特别是`name`和`id).` )。在提交完成后，如果没有任何错误，用户希望看到之前提交的数据显示在另一个屏幕上。

### 3.1。视图

让我们首先创建一个带有 id 和 name 字段的简单表单:

```java
<form:form method="POST" action="/spring-mvc-basics/addEmployee" 
  modelAttribute="employee">
    <form:label path="name">Name</form:label>
    <form:input path="name" />

    <form:label path="id">Id</form:label>
    <form:input path="id" />

    <input type="submit" value="Submit" />
</form:form> 
```

### 3.2。控制器

这里是控制器类，我们将在其中实现上述视图的逻辑:

```java
@Controller
@ControllerAdvice
public class EmployeeController {

    private Map<Long, Employee> employeeMap = new HashMap<>();

    @RequestMapping(value = "/addEmployee", method = RequestMethod.POST)
    public String submit(
      @ModelAttribute("employee") Employee employee,
      BindingResult result, ModelMap model) {
        if (result.hasErrors()) {
            return "error";
        }
        model.addAttribute("name", employee.getName());
        model.addAttribute("id", employee.getId());

        employeeMap.put(employee.getId(), employee);

        return "employeeView";
    }

    @ModelAttribute
    public void addAttributes(Model model) {
        model.addAttribute("msg", "Welcome to the Netherlands!");
    }
}
```

在`submit()`方法中，我们有一个`Employee`对象绑定到我们的`View`。我们可以简单地将表单字段映射到对象模型。在这个方法中，我们从表单中获取值，并将它们设置为`[ModelMap](https://web.archive.org/web/20220625222500/https://docs.spring.io/spring/docs/current/javadoc-api/org/springframework/ui/ModelMap.html)`。

**最后，我们返回`employeeView`，这意味着我们调用各自的 JSP 文件作为`View`代表。**

此外，还有一个`addAttributes()`方法。它的目的是在`Model`中添加将被全局识别的值。也就是说，对每个控制器方法的每个请求都将返回一个默认值作为响应。我们还必须将特定的类标注为`@ControllerAdvice.`

### 3.3。型号

如前所述，`Model`对象非常简单，包含了所有“前端”属性所需的内容。现在让我们来看一个例子:

```java
@XmlRootElement
public class Employee {

    private long id;
    private String name;

    public Employee(long id, String name) {
        this.id = id;
        this.name = name;
    }

    // standard getters and setters removed
}
```

### 3.4。总结

`@ControllerAdvice`协助控制器，特别是适用于所有`@RequestMapping`方法的`@ModelAttribute`方法。当然，我们的`addAttributes()`方法将是最先运行的，在其余的`@RequestMapping`方法之前。

记住这一点，在运行了`submit()`和`addAttributes()`之后，我们可以在从`Controller`类返回的`View`中引用它们，方法是在一个美元化的花括号二人组中提到它们的名字，比如`${name}`。

### 3.5。结果视图

现在让我们打印从表单中收到的内容:

```java
<h3>${msg}</h3>
Name : ${name}
ID : ${id} 
```

## 4。结论

在本文中，我们研究了`@ModelAttribute` 注释在方法参数和方法级用例`.`中的使用

本文的实现可以在 [github](https://web.archive.org/web/20220625222500/https://github.com/eugenp/tutorials/tree/master/spring-web-modules/spring-mvc-basics-5) 项目中找到。