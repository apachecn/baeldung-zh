# JHipster 使用外部服务进行身份验证

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/jhipster-authentication-external-service>

## 1.介绍

默认情况下， [JHipster](https://web.archive.org/web/20220926200546/https://www.jhipster.tech/) 应用程序使用本地数据存储来保存用户名和密码。然而，在许多真实的场景中，可能需要使用现有的外部服务进行身份验证。

在本教程中，我们将了解如何在 JHipster 中使用外部服务进行身份验证。这可以是任何众所周知的服务，如 LDAP、社交登录或任何接受用户名和密码的任意服务。

## 2.JHipster 中的身份验证

JHipster 使用 [Spring Security](/web/20220926200546/https://www.baeldung.com/security-spring) 进行认证。**`AuthenticationManager`类负责验证用户名和密码。**

JHipster 中默认的`AuthenticationManager`只是根据本地数据存储检查用户名和密码。这可能是 MySQL、PostgreSQL、MongoDB 或 JHipster 支持的任何替代方案。

需要注意的是**`AuthenticationManager`仅用于初始登录**。一旦用户通过了身份验证，他们就会收到一个 JSON Web 令牌(JWT ),用于后续的 API 调用。

### 2.1.在 JHipster 中更改身份验证

但是，如果我们已经有了一个包含用户名和密码的数据存储，或者一个为我们执行身份验证的服务，该怎么办呢？

**为了提供一个定制的认证方案，我们简单地创建一个类型为`AuthenticationManager`** 的新 bean。这将优先于默认实现。

下面的例子展示了如何创建一个自定义的`AuthenticationManager`。它只有一个方法可以实现:

```java
public class CustomAuthenticationManager implements AuthenticationManager {
    @Override
    public Authentication authenticate(Authentication authentication) throws AuthenticationException {
        try {
            ResponseEntity<LoginResponse> response =
                restTemplate.postForEntity(REMOTE_LOGIN_URL, loginRequest, LoginResponse.class);

            if(response.getStatusCode().is2xxSuccessful()) {
                String login = authentication.getPrincipal().toString();
                User user = userService.getUserWithAuthoritiesByLogin(login)
                  .orElseGet(() -> userService.createUser(
                    createUserDTO(response.getBody(), authentication)));
                return createAuthentication(authentication, user);
            }
            else {
                throw new BadCredentialsException("Invalid username or password");
            }
        }
        catch (Exception e) {
            throw new AuthenticationServiceException("Failed to login", e);
        }
    }
}
```

在这个例子中，我们将用户名和凭证从`Authentication`对象传递给外部 API。

如果调用成功，我们返回一个新的`UsernamePasswordAuthenticationToken`来表示成功。注意**我们还创建了一个本地用户条目，我们将在稍后的**中讨论。

如果调用失败，我们抛出**一些`AuthenticationException`** 的变体，这样 Spring Security 将优雅地为我们回退。

这个例子是有意简单地展示自定义身份验证的基础。然而，它可以执行更复杂的操作，比如 LDAP 绑定和认证或者使用 OAuth。

## 3.其他考虑

到目前为止，我们一直关注 JHipster 中的认证流程。但是我们的 JHipster 应用程序还有几个地方需要修改。

### 3.1.前端代码

默认的 JHipster 代码实现了以下用户注册和激活过程:

*   用户使用他们的电子邮件和其他必需的详细信息注册一个帐户
*   JHipster 创建一个帐户并将其设置为非活动状态，然后向新用户发送一封带有激活链接的电子邮件
*   单击该链接后，用户的帐户将被标记为活动

密码重置也有类似的流程。

当 JHipster 管理用户帐户时，这些都是有意义的。但是当我们依赖外部服务进行身份验证时，就不需要它们了。

**因此，我们需要采取措施确保用户**无法访问这些账户管理功能。

这意味着从 Angular 或 React 代码中删除它们，这取决于 JHipster 应用程序中使用的框架。

以 Angular 为例，默认的登录提示包括密码重置和注册的链接。我们应该将它们从`app/shared/login/login.component.html`中移除:

```java
<div class="alert alert-warning">
  <a class="alert-link" (click)="requestResetPassword()">Did you forget your password?</a>
</div>
<div class="alert alert-warning">
  <span>You don't have an account yet?</span>
   <a class="alert-link" (click)="register()">Register a new account</a>
</div>
```

我们还必须从`app/layouts/navbar/navbar.component.html`中删除不需要的导航菜单项:

```java
<li *ngSwitchCase="true">
  <a class="dropdown-item" routerLink="password" routerLinkActive="active" (click)="collapseNavbar()">
    <fa-icon icon="clock" fixedWidth="true"></fa-icon>
    <span>Password</span>
  </a>
</li>
```

和

```java
<li *ngSwitchCase="false">
  <a class="dropdown-item" routerLink="register" routerLinkActive="active" (click)="collapseNavbar()">
    <fa-icon icon="user-plus" fixedWidth="true"></fa-icon>
    <span>Register</span>
  </a>
</li>
```

即使我们删除了所有的链接，**用户仍然可以手动导航到这些页面**。最后一步是从`app/account/account.route.ts`中移除未使用的角度路线。

完成此操作后，应该只保留设置路线:

```java
import { settingsRoute } from './';
const ACCOUNT_ROUTES = [settingsRoute];
```

### 3.2.Java APIs

在大多数情况下，只需删除前端帐户管理代码就足够了。然而，**为了绝对确定帐户管理代码没有被调用，我们也可以锁定相关的 Java API**。

最快的方法是更新`SecurityConfiguration`类来拒绝所有对相关 URL 的请求:

```java
.antMatchers("/api/register").denyAll()
.antMatchers("/api/activate").denyAll()
.antMatchers("/api/account/reset-password/init").denyAll()
.antMatchers("/api/account/reset-password/finish").denyAll()
```

这将阻止对 API 的任何远程访问，而不必删除任何代码。

### 3.3.电子邮件模板

JHipster 应用程序附带了一套默认的电子邮件模板，用于帐户注册、激活和密码重置。**前面的步骤将有效地防止默认电子邮件被发送**，但是在某些情况下，我们可能想要重用它们。

例如，我们可能希望在用户首次登录时发送一封欢迎电子邮件。默认模板包括帐户激活的步骤，所以我们必须修改它。

所有的电子邮件模板都位于`resources/templates/mail`中。它们是 HTML 文件，使用[百里香叶](/web/20220926200546/https://www.baeldung.com/thymeleaf-in-spring-mvc)将数据从 Java 代码传递到电子邮件中。

我们所要做的就是编辑模板以包含所需的文本和布局，然后使用`MailService`发送它。

### 3.4.角色

当我们创建本地 JHipster 用户条目时，**我们还必须确保它至少有一个角色**。通常，默认的`USER`角色对于新账户来说已经足够了。

如果外部服务提供了自己的角色映射，我们还有两个额外的步骤:

1.  确保 JHipster 中存在任何自定义角色
2.  更新我们的自定义`AuthenticationManager`以在创建新用户时设置自定义角色

JHipster 还提供了一个管理界面，用于向用户添加和删除角色。

### 3.5.帐户删除

值得一提的是，JHipster 还提供了一个帐户移除管理视图和 API。此视图仅对管理员用户可用。

**我们可以删除和限制这个代码，就像我们对帐户注册和密码重置所做的那样，但这并不是真正必要的**。当有人登录时，我们的定制`AuthenticationManager`将总是创建一个新的帐户条目，所以删除帐户实际上没有多大作用。

## 4.结论

在本教程中，我们看到了如何用我们自己的身份验证方案替换默认的 JHipster 身份验证代码。这可能是 LDAP、OIDC 或任何其他接受用户名和密码的服务。

我们还看到，使用外部认证服务还需要对 JHipster 应用程序的其他方面进行一些更改。这包括前端视图、API 等等。

和往常一样，本教程的示例代码可以在 GitHub 上的[处获得。](https://web.archive.org/web/20220926200546/https://github.com/eugenp/tutorials/tree/master/jhipster-5/bookstore-monolith)