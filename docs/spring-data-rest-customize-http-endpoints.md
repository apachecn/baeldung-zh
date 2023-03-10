# 在 Spring Data REST 中定制 HTTP 端点

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-data-rest-customize-http-endpoints>

## 1.介绍

Spring Data REST 可以删除很多 REST 服务中常见的样板文件。

在本教程中，我们将探索如何**定制一些 Spring Data REST 的 HTTP 绑定默认值**。

## 2.Spring 数据休息库基础

首先，让我们**创建一个扩展`CrudRepository`接口**的空接口，指定我们实体的类型及其主键的类型:

```java
public interface UserRepository extends CrudRepository<WebsiteUser, Long> {}
```

默认情况下， **Spring 生成所有需要的映射**，通过适当的 HTTP 方法配置每个可访问的资源，并返回适当的状态代码。

如果我们不需要`CrudRepository`中定义的所有资源，我们可以扩展基本的`Repository`接口，**只定义我们想要的资源**:

```java
public interface UserRepository extends Repository<WebsiteUser, Long> {
  void deleteById(Long aLong);
}
```

收到请求后， **Spring 读取所使用的 HTTP 方法，并根据资源类型，调用我们的接口**中定义的适当方法(如果有的话),否则返回 HTTP 状态 `405 (Method Not Allowed)`。

参考上面的代码，当 Spring 收到删除请求时，它会执行我们的`deleteById`方法。

## 3.限制公开哪些 HTTP 方法

假设我们有一个用户管理系统。那么，我们可能会有一个`UserRepository.`

而且，由于我们使用的是 Spring Data REST，我们从它的扩展中获益良多:

```java
@RepositoryRestResource(collectionResourceRel = "users", path = "users")
public interface UserRepository extends CrudRepository<WebsiteUser, Long> {}
```

**我们所有的资源都使用默认的 CRUD 模式**公开，因此发出以下命令:

```java
curl -v -X DELETE http://localhost:8080/users/<existing_user_id>
```

将返回一个 HTTP 状态`204 (No Content returned)`来确认删除。

现在，让我们假设**我们想要对第三方隐藏`delete`方法**，同时能够在内部使用它。

然后，我们可以首先将`deleteById`方法签名添加到我们的接口中，这向 Spring Data REST 发出信号，表明我们将对其进行配置。

然后，我们可以使用注释`@RestResource(exported = false)`，它将**配置 Spring 在触发 HTTP 方法暴露**时跳过这个方法:

```java
@Override
@RestResource(exported = false)
void deleteById(Long aLong);
```

现在，如果我们重复上面显示的相同的`cUrl`命令，我们将收到一个 HTTP 状态`405 (Method Not Allowed)`。

## 4.自定义支持的 HTTP 方法

`@RestResource`注释还让我们能够**定制映射到存储库方法**的 URL 路径和由 [HATEOAS](/web/20220913030920/https://www.baeldung.com/spring-hateoas-tutorial) 资源发现返回的 JSON 中的链接 id。

为此，我们使用注释的可选参数:

*   `**path**`为 URL 路径
*   **`rel`** 为链接 id

让我们回到我们的`UserRepository`并添加一个简单的`findByEmail`方法:

```java
WebsiteUser findByEmail(@Param("email") String email);
```

通过执行`cUrl`到`http://localhost:8080/users/search/`，我们现在可以看到我们的新方法与其他资源一起列出:

```java
{
  "_links": {
    "findByEmail": {
      "href": "http://localhost:8080/users/search/findByEmail{?email}"
    },
    "self": {
      "href": "http://localhost:8080/users/search/"
    }
  }
}
```

**如果我们不喜欢默认路径，我们可以简单地添加`@RestResource`注释**，而不是改变存储库方法:

```java
@RestResource(path = "byEmail", rel = "customFindMethod")
WebsiteUser findByEmail(@Param("email") String email);
```

如果我们再次进行资源发现，生成的 JSON 将确认我们的更改:

```java
{
  "_links": {
    "customFindMethod": {
      "href": "http://localhost:8080/users/search/byEmail{?email}",
      "templated": true
    },
    "self": {
      "href": "http://localhost:8080/users/search/"
    }
  }
}
```

## 5.程序配置

有时我们需要更细粒度的配置来公开或限制对 HTTP 方法的访问。例如，对集合资源的 POST，以及对项目资源的 PUT 和 PATCH，都使用相同的`save`方法。

**从 Spring Data REST 3.1 开始，到 Spring Boot 2.1** ，**我们可以通过`ExposureConfiguration` 类改变特定 HTTP 方法**的暴露。这个特殊的配置类公开了一个基于 lambda 的 API 来定义全局和基于类型的规则。

例如，我们可以使用`ExposureConfiguration` 来限制针对`UserRepository`的补丁请求:

```java
public class RestConfig implements RepositoryRestConfigurer {
    @Override
    public void configureRepositoryRestConfiguration(RepositoryRestConfiguration restConfig,
      CorsRegistry cors) {
        ExposureConfiguration config = restConfig.getExposureConfiguration();
        config.forDomainType(WebsiteUser.class).withItemExposure((metadata, httpMethods) ->
          httpMethods.disable(HttpMethod.PATCH));
    }
}
```

## 6.结论

在本文中，我们探讨了如何配置 Spring Data REST 来定制资源中默认支持的 HTTP 方法。

像往常一样，本文中使用的例子可以在我们的 [GitHub 项目](https://web.archive.org/web/20220913030920/https://github.com/eugenp/tutorials/tree/master/persistence-modules/spring-data-rest)中找到。