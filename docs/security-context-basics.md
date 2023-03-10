# 安全上下文基础:用户、主题和主体

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/security-context-basics>

## 1.概观

安全性是任何 Java 应用程序的基础部分。此外，我们可以找到许多可以处理安全问题的安全框架。此外，我们在这些框架中使用一些常见的术语，如主体、委托人和用户。

在本教程中， **我们将讲解安全框架的这些基本概念** 。此外，我们将展示他们的关系和差异。

## 2.科目

在安全上下文中，主题代表请求的来源。s主体是获取资源信息或修改资源的实体。此外，主体还可以是用户、程序、进程、文件、计算机、数据库等。

例如，一个人需要授权访问资源和应用程序来验证请求源。在这种情况下，这个人就是主语。

让我们来看看基于[`JAAS`](/web/20220630020859/https://www.baeldung.com/java-authentication-authorization-service)框架实现的例子:

```java
Subject subject = loginContext.getSubject();
PrivilegedAction privilegedAction = new ResourceAction();
Subject.doAsPrivileged(subject, privilegedAction, null); 
```

## 3.校长

成功通过身份验证后，我们就有了一个填充了许多相关身份的主题，如角色、社会安全号(SSN)等。换句话说，这些标识符是主体，主体代表它们。

例如，一个人可能有一个账号主体(“87654-3210”)和其他唯一标识符，以区别于其他主体。

让我们看看如何在成功登录后创建一个 `UserPrincipal` 并将其添加到一个`Subject:`

```java
@Override
public boolean commit() throws LoginException {
    if (!loginSucceeded) {
        return false;
    }
    userPrincipal = new UserPrincipal(username);
    subject.getPrincipals().add(userPrincipal);
    return true;
}
```

## 4.用户

通常，用户代表访问资源以执行某些操作或完成工作任务的人。

此外，我们可以将用户用作委托人，另一方面，委托人是分配给用户的身份。`UserPrincipal ` 是上一节 `.`中讨论的 `JAAS` 框架中用户的一个极好的例子

## 5.主体、委托人和用户之间的区别

正如我们在上面几节中看到的，我们可以通过使用主体来表示同一个用户身份的不同方面。它们是主体的子集，而用户是委托人的子集，委托人指的是最终用户或交互式操作者。

## 6.结论

在本教程中，我们讨论了在大多数安全框架中常见的主题、主体和用户的定义。此外，我们还展示了它们之间的差异。

所有这些例子和代码片段的实现都可以在 GitHub 项目中找到。