# 基于权限的阿帕奇·希罗访问控制

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/apache-shiro-access-control>

## 1。简介

在本教程中，我们将看看如何用[阿帕奇·希罗](/web/20220628092334/https://www.baeldung.com/apache-shiro) Java 安全框架实现**细粒度的基于权限的访问控制**。

## 2。设置

我们将使用与 Shiro 简介相同的设置—也就是说，我们将只把 [`shiro-core`](https://web.archive.org/web/20220628092334/https://search.maven.org/search?q=g:org.apache.shiro%20AND%20a:shiro-core&core=gav) 模块添加到我们的依赖项中:

```java
<dependency>
    <groupId>org.apache.shiro</groupId>
    <artifactId>shiro-core</artifactId>
    <version>1.4.1</version>
</dependency>
```

此外，出于测试目的，我们将使用一个简单的 INI 领域，将下面的`shiro.ini` 文件放在类路径的根目录下:

```java
[users]
jane.admin = password, admin
john.editor = password2, editor
zoe.author = password3, author

[roles]
admin = *
editor = articles:*
author = articles:create, articles:edit
```

然后，我们将使用上述领域初始化 Shiro:

```java
IniRealm iniRealm = new IniRealm("classpath:shiro.ini");
SecurityManager securityManager = new DefaultSecurityManager(iniRealm);
SecurityUtils.setSecurityManager(securityManager);
```

## 3。角色和权限

通常，当我们谈论身份验证和授权时，我们主要关注用户和角色的概念。

特别是，**角色是应用程序或服务的用户的交叉类。**因此，具有特定角色的所有用户都可以访问一些资源和操作，而对应用程序或服务的其他部分的访问可能受到限制。

角色集通常是预先设计好的，很少为了适应新的业务需求而改变。但是，角色也可以动态定义，例如，由管理员定义。

使用 Shiro，我们有几种方法来测试用户是否有特定的角色。最直接的方法是使用`hasRole` 方法:

```java
Subject subject = SecurityUtils.getSubject();
if (subject.hasRole("admin")) {       
    logger.info("Welcome Admin");              
}
```

### 3.1。权限

然而，如果我们通过测试用户是否有特定的角色来检查授权，就会出现问题。事实上，我们正在硬编码角色和权限之间的关系。换句话说，当我们想要授予或撤销对资源的访问权时，我们必须修改源代码。当然，这也意味着重建和重新部署。

我们可以做得更好；这就是为什么我们现在要引入权限的概念。权限代表软件可以做什么，我们可以授权或拒绝，而不是谁可以做什么。例如，“编辑当前用户的个人资料”、“批准文档”或“创建新文章”。

Shiro 对权限做了很少的假设。在最简单的情况下，权限是简单的字符串:

```java
Subject subject = SecurityUtils.getSubject();
if (subject.isPermitted("articles:create")) {
    //Create a new article
}
```

注意，在 Shiro 中，权限的使用完全是可选的。

### 3.2。将权限关联到用户

Shiro 有一个将权限与角色或单个用户相关联的灵活模型。然而，典型的领域，包括我们在本教程中使用的简单 INI 领域，只将权限与角色相关联。

因此，由一个`Principal,` 标识的用户有几个角色，每个角色有几个`Permission`。

例如，我们可以看到，在我们的 INI 文件中，用户`zoe.author` 拥有`author` 角色，这给了他们`articles:create`和`articles:edit`权限:

```java
[users]
zoe.author = password3, author
#Other users...

[roles]
author = articles:create, articles:edit
#Other roles...
```

同样，其他领域类型(如内置的 JDBC 领域)可以配置为将权限与角色相关联。

## 4。通配符权限

**Shiro 中权限的默认实现是通配符权限，**这是各种权限方案的灵活表示。

我们在 Shiro 中用字符串表示通配符权限。权限字符串由冒号分隔的一个或多个组件组成，例如:

```java
articles:edit:1
```

字符串每个部分的含义取决于应用程序，因为 Shiro 不强制任何规则。但是，在上面的例子中，我们可以非常清楚地将字符串解释为一个层次结构:

1.  我们公开的资源类别(文章)
2.  对此类资源的操作(编辑)
3.  我们希望允许或拒绝其操作的特定资源的 id

resource:action:id 的三层结构是 Shiro 应用程序中的一种常见模式，因为它在表示许多不同的场景时既简单又有效。

因此，我们可以重温一下前面的例子来遵循这个方案:

```java
Subject subject = SecurityUtils.getSubject();
if (subject.isPermitted("articles:edit:123")) {
    //Edit article with id 123
}
```

注意**通配符权限字符串中的组件数量不一定是三个，即使通常情况下是三个组件。**

### 4.1。权限隐含和实例级粒度

当我们将通配符权限与 Shiro 权限的另一个特性——蕴涵相结合时，它们大放异彩。

**当我们测试角色时，我们测试确切的成员:**要么一个`Subject` 有一个特定的角色，要么没有。换句话说，Shiro 测试角色的平等性。

另一方面，**当我们测试权限时，我们测试隐含:**`Subject`的权限是否隐含了我们要测试的权限？

隐含的具体含义取决于许可的实现。事实上，对于通配符权限，其含义是部分字符串匹配，顾名思义，可能有通配符。

因此，假设我们为`author` 角色分配了以下权限:

```java
[roles]
author = articles:*
```

然后，具有 `author`角色的每个人将被允许对文章进行所有可能的操作:

```java
Subject subject = SecurityUtils.getSubject();
if (subject.isPermitted("articles:create")) {
    //Create a new article
}
```

也就是说，字符串`articles:*`将匹配第一个组件是`articles.`的任何通配符权限

有了这个方案，我们既可以分配非常具体的权限——对具有给定 id 的特定资源的特定操作——也可以分配广泛的权限，例如编辑任何文章或对任何文章执行任何操作。

当然，出于性能方面的原因，因为这种暗示不是简单的相等比较，**我们应该总是针对最具体的权限**进行测试:

```java
if (subject.isPermitted("articles:edit:1")) { //Better than "articles:*"
    //Edit article
}
```

## 5。自定义权限实现

让我们简单介绍一下权限定制。尽管通配符权限涵盖了广泛的场景，但我们可能希望用为我们的应用程序定制的解决方案来替换它们。

假设我们需要对路径上的权限进行建模，使得路径上的权限`implies` 对所有子路径都有权限。实际上，我们可以使用通配符权限来完成任务，但是让我们忽略它。

那么，我们需要什么？

1.  一个`Permission` 实现
2.  告诉四郎这件事

我们来看看如何做到这两点。

### 5.1。编写权限实现

**`Permission` 实现是一个只有一个方法的类— `implies` :**

```java
public class PathPermission implements Permission {

    private final Path path;

    public PathPermission(Path path) {
        this.path = path;
    }

    @Override
    public boolean implies(Permission p) {
        if(p instanceof PathPermission) {
            return ((PathPermission) p).path.startsWith(path);
        }
        return false;
    }
}
```

如果`this`暗示另一个权限对象，该方法返回`true` ，否则返回`false` 。

### 5.2。告诉 Shiro 我们的实现

然后，将`Permission `实现集成到 Shiro 中有各种方法，但是最直接的方法是**将一个自定义`PermissionResolver`注入到我们的`Realm` :** 中

```java
IniRealm realm = new IniRealm();
Ini ini = Ini.fromResourcePath(Main.class.getResource("/com/.../shiro.ini").getPath());
realm.setIni(ini);
realm.setPermissionResolver(new PathPermissionResolver());
realm.init();

SecurityManager securityManager = new DefaultSecurityManager(realm);
```

**`PermissionResolver` 负责** **负责** **将我们权限的字符串表示转换成实际的`Permission` 对象:**

```java
public class PathPermissionResolver implements PermissionResolver {
    @Override
    public Permission resolvePermission(String permissionString) {
        return new PathPermission(Paths.get(permissionString));
    }
}
```

我们必须用基于路径的权限修改之前的`shiro.ini` :

```java
[roles]
admin = /
editor = /articles
author = /articles/drafts
```

然后，我们将能够检查路径上的权限:

```java
if(currentUser.isPermitted("/articles/drafts/new-article")) {
    log.info("You can access articles");
}
```

请注意，这里我们是以编程方式配置一个简单的领域。在一个典型的应用程序中，我们将使用一个`shiro.ini` 文件或其他方式(如 Spring)来配置 Shiro 和领域。真实世界的`shiro.ini`文件可能包含:

```java
[main]
permissionResolver = com.baeldung.shiro.permissions.custom.PathPermissionResolver
dataSource = org.apache.shiro.jndi.JndiObjectFactory
dataSource.resourceName = java://app/jdbc/myDataSource

jdbcRealm = org.apache.shiro.realm.jdbc.JdbcRealm
jdbcRealm.dataSource = $dataSource 
jdbcRealm.permissionResolver = $permissionResolver
```

## 6。结论

在本文中，我们回顾了阿帕奇·希罗是如何实现基于权限的访问控制的。

和往常一样，所有这些例子和代码片段的实现都可以在 GitHub 上的[处获得。](https://web.archive.org/web/20220628092334/https://github.com/eugenp/tutorials/tree/master/apache-shiro)