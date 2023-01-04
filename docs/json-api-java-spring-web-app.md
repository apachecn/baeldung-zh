# Spring 应用程序中的 JSON API

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/json-api-java-spring-web-app>

## 1。概述

在本文中，我们将开始探索**JSON-API[T2 规范](https://web.archive.org/web/20221208143841/http://jsonapi.org/)**以及如何将其集成到 Spring backed REST API 中。

我们将使用 Java 中 JSON-API 的 [Katharsis](https://web.archive.org/web/20221208143841/https://github.com/katharsis-project/katharsis-framework) 实现——我们将建立一个 Katharsis 支持的 Spring 应用程序——所以我们所需要的就是一个 Spring 应用程序。

## 2。肚子

首先，让我们看看我们的 maven 配置——我们需要将以下依赖项添加到我们的`pom.xml`中:

```
<dependency>
    <groupId>io.katharsis</groupId>
    <artifactId>katharsis-spring</artifactId>
    <version>3.0.2</version>
</dependency>
```

## 3。用户资源

接下来，让我们看看我们的用户资源:

```
@JsonApiResource(type = "users")
public class User {

    @JsonApiId
    private Long id;

    private String name;

    private String email;
}
```

请注意:

*   `@JsonApiResource`注释用于定义我们的资源`User`
*   `@JsonApiId`注释用于定义资源标识符

简单地说，这个例子的持久性将是一个 Spring 数据仓库:

```
public interface UserRepository extends JpaRepository<User, Long> {}
```

## 4。资源库

接下来，让我们讨论我们的资源库——每个资源都应该有一个`ResourceRepositoryV2`来发布其上可用的 API 操作:

```
@Component
public class UserResourceRepository implements ResourceRepositoryV2<User, Long> {

    @Autowired
    private UserRepository userRepository;

    @Override
    public User findOne(Long id, QuerySpec querySpec) {
        Optional<User> user = userRepository.findById(id); 
        return user.isPresent()? user.get() : null;
    }

    @Override
    public ResourceList<User> findAll(QuerySpec querySpec) {
        return querySpec.apply(userRepository.findAll());
    }

    @Override
    public ResourceList<User> findAll(Iterable<Long> ids, QuerySpec querySpec) {
        return querySpec.apply(userRepository.findAllById(ids));
    }

    @Override
    public <S extends User> S save(S entity) {
        return userRepository.save(entity);
    }

    @Override
    public void delete(Long id) {
        userRepository.deleteById(id);
    }

    @Override
    public Class<User> getResourceClass() {
        return User.class;
    }

    @Override
    public <S extends User> S create(S entity) {
        return save(entity);
    }
}
```

这里有一个简短的说明——这当然**与弹簧控制器**非常相似。

## 5。Katharsis 配置

因为我们正在使用`katharsis-spring`，所以我们需要做的就是在我们的 Spring Boot 应用程序中导入`KatharsisConfigV3`:

```
@Import(KatharsisConfigV3.class)
```

并在我们的`application.properties`中配置 Katharsis 参数:

```
katharsis.domainName=http://localhost:8080
katharsis.pathPrefix=/
```

这样，我们现在可以开始使用 API 了；例如:

*   GET " `http://localhost:8080/users`":获取所有用户。
*   POST " `http://localhost:8080/users`":添加新用户，等等。

## 6。关系

接下来，让我们讨论如何在 JSON API 中处理实体关系。

### 6.1。角色资源

首先，让我们介绍一种新资源—`Role`:

```
@JsonApiResource(type = "roles")
public class Role {

    @JsonApiId
    private Long id;

    private String name;

    @JsonApiRelation
    private Set<User> users;
}
```

然后在`User`和`Role`之间建立多对多关系:

```
@JsonApiRelation(serialize=SerializeType.EAGER)
private Set<Role> roles;
```

### 6.2。角色资源库

很快——这是我们的`Role`资源库:

```
@Component
public class RoleResourceRepository implements ResourceRepositoryV2<Role, Long> {

    @Autowired
    private RoleRepository roleRepository;

    @Override
    public Role findOne(Long id, QuerySpec querySpec) {
        Optional<Role> role = roleRepository.findById(id); 
        return role.isPresent()? role.get() : null;
    }

    @Override
    public ResourceList<Role> findAll(QuerySpec querySpec) {
        return querySpec.apply(roleRepository.findAll());
    }

    @Override
    public ResourceList<Role> findAll(Iterable<Long> ids, QuerySpec querySpec) {
        return querySpec.apply(roleRepository.findAllById(ids));
    }

    @Override
    public <S extends Role> S save(S entity) {
        return roleRepository.save(entity);
    }

    @Override
    public void delete(Long id) {
        roleRepository.deleteById(id);
    }

    @Override
    public Class<Role> getResourceClass() {
        return Role.class;
    }

    @Override
    public <S extends Role> S create(S entity) {
        return save(entity);
    }
}
```

这里需要理解的重要一点是，这种单一资源回购不处理关系方面——这需要一个单独的存储库。

### 6.3。关系库

为了处理`User`–`Role`之间的多对多关系，我们需要创建一种新型的存储库:

```
@Component
public class UserToRoleRelationshipRepository implements RelationshipRepositoryV2<User, Long, Role, Long> {

    @Autowired
    private UserRepository userRepository;

    @Autowired
    private RoleRepository roleRepository;

    @Override
    public void setRelation(User User, Long roleId, String fieldName) {}

    @Override
    public void setRelations(User user, Iterable<Long> roleIds, String fieldName) {
        Set<Role> roles = new HashSet<Role>();
        roles.addAll(roleRepository.findAllById(roleIds));
        user.setRoles(roles);
        userRepository.save(user);
    }

    @Override
    public void addRelations(User user, Iterable<Long> roleIds, String fieldName) {
        Set<Role> roles = user.getRoles();
        roles.addAll(roleRepository.findAllById(roleIds));
        user.setRoles(roles);
        userRepository.save(user);
    }

    @Override
    public void removeRelations(User user, Iterable<Long> roleIds, String fieldName) {
        Set<Role> roles = user.getRoles();
        roles.removeAll(roleRepository.findAllById(roleIds));
        user.setRoles(roles);
        userRepository.save(user);
    }

    @Override
    public Role findOneTarget(Long sourceId, String fieldName, QuerySpec querySpec) {
        return null;
    }

    @Override
    public ResourceList<Role> findManyTargets(Long sourceId, String fieldName, QuerySpec querySpec) {
        final Optional<User> userOptional = userRepository.findById(sourceId);
        User user = userOptional.isPresent() ? userOptional.get() : new User();
        return  querySpec.apply(user.getRoles());
    }

    @Override
    public Class<User> getSourceResourceClass() {
        return User.class;
    }

    @Override
    public Class<Role> getTargetResourceClass() {
        return Role.class;
    }
}
```

我们在这里忽略了关系库中的单个方法。

## 7。测试

最后，让我们分析几个请求，真正理解 JSON-API 输出是什么样子的。

我们将开始检索单个用户资源(id = 2):

**获取 http://localhost:8080/users/2**

```
{
    "data":{
        "type":"users",
        "id":"2",
        "attributes":{
            "email":"[[email protected]](/web/20221208143841/https://www.baeldung.com/cdn-cgi/l/email-protection)",
            "username":"tom"
        },
        "relationships":{
            "roles":{
                "links":{
                    "self":"http://localhost:8080/users/2/relationships/roles",
                    "related":"http://localhost:8080/users/2/roles"
                }
            }
        },
        "links":{
            "self":"http://localhost:8080/users/2"
        }
    },
    "included":[
        {
            "type":"roles",
            "id":"1",
            "attributes":{
                "name":"ROLE_USER"
            },
            "relationships":{
                "users":{
                    "links":{
                        "self":"http://localhost:8080/roles/1/relationships/users",
                        "related":"http://localhost:8080/roles/1/users"
                    }
                }
            },
            "links":{
                "self":"http://localhost:8080/roles/1"
            }
        }
    ]
}
```

外卖:

*   资源的主要属性见`**data.attributes**`
*   资源的主要关系见`**data.relationships**`
*   当我们使用`@JsonApiRelation(serialize=SerializeType.EAGER)`作为`roles`关系时，它包含在 JSON 中，并在节点**中找到包含的**

接下来，让我们获取包含角色的集合资源:

**获取 http://localhost:8080/roles**

```
{
    "data":[
        {
            "type":"roles",
            "id":"1",
            "attributes":{
                "name":"ROLE_USER"
            },
            "relationships":{
                "users":{
                    "links":{
                        "self":"http://localhost:8080/roles/1/relationships/users",
                        "related":"http://localhost:8080/roles/1/users"
                    }
                }
            },
            "links":{
                "self":"http://localhost:8080/roles/1"
            }
        },
        {
            "type":"roles",
            "id":"2",
            "attributes":{
                "name":"ROLE_ADMIN"
            },
            "relationships":{
                "users":{
                    "links":{
                        "self":"http://localhost:8080/roles/2/relationships/users",
                        "related":"http://localhost:8080/roles/2/users"
                    }
                }
            },
            "links":{
                "self":"http://localhost:8080/roles/2"
            }
        }
    ],
    "included":[

    ]
}
```

这里的快速收获是，我们获得系统中的所有角色——作为**数据**节点中的数组

## 8。结论

JSON-API 是一个很棒的规范——最终我们在 API 中使用 JSON 的方式增加了一些结构，并真正为一个真正的超媒体 API 提供了动力。

这篇文章探索了在 Spring 应用程序中设置它的一种方法。但是不管实现如何，规范本身——在我看来——是非常非常有前途的工作。

该示例的完整源代码可以在 GitHub 上的[处获得。这是一个 Maven 项目，可以导入并按原样运行。](https://web.archive.org/web/20221208143841/https://github.com/eugenp/tutorials/tree/master/spring-katharsis)