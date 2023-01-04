# Spring Web 注释

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-mvc-annotations>

[This article is part of a series:](javascript:void(0);)[• Spring Core Annotations](/web/20221128044356/https://www.baeldung.com/spring-core-annotations)
• Spring Web Annotations (current article)[• Spring Boot Annotations](/web/20221128044356/https://www.baeldung.com/spring-boot-annotations)
[• Spring Scheduling Annotations](/web/20221128044356/https://www.baeldung.com/spring-scheduling-annotations)
[• Spring Data Annotations](/web/20221128044356/https://www.baeldung.com/spring-data-annotations)
[• Spring Bean Annotations](/web/20221128044356/https://www.baeldung.com/spring-bean-annotations)

## 1.概观

在本教程中，我们将探索来自`org.springframework.web.bind.annotation`包的 Spring Web 注释。

## 2.`@RequestMapping`

简单地说， [`@RequestMapping`](/web/20221128044356/https://www.baeldung.com/spring-requestmapping) **标志着请求处理程序方法**内部的`@Controller`类；它可以通过以下方式进行配置:

*   `path,`或它的别名，`name,`和`value:`该方法被映射到哪个 URL
*   `method:`兼容的 HTTP 方法
*   `params:`根据 HTTP 参数的存在、不存在或值过滤请求
*   `headers:`根据 HTTP 头的存在、不存在或值过滤请求
*   `consumes:`该方法可以在 HTTP 请求体中使用哪些媒体类型
*   `produces:`该方法可以在 HTTP 响应体中产生哪些媒体类型

下面是一个简单的例子:

```java
@Controller
class VehicleController {

    @RequestMapping(value = "/vehicles/home", method = RequestMethod.GET)
    String home() {
        return "home";
    }
}
```

如果我们在类级别上应用这个注释，我们可以为一个`@Controller`类中的所有处理程序方法提供**默认设置。唯一的**例外是 URL，Spring 不会用方法级设置覆盖**，而是附加两个路径部分。**

例如，以下配置与上面的配置具有相同的效果:

```java
@Controller
@RequestMapping(value = "/vehicles", method = RequestMethod.GET)
class VehicleController {

    @RequestMapping("/home")
    String home() {
        return "home";
    }
}
```

此外，`@GetMapping`、`@PostMapping`、`@PutMapping`、`@DeleteMapping`和`@PatchMapping`是`@RequestMapping` 的不同变体，HTTP 方法已经分别设置为 GET、POST、PUT、DELETE 和 PATCH。

这些从 Spring 4.3 版本开始就可用了。

## 3.`@RequestBody`

让我们继续讨论[`@RequestBody`](/web/20221128044356/https://www.baeldung.com/spring-request-response-body)——它将 HTTP 请求的**主体映射到一个对象**:

```java
@PostMapping("/save")
void saveVehicle(@RequestBody Vehicle vehicle) {
    // ...
}
```

反序列化是自动的，取决于请求的内容类型。

## 4.`@PathVariable`

接下来，我们来说说`@PathVariable`。

这个注释表明一个**方法参数被绑定到一个 URI 模板变量**。我们可以用`@RequestMapping`注释指定 URI 模板，并用`@PathVariable`将一个方法参数绑定到模板的一个部分。

我们可以用`name`或它的别名`value`参数来实现这一点:

```java
@RequestMapping("/{id}")
Vehicle getVehicle(@PathVariable("id") long id) {
    // ...
}
```

如果模板中部件的名称与方法参数的名称匹配，我们不必在注释中指定它:

```java
@RequestMapping("/{id}")
Vehicle getVehicle(@PathVariable long id) {
    // ...
}
```

此外，我们可以通过将参数`required`设置为 false 来将路径变量标记为可选:

```java
@RequestMapping("/{id}")
Vehicle getVehicle(@PathVariable(required = false) long id) {
    // ...
}
```

## 5.`@RequestParam`

我们用`@RequestParam`代表**访问 HTTP 请求参数**:

```java
@RequestMapping
Vehicle getVehicleByParam(@RequestParam("id") long id) {
    // ...
}
```

它具有与`@PathVariable`注释相同的配置选项。

除了这些设置，当 Spring 在请求中找不到值或值为空时，我们可以使用`@RequestParam`指定一个注入值。为了实现这一点，我们必须设置`defaultValue`参数。

提供默认值会隐式地将`required`设置为`false:`

```java
@RequestMapping("/buy")
Car buyCar(@RequestParam(defaultValue = "5") int seatCount) {
    // ...
}
```

除了参数，还有**其他我们可以访问的 HTTP 请求部分:cookies 和头**。我们可以分别用注释 **`@CookieValue`和`@RequestHeader`** 来访问它们。

我们可以像配置`@RequestParam`一样配置它们。

## 6.响应处理注释

在接下来的章节中，我们将看到在 Spring MVC 中操纵 HTTP 响应的最常见的注释。

### 6.1.`@ResponseBody`

如果我们用`[@ResponseBody](/web/20221128044356/https://www.baeldung.com/spring-request-response-body),` **标记一个请求处理器方法，Spring 会将该方法的结果视为响应本身**:

```java
@ResponseBody
@RequestMapping("/hello")
String hello() {
    return "Hello World!";
}
```

如果我们用这个注释来注释一个`@Controller`类，所有的请求处理器方法都会使用它。

### 6.2.`@ExceptionHandler`

有了这个注释，我们可以声明一个定制的错误处理方法。当请求处理程序方法抛出任何指定的异常时，Spring 调用这个方法。

捕获的异常可以作为参数传递给方法:

```java
@ExceptionHandler(IllegalArgumentException.class)
void onIllegalArgumentException(IllegalArgumentException exception) {
    // ...
}
```

### 6.3.`@ResponseStatus`

如果我们用这个注释注释一个请求处理器方法，我们可以指定响应的**期望的 HTTP 状态。我们可以用参数`code`或者它的别名`value`来声明状态代码。**

同样，我们可以使用`reason`参数提供一个原因。

我们也可以将它与`@ExceptionHandler`一起使用:

```java
@ExceptionHandler(IllegalArgumentException.class)
@ResponseStatus(HttpStatus.BAD_REQUEST)
void onIllegalArgumentException(IllegalArgumentException exception) {
    // ...
}
```

有关 HTTP 响应状态的更多信息，请访问[本文](/web/20221128044356/https://www.baeldung.com/spring-mvc-controller-custom-http-status-code)。

## 7.其他 Web 注释

一些注释不直接管理 HTTP 请求或响应。在接下来的部分，我们将介绍最常见的。

### 7.1.`@Controller`

我们可以用`@Controller`定义一个 Spring MVC 控制器。更多信息，请访问[我们关于春豆注解的文章](/web/20221128044356/https://www.baeldung.com/spring-bean-annotations)。

### 7.2.`@RestController`

`@RestController` **结合了`@Controller`和`@ResponseBody`** 。

因此，以下声明是等效的:

```java
@Controller
@ResponseBody
class VehicleRestController {
    // ...
}
```

```java
@RestController
class VehicleRestController {
    // ...
}
```

### 7.3.`@ModelAttribute`

有了这个注释，我们可以通过提供模型键来访问已经在 MVC `@Controller,`的模型中的元素:

```java
@PostMapping("/assemble")
void assembleVehicle(@ModelAttribute("vehicle") Vehicle vehicleInModel) {
    // ...
}
```

与`@PathVariable`和`@RequestParam`一样，如果参数具有相同的名称，我们不必指定模型键:

```java
@PostMapping("/assemble")
void assembleVehicle(@ModelAttribute Vehicle vehicle) {
    // ...
}
```

除此之外，`@ModelAttribute`还有另外一个用途:如果我们用它来注释一个方法，Spring 会**自动将该方法的返回值添加到模型**中:

```java
@ModelAttribute("vehicle")
Vehicle getVehicle() {
    // ...
}
```

像以前一样，我们不必指定模型键，Spring 默认使用方法的名称:

```java
@ModelAttribute
Vehicle vehicle() {
    // ...
}
```

在 Spring 调用请求处理程序方法之前，它调用类中所有的`@ModelAttribute`注释方法。

更多关于`@ModelAttribute`的信息可以在[这篇文章](/web/20221128044356/https://www.baeldung.com/spring-mvc-and-the-modelattribute-annotation)中找到。

### 7.4.`@CrossOrigin`

`@CrossOrigin` **为带注释的请求处理程序方法启用跨域通信**:

```java
@CrossOrigin
@RequestMapping("/hello")
String hello() {
    return "Hello World!";
}
```

如果我们用它来标记一个类，它将应用于其中的所有请求处理程序方法。

我们可以用这个注释的参数微调 CORS 行为。

更多详情请访问[本文](/web/20221128044356/https://www.baeldung.com/spring-cors)。

## 8.结论

在本文中，我们看到了如何用 Spring MVC 处理 HTTP 请求和响应。

像往常一样，这些例子可以在 GitHub 的[上找到。](https://web.archive.org/web/20221128044356/https://github.com/eugenp/tutorials/tree/master/spring-boot-modules/spring-boot-annotations)

Next **»**[Spring Boot Annotations](/web/20221128044356/https://www.baeldung.com/spring-boot-annotations)**«** Previous[Spring Core Annotations](/web/20221128044356/https://www.baeldung.com/spring-core-annotations)