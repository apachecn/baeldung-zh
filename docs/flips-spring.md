# 春季空翻指南

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/flips-spring>

## 1。概述

在本教程中，我们将看看 [Flips，](https://web.archive.org/web/20220703153300/https://github.com/Feature-Flip/flips)一个以强大注释的形式为 Spring Core、Spring MVC 和 Spring Boot 应用程序实现特性标志的库。

特性标志(或切换)是一种快速安全地交付新特性的模式。这些切换允许我们在不改变或部署新代码的情况下修改应用行为**。** Martin Fowler 的博客上有一篇关于功能标志[的非常翔实的文章](https://web.archive.org/web/20220703153300/https://martinfowler.com/articles/feature-toggles.html)。

## 2。Maven 依赖关系

在我们开始之前，我们需要将翻转库添加到我们的`pom.xml:`

```java
<dependency>
    <groupId>com.github.feature-flip</groupId>
    <artifactId>flips-core</artifactId>
    <version>1.0.1</version>
</dependency>
```

Maven Central 有[最新版本的库](https://web.archive.org/web/20220703153300/https://search.maven.org/classic/#search%7Cga%7C1%7Cg%3A%22com.github.feature-flip%22)，Github 项目在这里是。

当然，我们还需要包括一个弹簧:

```java
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
    <version>1.5.10.RELEASE</version>
</dependency>
```

由于 Flips 还不兼容 Spring 版本 5.x，我们将在 4.x 分支中使用 Spring Boot 的[最新版本。](https://web.archive.org/web/20220703153300/https://search.maven.org/classic/#artifactdetails%7Corg.springframework.boot%7Cspring-boot-starter-web%7C1.5.10.RELEASE%7Cjar)

## 3。简单的空翻休息服务

让我们把一个简单的 Spring Boot 项目放在一起，添加和切换新的功能和标志。

我们的 REST 应用程序将提供对`Foo` 资源的访问:

```java
public class Foo {
    private String name;
    private int id;
} 
```

我们将简单地创建一个`Service`来维护一个`Foos`列表:

```java
@Service
public class FlipService {

    private List<Foo> foos;

    public List<Foo> getAllFoos() {
        return foos;
    }

    public Foo getNewFoo() {
        return new Foo("New Foo!", 99);
    }
} 
```

我们将继续引用其他服务方法，但是这个片段应该足以说明`FlipService`在系统中做了什么。

当然，我们需要创建一个控制器:

```java
@RestController
public class FlipController {

    private FlipService flipService;

    // constructors

    @GetMapping("/foos")
    public List<Foo> getAllFoos() {
        return flipService.getAllFoos();
    }
} 
```

## 4。基于配置的控制特性

翻转的最基本用途是根据配置启用或禁用某项功能。翻转对此有几个注释。

### 4.1。环境属性

让我们想象我们给`FlipService`添加了一个新的功能；通过 id 检索`Foos`。

让我们将新请求添加到控制器中:

```java
@GetMapping("/foos/{id}")
@FlipOnEnvironmentProperty(
  property = "feature.foo.by.id", 
  expectedValue = "Y")
public Foo getFooById(@PathVariable int id) {
    return flipService.getFooById(id)
      .orElse(new Foo("Not Found", -1));
} 
```

`@FlipOnEnvironmentProperty`控制这个 API 是否可用。

简单来说，当`feature.foo.by.id`为`Y`时，我们可以通过 Id 进行请求。如果没有定义(或者根本没有定义)，翻转将禁用 API 方法。

**如果一个特性没有启用，那么 Flips 将抛出`FeatureNotEnabledException`，Spring 将向 REST 客户端返回“未实现”。**

当我们调用属性设置为`N`的 API 时，我们看到的是:

```java
Status = 501
Headers = {Content-Type=[application/json;charset=UTF-8]}
Content type = application/json;charset=UTF-8
Body = {
    "errorMessage": "Feature not enabled, identified by method 
      public com.baeldung.flips.model.Foo
      com.baeldung.flips.controller.FlipController.getFooById(int)",
    "className":"com.baeldung.flips.controller.FlipController",
    "featureName":"getFooById"
}
```

正如预期的那样，Spring 捕获了`FeatureNotEnabledException`并将状态 501 返回给客户机。

### 4.2。活动配置文件

Spring 早就给了我们将 beans 映射到不同的[配置文件](/web/20220703153300/https://www.baeldung.com/spring-profiles)的能力，比如`dev`、`test`或`prod`。扩展这一功能以将特征标志映射到活动配置文件具有直观的意义。

让我们看看如何根据活动的[弹簧轮廓](/web/20220703153300/https://www.baeldung.com/spring-profiles)启用或禁用功能:

```java
@RequestMapping(value = "/foos", method = RequestMethod.GET)
@FlipOnProfiles(activeProfiles = "dev")
public List getAllFoos() {
    return flipService.getAllFoos();
} 
```

`@FlipOnProfiles`注释接受一个概要文件名列表。如果激活的概要文件在列表中，那么 API 是可访问的。

### 4.3。弹簧表达式

Spring 的表达式语言(SpEL) 是操纵运行时环境的强大机制。翻转也为我们提供了一种切换功能的方式。

`@FlipOnSpringExpression`切换基于返回布尔值的 SpEL 表达式的方法。

让我们用一个简单的表达式来控制一个新特性:

```java
@FlipOnSpringExpression(expression = "(2 + 2) == 4")
@GetMapping("/foo/new")
public Foo getNewFoo() {
    return flipService.getNewFoo();
} 
```

### 4.4。禁用

要完全禁用某项功能，请使用`@FlipOff`:

```java
@GetMapping("/foo/first")
@FlipOff
public Foo getFirstFoo() {
    return flipService.getLastFoo();
} 
```

在本例中，`getFirstFoo()`完全不可访问。

正如我们将在下面看到的，我们可以组合翻转注释，使得使用`@FlipOff` 根据环境或其他标准禁用某个特性成为可能。

## 5。带日期/时间的控制功能

翻转可以基于日期/时间或星期几来切换功能。将新特性的可用性与某一天或日期联系起来有明显的优势。

### 5.1。日期和时间

`@FlipOnDateTime`接受格式化为[T2【ISO 8601】格式的属性名。](https://web.archive.org/web/20220703153300/https://www.iso.org/iso-8601-date-and-time-format.html)

因此，让我们设置一个属性，指示将于 3 月 1 日激活的新功能:

```java
first.active.after=2018-03-01T00:00:00Z
```

然后我们将编写一个 API 来检索第一个 Foo:

```java
@GetMapping("/foo/first")
@FlipOnDateTime(cutoffDateTimeProperty = "first.active.after")
public Foo getFirstFoo() {
    return flipService.getLastFoo();
} 
```

翻转将检查命名属性。如果该属性存在，并且指定的日期/时间已经过去，则启用该功能。

### 5.2。星期几

该库提供了`@FlipOnDaysOfWeek`，对于 A/B 测试等操作很有用:

```java
@GetMapping("/foo/{id}")
@FlipOnDaysOfWeek(daysOfWeek={DayOfWeek.MONDAY, DayOfWeek.WEDNESDAY})
public Foo getFooByNewId(@PathVariable int id) {
    return flipService.getFooById(id).orElse(new Foo("Not Found", -1));
} 
```

`getFooByNewId()`仅在周一和周三可用。

## 6。替换一个 Bean

打开和关闭方法是有用的，但是我们可能希望通过新的对象引入新的行为。指示调用新 bean 中的方法。

翻转注释可以在任何弹簧`@Component.`上工作，到目前为止，我们只修改了我们的`@RestController`，让我们试着修改我们的`Service.`

我们将创建一个行为与`FlipService`不同的新服务:

```java
@Service
public class NewFlipService {
    public Foo getNewFoo() {
        return new Foo("Shiny New Foo!", 100);
    }
} 
```

我们将用新版本替换旧服务的`getNewFoo()`:

```java
@FlipBean(with = NewFlipService.class)
public Foo getNewFoo() {
    return new Foo("New Foo!", 99);
} 
```

翻转将调用`getNewThing()`到`NewFlipService. @FlipBean`是另一个开关，当与其他开关结合使用时最有用。现在让我们来看看。

## 7。组合开关

我们通过指定多个开关来组合开关。Flips 使用隐含的“与”逻辑按顺序计算这些值。因此，它们都必须为真才能打开该功能。

让我们结合前面的两个例子:

```java
@FlipBean(
  with = NewFlipService.class)
@FlipOnEnvironmentProperty(
  property = "feature.foo.by.id", 
  expectedValue = "Y")
public Foo getNewFoo() {
    return new Foo("New Foo!", 99);
} 
```

我们已经使用了新的可配置服务。

## 8。结论

在这个简短的指南中，我们创建了一个简单的 Spring Boot 服务，并使用翻转注释来打开和关闭 API。我们看到了如何使用配置信息和日期/时间来切换特性，以及如何通过在运行时交换 beans 来切换特性。

代码样本一如既往地可以在 GitHub 上找到[。](https://web.archive.org/web/20220703153300/https://github.com/eugenp/tutorials/tree/master/spring-4)