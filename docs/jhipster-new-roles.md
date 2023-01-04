# 在 JHipster 中创建新的角色和权限

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/jhipster-new-roles>

## 1.概观

JHipster 有两个默认角色——用户和管理员——但有时我们需要添加自己的角色。

在本教程中，我们将创建一个名为 MANAGER 的新角色，用于为用户提供额外的权限。

请注意，**杰普斯特使用术语`authorities`与`roles`** 有些互换。不管怎样，我们本质上指的是同一件事。

## 2.代码更改

创建新角色的第一步是用**更新类`AuthoritiesConstants`** 。这个文件是在我们创建一个新的 JHipster 应用程序时自动生成的，它包含应用程序中所有角色和权限的常量。

要创建新的经理角色，我们只需在该文件中添加一个新的常量:

```
public static final String MANAGER = "ROLE_MANAGER";
```

## 3.模式更改

下一步是定义我们的数据存储中的新角色。

JHipster 支持各种持久数据存储，并创建一个初始设置任务，用用户和权限填充数据存储。

为了在数据库设置中添加一个新角色，**我们必须编辑`InitialSetupMigration.java`文件**。它已经有了一个名为`addAuthorities`的方法，我们只需将我们的新角色添加到现有代码中:

```
public void addAuthorities(MongoTemplate mongoTemplate) {
    // Add these lines after the existing, auto-generated code
    Authority managerAuthority = new Authority();
    managerAuthority.setName(AuthoritiesConstants.MANAGER);
    mongoTemplate.save(managerAuthority);
}
```

这个例子使用 MongoDB，但是步骤与 JHipster 支持的其他持久性存储非常相似。

**注意，一些数据存储，如 H2，仅仅依赖于一个名为`authorities.csv,`** 的文件，因此没有任何需要更新的生成代码。

## 4.利用我们的新角色

现在我们已经定义了一个新的角色，让我们看看如何在代码中使用它。

### 4.1.Java 代码

在后端，有两种主要的方法来检查用户是否有权执行操作。

首先，**如果我们想要限制对特定 API 的访问，我们可以修改`SecurityConfiguration`** :

```
public void configure(HttpSecurity http) throws Exception {
    http
        .authorizeRequests()
            .antMatchers("/management/**").hasAuthority(AuthoritiesConstants.MANAGER);
}
```

其次，**我们可以在应用程序**的任何地方使用`SecurityUtils`来检查用户是否处于某个角色中:

```
if (SecurityUtils.isCurrentUserInRole(AuthoritiesConstants.MANAGER)) {
    // perform some logic that is applicable to manager role
}
```

### 4.2.前端

JHipster 提供了两种在前端检查角色的方法。注意，这些例子使用了 Angular，但是 React 也有类似的结构。

首先，**模板中的任何元素都可以使用`*jhiHasAnyAuthority`指令**。它接受单个字符串或字符串数组:

```
<div *jhiHasAnyAuthority="'ROLE_MANAGER'">
    <!-- manager related code here -->
</div>
```

其次，**`Principal`类可以检查**用户是否有特定的角色:

```
isManager() {
    return this.principal.identity()
      .then(account => this.principal.hasAnyAuthority(['ROLE_MANAGER']));
}
```

## 5.结论

在本文中，我们看到了在 JHipster 中创建新的角色和权限是多么简单。虽然默认的用户和管理员角色对于大多数应用程序来说是一个很好的起点，但是额外的角色提供了更多的灵活性。

有了额外的角色，我们可以更好地控制哪些用户可以访问 API，以及他们可以在前端看到哪些数据。

和往常一样，这段代码可以在 GitHub 上找到。