# 春季安全注册教程

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-security-registration>

[This article is part of a series:](javascript:void(0);)• Spring Security Registration Tutorial (current article)[• The Registration Process With Spring Security](/web/20220905213025/https://www.baeldung.com/registration-with-spring-mvc-and-spring-security)
[• Registration – Activate a New Account by Email](/web/20220905213025/https://www.baeldung.com/registration-verify-user-by-email)
[• Spring Security Registration – Resend Verification Email](/web/20220905213025/https://www.baeldung.com/spring-security-registration-verification-email)
[• Registration with Spring Security – Password Encoding](/web/20220905213025/https://www.baeldung.com/spring-security-registration-password-encoding-bcrypt)
[• The Registration API becomes RESTful](/web/20220905213025/https://www.baeldung.com/registration-restful-api)
[• Spring Security – Reset Your Password](/web/20220905213025/https://www.baeldung.com/spring-security-registration-i-forgot-my-password)
[• Registration – Password Strength and Rules](/web/20220905213025/https://www.baeldung.com/registration-password-strength-and-rules)
[• Updating your Password](/web/20220905213025/https://www.baeldung.com/updating-your-password/)

为您的 web 应用程序构建一个成熟的、准备好投入生产的注册不仅仅是组装一个简单的注册页面。

有很多问题需要回答:

*   我如何验证新用户的电子邮件地址？
*   我如何正确安全地**存储用户凭证**？
*   如果用户**忘记了他们的密码**怎么办？
*   **用户自己改密码**怎么办？
*   密码应该有多强？我如何在应用程序中强制执行一些合理的默认设置，以便我的用户拥有好的、强有力的密码？
*   如果我有多种类型的用户怎么办？我需要一个好方法来**存储角色和权限**。
*   那么**安全问题**呢？我应该拥有它们吗？
*   我如何在良好的本地化支持下完成所有这些工作？这涉及到很多信息。

![Reg Basics - icon](img/648e874d25c2a294540f14f98c689d24.png)

## 注册流程基础

*   ***[注册流程](/web/20220905213025/https://www.baeldung.com/registration-with-spring-mvc-and-spring-security)***
**   ***[通过电子邮件激活新账户](/web/20220905213025/https://www.baeldung.com/registration-verify-user-by-email)*****   ***[注册——密码编码](/web/20220905213025/https://www.baeldung.com/spring-security-registration-password-encoding-bcrypt)*****   ***[注册 API 变成 RESTful](/web/20220905213025/https://www.baeldung.com/registration-restful-api)*****   ***[重设您的密码](/web/20220905213025/https://www.baeldung.com/spring-security-registration-i-forgot-my-password)*****   ***[密码强度和规则](/web/20220905213025/https://www.baeldung.com/registration-password-strength-and-rules)********