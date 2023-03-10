# Spring MVC 自定义验证

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-mvc-custom-validator>

## 1。概述

通常，当我们需要验证用户输入时，Spring MVC 会提供标准的预定义验证器。

然而，当我们需要验证更特殊类型的输入时，我们有能力创建我们自己的定制验证逻辑。

在本教程中，我们将做到这一点；我们将创建一个自定义验证器来验证一个带有电话号码字段的表单，然后我们将展示一个用于多个字段的自定义验证器。

本教程重点介绍 Spring MVC。我们题为[Spring Boot 验证](/web/20221004131738/https://www.baeldung.com/spring-boot-bean-validation)的文章描述了如何在 Spring Boot 创建定制验证。

## 2。设置

为了从 API 中获益，我们将把依赖项添加到我们的`pom.xml`文件中:

```java
<dependency>
    <groupId>org.hibernate</groupId>
    <artifactId>hibernate-validator</artifactId>
    <version>6.0.10.Final</version>
</dependency> 
```

依赖关系的最新版本可以在[这里](https://web.archive.org/web/20221004131738/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22org.hibernate%22%20AND%20a%3A%22hibernate-validator%22)查看。

如果我们使用 Spring Boot，那么我们只能添加`[spring-boot-starter-web](/web/20221004131738/https://www.baeldung.com/spring-boot-starters),`，这也将带来`hibernate-validator`依赖。

## 3。自定义验证

创建一个定制的验证器需要推出我们自己的注释，并在我们的模型中使用它来执行验证规则。

因此，让我们创建我们的**自定义验证器，它检查电话号码**。电话号码必须至少有 8 位数字，但不能超过 11 位。

## 4。新注解

让我们创建一个新的`@interface`来定义我们的注释:

```java
@Documented
@Constraint(validatedBy = ContactNumberValidator.class)
@Target( { ElementType.METHOD, ElementType.FIELD })
@Retention(RetentionPolicy.RUNTIME)
public @interface ContactNumberConstraint {
    String message() default "Invalid phone number";
    Class<?>[] groups() default {};
    Class<? extends Payload>[] payload() default {};
}
```

使用`@Constraint` 注释，我们定义了将要验证我们字段的类。`message()` 是显示在用户界面上的错误信息。最后，附加代码大多是样板代码，以符合 Spring 标准。

## 5。创建验证器

现在让我们创建一个执行验证规则的验证器类:

```java
public class ContactNumberValidator implements 
  ConstraintValidator<ContactNumberConstraint, String> {

    @Override
    public void initialize(ContactNumberConstraint contactNumber) {
    }

    @Override
    public boolean isValid(String contactField,
      ConstraintValidatorContext cxt) {
        return contactField != null && contactField.matches("[0-9]+")
          && (contactField.length() > 8) && (contactField.length() < 14);
    }

}
```

验证类实现了`ConstraintValidator` 接口，还必须实现`isValid`方法；正是在这个方法中，我们定义了我们的验证规则。

自然，我们在这里用一个简单的验证规则来展示验证器是如何工作的。

`ConstraintValidator` 定义逻辑来验证给定对象的给定约束。实施必须遵守以下限制:

*   该对象必须解析为非参数化类型
*   对象的泛型参数必须是无界通配符类型

## 6。应用验证注释

在我们的例子中，我们创建了一个简单的类，其中有一个应用验证规则的字段。在这里，我们设置要验证的注释字段:

```java
@ContactNumberConstraint
private String phone;
```

我们定义了一个字符串字段，并用我们的自定义注释对其进行了注释，在我们的控制器中，我们创建了映射并处理了任何错误:

```java
@Controller
public class ValidatedPhoneController {

    @GetMapping("/validatePhone")
    public String loadFormPage(Model m) {
        m.addAttribute("validatedPhone", new ValidatedPhone());
        return "phoneHome";
    }

    @PostMapping("/addValidatePhone")
    public String submitForm(@Valid ValidatedPhone validatedPhone,
      BindingResult result, Model m) {
        if(result.hasErrors()) {
            return "phoneHome";
        }
        m.addAttribute("message", "Successfully saved phone: "
          + validatedPhone.toString());
        return "phoneHome";
    }   
}
```

我们定义了这个简单的控制器，它有一个单独的`JSP`页面，并使用了`submitForm` 方法来强制验证我们的电话号码。

## 7。视图

我们的视图是一个基本的 JSP 页面，其表单只有一个字段。当用户提交表单时，我们的自定义验证器将验证该字段，并重定向到相同的页面，显示验证成功或失败的消息:

```java
<form:form 
  action="/${pageContext.request.contextPath}/addValidatePhone"
  modelAttribute="validatedPhone">
    <label for="phoneInput">Phone: </label>
    <form:input path="phone" id="phoneInput" />
    <form:errors path="phone" cssClass="error" />
    <input type="submit" value="Submit" />
</form:form> 
```

## 8。测试

现在，让我们测试一下我们的控制器，看看它是否给出了适当的响应和视图:

```java
@Test
public void givenPhonePageUri_whenMockMvc_thenReturnsPhonePage(){
    this.mockMvc.
      perform(get("/validatePhone")).andExpect(view().name("phoneHome"));
}
```

我们还要测试我们的字段是否基于用户输入进行了验证:

```java
@Test
public void 
  givenPhoneURIWithPostAndFormData_whenMockMVC_thenVerifyErrorResponse() {

    this.mockMvc.perform(MockMvcRequestBuilders.post("/addValidatePhone").
      accept(MediaType.TEXT_HTML).
      param("phoneInput", "123")).
      andExpect(model().attributeHasFieldErrorCode(
          "validatedPhone","phone","ContactNumberConstraint")).
      andExpect(view().name("phoneHome")).
      andExpect(status().isOk()).
      andDo(print());
}
```

在测试中，我们向用户提供输入“123”，正如我们所料，一切都正常，我们在客户端看到了错误。

## 9.自定义类级别验证

还可以在类级别定义自定义验证注释，以验证该类的多个属性。

这个场景的一个常见用例是验证一个类的两个字段是否有匹配的值。

### 9.1。创建注释

让我们添加一个名为`FieldsValueMatch`的新注释，它可以在以后应用于一个类。注释将有两个参数，`field`和`fieldMatch,`，它们代表要比较的字段的名称:

```java
@Constraint(validatedBy = FieldsValueMatchValidator.class)
@Target({ ElementType.TYPE })
@Retention(RetentionPolicy.RUNTIME)
public @interface FieldsValueMatch {

    String message() default "Fields values don't match!";

    String field();

    String fieldMatch();

    @Target({ ElementType.TYPE })
    @Retention(RetentionPolicy.RUNTIME)
    @interface List {
        FieldsValueMatch[] value();
    }
}
```

我们可以看到我们的自定义注释还包含一个`List`子接口，用于在一个类上定义多个`FieldsValueMatch`注释。

### 9.2。创建验证器

接下来我们需要添加包含实际验证逻辑的`FieldsValueMatchValidator`类:

```java
public class FieldsValueMatchValidator 
  implements ConstraintValidator<FieldsValueMatch, Object> {

    private String field;
    private String fieldMatch;

    public void initialize(FieldsValueMatch constraintAnnotation) {
        this.field = constraintAnnotation.field();
        this.fieldMatch = constraintAnnotation.fieldMatch();
    }

    public boolean isValid(Object value, 
      ConstraintValidatorContext context) {

        Object fieldValue = new BeanWrapperImpl(value)
          .getPropertyValue(field);
        Object fieldMatchValue = new BeanWrapperImpl(value)
          .getPropertyValue(fieldMatch);

        if (fieldValue != null) {
            return fieldValue.equals(fieldMatchValue);
        } else {
            return fieldMatchValue == null;
        }
    }
}
```

`isValid()`方法检索两个字段的值，并检查它们是否相等。

### 9.3。应用注释

让我们为用户注册所需的数据创建一个`NewUserForm`模型类。它将有两个`email`和`password`属性，以及两个`verifyEmail`和`verifyPassword`属性来重新输入这两个值。

因为我们有两个字段要检查它们对应的匹配字段，所以让我们在`NewUserForm`类上添加两个`@FieldsValueMatch`注释，一个用于`email`值，一个用于`password`值:

```java
@FieldsValueMatch.List({ 
    @FieldsValueMatch(
      field = "password", 
      fieldMatch = "verifyPassword", 
      message = "Passwords do not match!"
    ), 
    @FieldsValueMatch(
      field = "email", 
      fieldMatch = "verifyEmail", 
      message = "Email addresses do not match!"
    )
})
public class NewUserForm {
    private String email;
    private String verifyEmail;
    private String password;
    private String verifyPassword;

    // standard constructor, getters, setters
}
```

为了在 Spring MVC 中验证模型，让我们创建一个带有`/user` POST 映射的控制器，它接收一个用`@Valid`注释的`NewUserForm`对象，并验证是否有任何验证错误:

```java
@Controller
public class NewUserController {

    @GetMapping("/user")
    public String loadFormPage(Model model) {
        model.addAttribute("newUserForm", new NewUserForm());
        return "userHome";
    }

    @PostMapping("/user")
    public String submitForm(@Valid NewUserForm newUserForm, 
      BindingResult result, Model model) {
        if (result.hasErrors()) {
            return "userHome";
        }
        model.addAttribute("message", "Valid form");
        return "userHome";
    }
}
```

### 9.4。测试注释

为了验证我们的自定义类级注释，让我们编写一个`JUnit`测试，将匹配信息发送到`/user`端点，然后验证响应不包含错误:

```java
public class ClassValidationMvcTest {
  private MockMvc mockMvc;

    @Before
    public void setup(){
        this.mockMvc = MockMvcBuilders
          .standaloneSetup(new NewUserController()).build();
    }

    @Test
    public void givenMatchingEmailPassword_whenPostNewUserForm_thenOk() 
      throws Exception {
        this.mockMvc.perform(MockMvcRequestBuilders
          .post("/user")
          .accept(MediaType.TEXT_HTML).
          .param("email", "[[email protected]](/web/20221004131738/https://www.baeldung.com/cdn-cgi/l/email-protection)")
          .param("verifyEmail", "[[email protected]](/web/20221004131738/https://www.baeldung.com/cdn-cgi/l/email-protection)")
          .param("password", "pass")
          .param("verifyPassword", "pass"))
          .andExpect(model().errorCount(0))
          .andExpect(status().isOk());
    }
}
```

然后，我们还将添加一个`JUnit`测试，它向`/user`端点发送不匹配的信息，并断言结果将包含两个错误:

```java
@Test
public void givenNotMatchingEmailPassword_whenPostNewUserForm_thenOk() 
  throws Exception {
    this.mockMvc.perform(MockMvcRequestBuilders
      .post("/user")
      .accept(MediaType.TEXT_HTML)
      .param("email", "[[email protected]](/web/20221004131738/https://www.baeldung.com/cdn-cgi/l/email-protection)")
      .param("verifyEmail", "[[email protected]](/web/20221004131738/https://www.baeldung.com/cdn-cgi/l/email-protection)")
      .param("password", "pass")
      .param("verifyPassword", "passsss"))
      .andExpect(model().errorCount(2))
      .andExpect(status().isOk());
    }
```

## 10。总结

在这篇简短的文章中，我们学习了如何创建自定义验证器来验证一个字段或类，然后将它们连接到 Spring MVC 中。

和往常一样，这篇文章的代码可以在 Github 的[上找到。](https://web.archive.org/web/20221004131738/https://github.com/eugenp/tutorials/tree/master/spring-web-modules/spring-mvc-basics-5)