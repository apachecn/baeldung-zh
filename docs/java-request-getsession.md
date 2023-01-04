# request.getSession()和 request.getSession(true)之间的区别

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-request-getsession>

## 1.概观

在这个快速教程中，我们将看到调用`HttpServletRequest#` `getSession()`和`HttpServletRequest#` `getSession(boolean)`的区别。

## 2.有什么区别？

*方法 getSession()* 和`getSession(boolean)` 非常相似。不过，还是有一点不同。区别在于，如果会话不存在，是否应该创建它。

**调用`getSession() `和`getSession(true) `功能相同**:获取当前会话，如果还不存在，则创建一个。

不过，调用 **`getSession(false)`会检索当前会话，如果当前会话不存在，则返回`null`。**除了别的以外，当我们想要询问会话是否存在时，这非常方便。

## 3.例子

在本例中，我们考虑这种情况:

*   用户输入`user id `并登录到应用程序
*   然后，用户输入`user name`和`age`，并希望为登录用户更新这些细节

我们将在会话中存储用户值，以理解`HttpServletRequest#getSession()` 和`HttpServletRequest#getSession(boolean).`的用法

首先，让我们创建一个 servlet，我们在它的`doGet()`方法中使用了`HttpServletRequest#getSession()` :

```java
protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
    HttpSession session = request.getSession();
    session.setAttribute("userId", request.getParameter("userId"));
} 
```

此时，servlet 将检索现有的会话，或者为登录用户创建一个新的会话(如果它不存在的话)。

接下来，我们将在会话中设置`userName`属性。

因为我们想要更新相应用户 id 的用户详细信息，所以我们想要相同的会话，而不想创建新的会话来存储用户名。

所以现在，我们将使用`HttpServletRequest#getSession(boolean)` 和`false`值`:`

```java
protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
    HttpSession session = request.getSession(false);
    if (session != null) {
        session.setAttribute("userName", request.getParameter("userName"));
    }
}
```

这将导致在先前设置`userId`的同一会话上设置`userName`属性。

## 4.结论

在本教程中，我们已经解释了`HttpServletRequest#getSession()`和`HttpServletRequest#getSession(boolean)`方法之间的区别。

GitHub 上的[提供了完整的示例。](https://web.archive.org/web/20221116132054/https://github.com/eugenp/tutorials/tree/master/web-modules/javax-servlets)