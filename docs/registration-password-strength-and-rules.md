# 注册-密码强度和规则

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/registration-password-strength-and-rules>

[This article is part of a series:](javascript:void(0);)[• Spring Security Registration Tutorial](/web/20221226054238/https://www.baeldung.com/spring-security-registration)
[• The Registration Process With Spring Security](/web/20221226054238/https://www.baeldung.com/registration-with-spring-mvc-and-spring-security)
[• Registration – Activate a New Account by Email](/web/20221226054238/https://www.baeldung.com/registration-verify-user-by-email)
[• Spring Security Registration – Resend Verification Email](/web/20221226054238/https://www.baeldung.com/spring-security-registration-verification-email)
[• Registration with Spring Security – Password Encoding](/web/20221226054238/https://www.baeldung.com/spring-security-registration-password-encoding-bcrypt)
[• The Registration API becomes RESTful](/web/20221226054238/https://www.baeldung.com/registration-restful-api)
[• Spring Security – Reset Your Password](/web/20221226054238/https://www.baeldung.com/spring-security-registration-i-forgot-my-password)
• Registration – Password Strength and Rules (current article)[• Updating your Password](/web/20221226054238/https://www.baeldung.com/updating-your-password)

## 1。概述

在这个快速教程中，我们将看看如何在注册期间实现并展示**正确的密码约束。比如——密码应该包含一个特殊字符，或者至少有 8 个字符长。**

我们希望能够使用强大的密码规则——但我们不想实际手动实施这些规则。所以，我们要好好利用成熟的[帕萨特库](https://web.archive.org/web/20221226054238/http://www.passay.org/)。

## 2。自定义密码约束

首先，让我们创建一个自定义约束`ValidPassword`:

```java
@Documented
@Constraint(validatedBy = PasswordConstraintValidator.class)
@Target({ TYPE, FIELD, ANNOTATION_TYPE })
@Retention(RUNTIME)
public @interface ValidPassword {

    String message() default "Invalid Password";

    Class<?>[] groups() default {};

    Class<? extends Payload>[] payload() default {};

}
```

并在`UserDto`中使用它:

```java
@ValidPassword
private String password;
```

## 3。自定义密码验证器

现在，让我们使用这个库来创建一些强大的密码规则，而不必实际手动实现它们。

我们将创建密码验证器`PasswordConstraintValidator`–我们将定义密码的规则:

```java
public class PasswordConstraintValidator implements ConstraintValidator<ValidPassword, String> {

    @Override
    public void initialize(ValidPassword arg0) {
    }

    @Override
    public boolean isValid(String password, ConstraintValidatorContext context) {
        PasswordValidator validator = new PasswordValidator(Arrays.asList(
           new LengthRule(8, 30), 
           new UppercaseCharacterRule(1), 
           new DigitCharacterRule(1), 
           new SpecialCharacterRule(1), 
           new NumericalSequenceRule(3,false), 
           new AlphabeticalSequenceRule(3,false), 
           new QwertySequenceRule(3,false),
           new WhitespaceRule()));

        RuleResult result = validator.validate(new PasswordData(password));
        if (result.isValid()) {
            return true;
        }
        context.disableDefaultConstraintViolation();
        context.buildConstraintViolationWithTemplate(
          Joiner.on(",").join(validator.getMessages(result)))
          .addConstraintViolation();
        return false;
    }
}
```

请注意**我们如何在这里**创建新的约束冲突并禁用默认冲突——以防密码无效。

最后，让我们将`Passay`库添加到我们的 pom 中:

```java
<dependency>
	<groupId>org.passay</groupId>
	<artifactId>passay</artifactId>
	<version>1.0</version>
</dependency>
```

作为一点历史信息，Passay 是古老的 Java 库的后代。

## 4。JS 密码计

现在服务器端已经完成了，让我们看看客户端，用 JavaScript 实现一个简单的**`Password Strength`**功能。

我们将使用一个简单的 jQuery 插件-[jQuery 密码强度计用于 Twitter 引导](https://web.archive.org/web/20221226054238/https://plugins.jquery.com/pwstrength-bootstrap/)-在`registration.html`中显示密码强度:

```java
<input id="password" name="password" type="password"/>

<script src="http://ajax.googleapis.com/ajax/libs/jquery/1.11.2/jquery.min.js"></script>
<script src="pwstrength.js"></script>                  
<script type="text/javascript">
$(document).ready(function () {
    options = {
        common: {minChar:8},
        ui: {
            showVerdictsInsideProgressBar:true,
            showErrors:true,
            errorMessages:{
                wordLength: '<spring:message code="error.wordLength"/>',
                wordNotEmail: '<spring:message code="error.wordNotEmail"/>',
                wordSequences: '<spring:message code="error.wordSequences"/>',
                wordLowercase: '<spring:message code="error.wordLowercase"/>',
                wordUppercase: '<spring:message code="error.wordUppercase"/>',
                wordOneNumber: '<spring:message code="error.wordOneNumber"/>',
                wordOneSpecialChar: '<spring:message code="error.wordOneSpecialChar"/>'
            }
        }
    };
    $('#password').pwstrength(options);
});
</script>
```

## 5。结论

就是这样——一种简单但非常有用的方法，可以在客户端显示密码的强度，并在服务器端实施特定的密码规则。

本教程的**完整实现**可以在[的 github 项目](https://web.archive.org/web/20221226054238/https://github.com/Baeldung/spring-security-registration "The Full Registration/Authentication Example Project on Github ")中找到——这是一个基于 Eclipse 的项目，所以它应该很容易导入和运行。

Next **»**[Updating your Password](/web/20221226054238/https://www.baeldung.com/updating-your-password)**«** Previous[Spring Security – Reset Your Password](/web/20221226054238/https://www.baeldung.com/spring-security-registration-i-forgot-my-password)