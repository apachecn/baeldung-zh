# Grails 3 和 GORM 简介

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/grails-gorm-tutorial>

## 1。概述

这是对 Grails 3 和 GORM 的快速介绍。

当然，我们将使用 Groovy，并且——不言而喻——框架也将 Hibernate 用于 ORM，Spring 框架用于依赖注入，SiteMash 用于布局和主题，等等。

## 2。数据源配置

我们可以在不指定任何显式数据源配置的情况下开始——默认情况下，Grails 将 HSQLDB 数据库用于开发和测试环境。

但是如果您想改变这些默认值，您可以在`application.yml`中定义您选择的数据源:

```java
environments:
    development:
        dataSource:
             driverClassName : "com.mysql.jdbc.Driver"           
             url : "jdbc:mysql://localhost:8080/test"
             dialect : org.hibernate.dialect.MySQL5InnoDBDialect 
```

同样，我们可以在这里创建多个环境，如果需要，可以在`development`旁边创建。

## 3。域名

Grails 能够根据数据库配置中的`dbCreate`属性为我们的域类创建数据库结构。

让我们在这里定义其中一个域类:

```java
Class User {
    String userName
    String password
    String email
    String age
    static constraints = {
        userName blank: false, unique: true
        password size: 5..10, blank: false
        email email: true, blank: true
    }
}
```

注意我们是如何在模型中指定我们的**验证约束的，这使得事情变得漂亮、干净，并且没有注释。**

当实体被持久化时，Grails 将自动检查这些约束，如果这些约束中的任何一个被破坏，框架将抛出适当的验证异常。

我们还可以在模型的`mapping`属性中指定 GORM 映射:

```java
static mapping = { sort "userName" }
```

现在，如果我们调用`User.list()` ，我们将得到按照`username` 排序的结果**。**

我们当然可以通过将 sort 传递给 list API 来获得相同的结果:

```java
User.list(sort: "userName")
```

## 4。积垢操作

当我们看 API 操作时，**脚手架**在开始时扮演了一个非常有趣的角色；它允许您为域类生成基本的 CRUD API，包括:

*   必要的观点
*   标准 CRUD 操作的控制器动作
*   两种类型:动态和静态

下面是动态脚手架的工作原理:

```java
class UserController {
    static scaffold = true
}
```

通过编写这一行代码，框架将在运行时生成 7 个方法:显示、编辑、删除、创建、保存和更新。这些将作为特定领域实体的 API 发布。

静态脚手架示例:

*   要创建带有脚手架的视图，请使用:"`grails generate-views User`"
*   要使用脚手架创建控制器和视图，请使用:"`grails generate-controller User`"
*   要在一个命令中创建所有内容，请使用:"`grails generate-all User`"

这些命令将为特定的域对象自动生成必要的管道。

现在让我们快速地看一下如何使用这些操作——例如，对于我们的`User`域对象。

为了**创建新的“用户”记录**:

```java
def user = new User(username: "test", password: "test123", email: "[[email protected]](/web/20220122061328/https://www.baeldung.com/cdn-cgi/l/email-protection)", age: 14)
user.save()
```

**取单条记录**:


```java
def user = User.get(1) 
```

这个`get` API 将在可编辑模式下检索域对象。对于只读模式，我们可以使用`read` API:

```java
def user = User.read(1)
```

更新现有记录 :

```java
def user = User.get(1)
user.userName = "testUpdate"
user.age = 20
user.save() 
```

以及对已有记录的简单删除操作:

```java
def user = User.get(1)
user.delete()
```

## 5。GORM 查询

### 5.1。`find`

让我们从`find` API 开始:

```java
def user = User.find("from User as u where u.username = 'test' ")
```

我们也可以使用不同的语法来传入参数:

```java
def user = User.find("from User as u where u.username?", ['test'])
```

我们也可以使用命名参数:

```java
def user = User.find("from User as u where u.username=?", [username: 'test'])
```

### 5.2。`findBy`

Grails 提供了一个动态查找工具，它使用域属性在运行时执行查询并返回第一个匹配的记录:

```java
def user = User.findByUsername("test")
user = User.findByUsernameAndAge("test", 20)
user = User.findByUsernameLike("tes")
user = User.findByUsernameAndAgeNotEquals("test", "100")
```

你可以在这里找到更多表情[。](https://web.archive.org/web/20220122061328/https://grails.github.io/grails-doc/3.0.x/guide/GORM.html#finders)

### 5.3。标准

我们还可以使用一些灵活的标准来检索数据:

```java
def user = User.find { username == "test"}
def user = User.createCriteria()
def results = user.list {
    like ("userName", "te%")
    and 
    {
        between("age", 10, 20)
    }
    order("userName", "desc")
}
```

这里有一个简单的注意事项——当使用标准查询时，使用“{ }”而不是“()”。

### 5.4。执行查询/更新

GORM 还支持 HQL 查询语法——对于读操作:

```java
def user = User.executeQuery(
  "select u.userName from User u where u.userName = ?", ['test'])
```

以及写操作:

```java
def user = User.executeUpdate("delete User u where u.username =?", ['test'])
```

## 6。结论

这是对 Grails 和 GORM 的一个非常快速的介绍——用作框架入门指南。