# spring Security –@ pre filter 和@PostFilter

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-security-prefilter-postfilter>

## 1。概述

在本文中，我们将学习如何使用`[@PreFilter](https://web.archive.org/web/20220812064151/https://docs.spring.io/spring-security/site/docs/current/api/org/springframework/security/access/prepost/PreFilter.html)`和`[@PostFilter](https://web.archive.org/web/20220812064151/https://docs.spring.io/spring-security/site/docs/current/api/org/springframework/security/access/prepost/PostFilter.html)`注释来保护 Spring 应用程序中的操作。

当与认证的主体信息一起使用时，`@PreFilter`和`@PostFilter` 允许我们使用 Spring 表达式语言定义细粒度的安全规则。

## 2。介绍`@PreFilter`和`@PostFilter`和

简单地说， `@PreFilter`和`@PostFilter`注释是**,用于根据我们定义的自定义安全规则过滤对象列表**。

`@PostFilter`定义了过滤方法返回列表的规则，通过**将该规则应用到列表**中的每个元素。如果评估值为真，该项目将保留在列表中。否则，该项目将被删除。

`@PreFilter` 以非常相似的方式工作，但是，过滤应用于作为输入参数传递给带注释的方法的列表。

这两种注释都可以用在方法或类型(类和接口)上。在本文中，我们将只在方法上使用它们。

默认情况下，这些注释是不活动的——我们需要在我们的安全配置中使用`@EnableGlobalMethodSecurity` 注释和`prePostEnabled = true`来启用它们:

```java
@Configuration
@EnableWebSecurity
@EnableGlobalMethodSecurity(prePostEnabled = true)
public class WebSecurityConfig extends WebSecurityConfigurerAdapter {
    // ...
}
```

## 3。编写安全规则

为了在这两个注释中编写安全规则，我们将使用 Spring-EL 表达式；我们还可以使用内置对象`filterObject` 来获取对被测试的特定列表元素的引用。

Spring Security 提供了许多其他内置对象来创建非常具体和精确的规则。

**例如**，我们可以用`@PreFilter` 来检查一个`Task`对象的`assignee`属性是否等于当前认证用户的`name` :

```java
@PostFilter("filterObject.assignee == authentication.name")
List<Task> findAll() {
    ...
}
```

我们在这里使用了`@PostFilter`注释，因为我们希望该方法首先执行并获取所有任务，它们通过我们的过滤规则传递列表中的每个任务。

因此，如果认证用户是`michael`，那么由`findAll` 方法返回的最终任务列表将只包含分配给`michael`的任务，即使数据库有分配给`jim`和`pam`的任务。

现在让我们把规则变得更有趣一点。假设如果用户是经理，他们可以查看所有任务，而不管任务分配给谁:

```java
@PostFilter("hasRole('MANAGER') or filterObject.assignee == authentication.name")
List<Task> findAll() {
    // ...
}
```

我们已经使用了内置方法`hasRole`来检查经过身份验证的用户是否具有管理者的角色。如果`hasRole`返回 true，任务将被保留在最终列表中。因此，如果用户是经理，规则将为列表中的每个项目返回 true。因此，最终列表将包含所有项目。

现在让我们使用`@PreFilter`过滤作为参数传递给`save`方法的列表:

```java
@PreFilter("hasRole('MANAGER') or filterObject.assignee == authentication.name")
Iterable<Task> save(Iterable<Task> entities) {
    // ...
}
```

安全规则与我们在`@PostFilter`示例中使用的规则相同。这里的主要区别是，在方法执行之前，列表项将被过滤，从而允许我们从列表中删除一些项，防止它们被保存在数据库中。

所以不是经理的`jim`可能试图保存一个任务列表，其中一些任务被分配给`pam`。然而，只有那些分配给`jim`的任务将被包括在内，其他的将被忽略。

## 4。大型列表的性能

很酷，也很容易使用，但是在处理非常大的列表时效率会很低，因为获取操作会检索所有的数据，然后应用过滤器。

例如，假设我们的数据库中有数千个任务，我们想要检索当前分配给`pam`的五个任务。如果我们使用`@PreFilter` `,` ，数据库操作将首先获取所有任务，并遍历所有任务，过滤掉没有分配给`pam`的任务。

## 5。结论

这篇简短的文章解释了如何使用 Spring Security 的`@PreFilter`和`@PostFilter`注释创建一个简单但安全的应用程序。

查看 Github 库中的完整代码示例[。](https://web.archive.org/web/20220812064151/https://github.com/eugenp/tutorials/tree/master/spring-security-modules/spring-security-core)