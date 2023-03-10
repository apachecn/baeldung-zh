# 基于 Spring 安全角色过滤 Jackson JSON 输出

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-security-role-filter-json>

## 1.概观

在这个快速教程中，我们将展示如何根据 Spring Security 中定义的用户角色过滤 JSON 序列化输出。

## 2.为什么我们需要过滤？

让我们考虑一个简单但常见的用例，其中我们有一个服务于不同角色用户的 web 应用程序。比如让这些角色分别是`User`和`Admin`。

首先，让我们定义一个需求，即 **`Admins`可以完全访问通过公共 REST API 公开的对象**的内部状态。相反， **`Users`应该只看到一组预定义的对象属性。**

我们将使用 [Spring 安全框架](/web/20221208143856/https://www.baeldung.com/security-spring)来防止对 web 应用程序资源的未授权访问。

让我们定义一个对象，我们将在 API 中将其作为 REST 响应负载返回:

```java
class Item {
    private int id;
    private String name;
    private String ownerName;

    // getters
}
```

当然，我们可以为应用程序中的每个角色定义一个单独的数据传输对象类。然而，这种方法会给我们的代码库带来无用的重复或复杂的类层次结构。

另一方面，我们可以使用杰克逊库的 JSON 视图特性。正如我们将在下一节看到的，它使得**定制 JSON 表示像在字段上添加注释**一样简单。

## 3.`@JsonView`注释

Jackson 库通过用`@JsonView`注释标记我们希望包含在 JSON 表示中的字段，支持定义**多重序列化/反序列化上下文**。该注释有一个`Class`类型的必需参数**，用于区分上下文。**

当在我们的类中用`@JsonView`标记字段时，我们应该记住，默认情况下，序列化上下文包括所有没有明确标记为视图一部分的属性。为了覆盖这种行为，我们可以禁用`DEFAULT_VIEW_INCLUSION`映射器特性。

首先，让我们**用一些内部类定义一个`View`类，我们将用它们作为`@JsonView`注释**的参数:

```java
class View {
    public static class User {}
    public static class Admin extends User {}
} 
```

接下来，我们将`@JsonView`注释添加到我们的类中，使得`ownerName`只能由管理员角色访问:

```java
@JsonView(View.User.class)
private int id;
@JsonView(View.User.class)
private String name;
@JsonView(View.Admin.class)
private String ownerName;
```

## 4.如何将`@JsonView`注释与 Spring Security 集成

现在，让我们添加一个包含所有角色及其名称的枚举。之后，让我们介绍 JSON 视图和安全角色之间的映射:

```java
enum Role {
    ROLE_USER,
    ROLE_ADMIN
}

class View {

    public static final Map<Role, Class> MAPPING = new HashMap<>();

    static {
        MAPPING.put(Role.ADMIN, Admin.class);
        MAPPING.put(Role.USER, User.class);
    }

    //...
}
```

最后，我们来到了集成的中心点。为了绑定 JSON 视图和 Spring 安全角色，我们需要**定义适用于应用程序中所有控制器方法的控制器通知**。

到目前为止，我们唯一需要做的就是用**覆盖`AbstractMappingJacksonResponseBodyAdvice`** 类的`beforeBodyWriteInternal`方法:

```java
@RestControllerAdvice
class SecurityJsonViewControllerAdvice extends AbstractMappingJacksonResponseBodyAdvice {

    @Override
    protected void beforeBodyWriteInternal(
      MappingJacksonValue bodyContainer,
      MediaType contentType,
      MethodParameter returnType,
      ServerHttpRequest request,
      ServerHttpResponse response) {
        if (SecurityContextHolder.getContext().getAuthentication() != null
          && SecurityContextHolder.getContext().getAuthentication().getAuthorities() != null) {
            Collection<? extends GrantedAuthority> authorities
              = SecurityContextHolder.getContext().getAuthentication().getAuthorities();
            List<Class> jsonViews = authorities.stream()
              .map(GrantedAuthority::getAuthority)
              .map(AppConfig.Role::valueOf)
              .map(View.MAPPING::get)
              .collect(Collectors.toList());
            if (jsonViews.size() == 1) {
                bodyContainer.setSerializationView(jsonViews.get(0));
                return;
            }
            throw new IllegalArgumentException("Ambiguous @JsonView declaration for roles "
              + authorities.stream()
              .map(GrantedAuthority::getAuthority).collect(Collectors.joining(",")));
        }
    }
}
```

这样，**我们应用程序的每个响应都将经过这个通知**，它将根据我们定义的角色映射找到合适的视图表示。注意，这种方法要求我们**在处理具有多重角色的用户时**要小心。

## 5.结论

在这个简短的教程中，我们学习了如何在基于 Spring 安全角色的 web 应用程序中过滤 JSON 输出。

所有相关代码都可以在 Github 上找到[。](https://web.archive.org/web/20221208143856/https://github.com/eugenp/tutorials/tree/master/spring-security-modules/spring-security-core)