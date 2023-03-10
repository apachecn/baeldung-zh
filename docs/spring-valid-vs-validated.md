# Spring 中@Valid 和@Validated 注释的区别

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-valid-vs-validated>

## 1.概观

在这个快速教程中，我们将关注 Spring 中的 [`@Valid`](https://web.archive.org/web/20220827110142/https://docs.oracle.com/javaee/7/api/javax/validation/Valid.html) 和`[@Validated](https://web.archive.org/web/20220827110142/https://docs.spring.io/spring/docs/current/javadoc-api/org/springframework/validation/annotation/Validated.html)` 注释之间的区别。

在大多数应用程序中，验证用户输入是一个常见的功能。在 Java 生态系统中，我们专门使用 [Java 标准 Bean 验证 API](/web/20220827110142/https://www.baeldung.com/javax-validation) 来支持这一点，它从 4.0 版本开始就与 Spring 很好地集成在一起。**`@Valid` 和`@Validated`注释源自这个标准 Bean API** 。

在接下来的部分中，我们将更详细地探讨它们。

## 2.`@Valid`和`@Validated`注释

在 Spring 中，我们使用 JSR-303 的 **`@Valid` 注释进行方法级验证**。**我们也用它来标记一个成员属性进行验证**。但是，该注释不支持组验证。

组有助于限制验证过程中应用的约束。一个特殊的用例是 UI 向导。在第一步中，我们可能有某个字段子组。在随后的步骤中，可能有另一个组属于同一个 bean。所以我们需要在每一步对这些有限的字段施加约束，但是`@Valid` 不支持这个。

在这种情况下，**对于组级别，我们必须使用 Spring 的`@Validated,`** ，它是 JSR-303 的`@Valid`的变体。这在方法级别使用。为了标记成员属性，我们继续使用`@Valid`注释。

现在让我们通过一个例子深入了解一下这些注释的用法。

## 3.例子

让我们考虑一个使用 Spring Boot 开发的简单用户注册表单。首先，我们只有`name`和`password`属性:

```java
public class UserAccount {

    @NotNull
    @Size(min = 4, max = 15)
    private String password;

    @NotBlank
    private String name;

    // standard constructors / setters / getters / toString

} 
```

接下来，我们来看看控制器。这里我们将使用带有`@Valid`注释的`saveBasicInfo`方法来验证用户输入:

```java
@RequestMapping(value = "/saveBasicInfo", method = RequestMethod.POST)
public String saveBasicInfo(
  @Valid @ModelAttribute("useraccount") UserAccount useraccount, 
  BindingResult result, 
  ModelMap model) {
    if (result.hasErrors()) {
        return "error";
    }
    return "success";
}
```

现在让我们来测试这个方法:

```java
@Test
public void givenSaveBasicInfo_whenCorrectInput_thenSuccess() throws Exception {
    this.mockMvc.perform(MockMvcRequestBuilders.post("/saveBasicInfo")
      .accept(MediaType.TEXT_HTML)
      .param("name", "test123")
      .param("password", "pass"))
      .andExpect(view().name("success"))
      .andExpect(status().isOk())
      .andDo(print());
}
```

在确认测试成功运行后，我们将扩展功能。下一个合乎逻辑的步骤是将它转换成一个多步注册表单，就像大多数向导一样。第一步的`name`和`password`保持不变。第二步，我们将获取额外的信息，如`age`和`phone`。然后，我们将使用这些附加字段更新我们的域对象:

```java
public class UserAccount {

    @NotNull
    @Size(min = 4, max = 15)
    private String password;

    @NotBlank
    private String name;

    @Min(value = 18, message = "Age should not be less than 18")
    private int age;

    @NotBlank
    private String phone;

    // standard constructors / setters / getters / toString   

} 
```

然而，这一次我们会注意到前面的测试失败了。这是因为我们没有传入`age`和`phone` 字段，它们仍然不在 UI `.`上的图片中。为了支持这种行为，我们将需要组验证和`@Validated`注释。

为此，我们需要将字段分组，创建两个不同的组。首先，我们需要创建两个标记接口，每个组或每个步骤一个单独的接口。我们可以参考我们关于[组验证](/web/20220827110142/https://www.baeldung.com/javax-validation-groups)的文章，了解具体的实现。在这里，让我们集中讨论注释的不同之处。

我们将为第一步提供 `BasicInfo` 接口，为第二步提供`AdvanceInfo`接口。此外，我们将更新我们的`UserAccount`类来使用这些标记接口:

```java
public class UserAccount {

    @NotNull(groups = BasicInfo.class)
    @Size(min = 4, max = 15, groups = BasicInfo.class)
    private String password;

    @NotBlank(groups = BasicInfo.class)
    private String name;

    @Min(value = 18, message = "Age should not be less than 18", groups = AdvanceInfo.class)
    private int age;

    @NotBlank(groups = AdvanceInfo.class)
    private String phone;

    // standard constructors / setters / getters / toString   

} 
```

此外，我们将更新我们的控制器，使用`@Validated`注释代替`@Valid`:

```java
@RequestMapping(value = "/saveBasicInfoStep1", method = RequestMethod.POST)
public String saveBasicInfoStep1(
  @Validated(BasicInfo.class) 
  @ModelAttribute("useraccount") UserAccount useraccount, 
  BindingResult result, ModelMap model) {
    if (result.hasErrors()) {
        return "error";
    }
    return "success";
}
```

由于这次更新，我们的测试现在可以成功运行了。我们还将测试这个新方法:

```java
@Test
public void givenSaveBasicInfoStep1_whenCorrectInput_thenSuccess() throws Exception {
    this.mockMvc.perform(MockMvcRequestBuilders.post("/saveBasicInfoStep1")
      .accept(MediaType.TEXT_HTML)
      .param("name", "test123")
      .param("password", "pass"))
      .andExpect(view().name("success"))
      .andExpect(status().isOk())
      .andDo(print());
}
```

这也运行成功。因此，我们可以看到**`@Validated`的使用对于组验证是多么重要。**

接下来，让我们看看`@Valid`对于触发嵌套属性的验证是多么重要。

## 4.使用`@Valid`注释标记嵌套对象

**`@Valid` 注释用于标记嵌套的属性，特别是**。这将触发嵌套对象的验证。例如，在我们当前的场景中，我们可以创建一个`UserAddress `对象:

```java
public class UserAddress {

    @NotBlank
    private String countryCode;

    // standard constructors / setters / getters / toString
}
```

为了确保这个嵌套对象的有效性，我们将用`@Valid`注释来修饰这个属性:

```java
public class UserAccount {

    //...

    @Valid
    @NotNull(groups = AdvanceInfo.class)
    private UserAddress useraddress;

    // standard constructors / setters / getters / toString 
}
```

## 5.利弊

让我们看看在 Spring 中使用`@Valid`和`@Validated`注释的利弊。

**`@Valid`注释确保整个对象的有效性。**重要的是，它执行整个对象图的验证。**然而，这给只需要部分验证的场景带来了问题。**

**另一方面，我们可以使用`@Validated`进行分组验证，包括上面的部分验证。**然而，在这种情况下，被验证的实体必须知道它们被使用的所有组或用例的验证规则。这里我们混合了关注点，这可能导致反模式。

## 6.结论

在这篇简短的文章中，我们探讨了`@Valid`和`@Validated`注释之间的主要区别。

最后，对于任何基本的验证，我们将在方法调用中使用 JSR `@Valid`注释。另一方面，对于任何组验证，包括[组序列](https://web.archive.org/web/20220827110142/https://docs.oracle.com/javaee/7/api/javax/validation/GroupSequence.html)，我们需要在方法调用中使用 Spring 的`@Validated`注释。还需要`@Valid `注释来触发嵌套属性的验证。

和往常一样，本文中的代码可以从 GitHub 上的[处获得。](https://web.archive.org/web/20220827110142/https://github.com/eugenp/tutorials/tree/master/spring-boot-modules/spring-boot-mvc-3)