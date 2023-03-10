# Spring Data MongoDB 中的查询指南

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/queries-in-spring-data-mongodb>

## 1。概述

本教程将着重于在 Spring Data MongoDB 中构建不同类型的查询。

我们将看到使用`Query`和`Criteria`类查询文档、自动生成的查询方法、JSON 查询和 QueryDSL。

对于 Maven 的设置，请看一下我们的介绍文章。

## 2。单据查询

用 Spring 数据查询 MongoDB 的一个更常见的方法是使用`Query`和`Criteria`类，它们非常接近于本地操作符。

### 2.1。`Is`

这只是一个使用等式的标准。让我们看看它是如何工作的。

在下面的例子中，我们将寻找名为`Eric`的用户。

让我们看看我们的数据库:

```java
[
    {
        "_id" : ObjectId("55c0e5e5511f0a164a581907"),
        "_class" : "org.baeldung.model.User",
        "name" : "Eric",
        "age" : 45
    },
    {
        "_id" : ObjectId("55c0e5e5511f0a164a581908"),
        "_class" : "org.baeldung.model.User",
        "name" : "Antony",
        "age" : 55
    }
}
```

现在让我们看看查询代码:

```java
Query query = new Query();
query.addCriteria(Criteria.where("name").is("Eric"));
List<User> users = mongoTemplate.find(query, User.class); 
```

正如所料，该逻辑返回:

```java
{
    "_id" : ObjectId("55c0e5e5511f0a164a581907"),
    "_class" : "org.baeldung.model.User",
    "name" : "Eric",
    "age" : 45
}
```

### 2.2。`Regex`

一种更加灵活和强大的查询类型是正则表达式。这使用 MongoDB `$regex`创建了一个标准，该标准返回适合这个字段的正则表达式的所有记录。

它的工作方式类似于`startingWith` 和 `endingWith`操作。

在本例中，我们将查找所有名称以`A`开头的用户。

以下是数据库的状态:

```java
[
    {
        "_id" : ObjectId("55c0e5e5511f0a164a581907"),
        "_class" : "org.baeldung.model.User",
        "name" : "Eric",
        "age" : 45
    },
    {
        "_id" : ObjectId("55c0e5e5511f0a164a581908"),
        "_class" : "org.baeldung.model.User",
        "name" : "Antony",
        "age" : 33
    },
    {
        "_id" : ObjectId("55c0e5e5511f0a164a581909"),
        "_class" : "org.baeldung.model.User",
        "name" : "Alice",
        "age" : 35
    }
]
```

现在让我们创建查询:

```java
Query query = new Query();
query.addCriteria(Criteria.where("name").regex("^A"));
List<User> users = mongoTemplate.find(query,User.class);
```

这将运行并返回 2 条记录:

```java
[
    {
        "_id" : ObjectId("55c0e5e5511f0a164a581908"),
        "_class" : "org.baeldung.model.User",
        "name" : "Antony",
        "age" : 33
    },
    {
        "_id" : ObjectId("55c0e5e5511f0a164a581909"),
        "_class" : "org.baeldung.model.User",
        "name" : "Alice",
        "age" : 35
    }
]
```

下面是另一个简单的例子，这一次寻找名字以`c`结尾的所有用户:

```java
Query query = new Query();
query.addCriteria(Criteria.where("name").regex("c$"));
List<User> users = mongoTemplate.find(query, User.class); 
```

所以结果会是:

```java
{
    "_id" : ObjectId("55c0e5e5511f0a164a581907"),
    "_class" : "org.baeldung.model.User",
    "name" : "Eric",
    "age" : 45
}
```

### 2.3。`Lt`和`gt`

这些运算符使用`$lt`(小于)和 `$gt`(大于)运算符创建一个标准。

让我们举一个简单的例子，我们正在寻找年龄在 20 到 50 岁之间的所有用户。

该数据库是:

```java
[
    {
        "_id" : ObjectId("55c0e5e5511f0a164a581907"),
        "_class" : "org.baeldung.model.User",
        "name" : "Eric",
        "age" : 45
    },
    {
        "_id" : ObjectId("55c0e5e5511f0a164a581908"),
        "_class" : "org.baeldung.model.User",
        "name" : "Antony",
        "age" : 55
    }
}
```

查询代码:

```java
Query query = new Query();
query.addCriteria(Criteria.where("age").lt(50).gt(20));
List<User> users = mongoTemplate.find(query,User.class); 
```

年龄大于 20 岁小于 50 岁的所有用户的结果:

```java
{
    "_id" : ObjectId("55c0e5e5511f0a164a581907"),
    "_class" : "org.baeldung.model.User",
    "name" : "Eric",
    "age" : 45
}
```

### 2.4。 `Sort`

`Sort`用于指定结果的排序顺序。

以下示例返回按年龄升序排序的所有用户。

首先，这是现有的数据:

```java
[
    {
        "_id" : ObjectId("55c0e5e5511f0a164a581907"),
        "_class" : "org.baeldung.model.User",
        "name" : "Eric",
        "age" : 45
    },
    {
        "_id" : ObjectId("55c0e5e5511f0a164a581908"),
        "_class" : "org.baeldung.model.User",
        "name" : "Antony",
        "age" : 33
    },
    {
        "_id" : ObjectId("55c0e5e5511f0a164a581909"),
        "_class" : "org.baeldung.model.User",
        "name" : "Alice",
        "age" : 35
    }
] 
```

执行`sort`后:

```java
Query query = new Query();
query.with(Sort.by(Sort.Direction.ASC, "age"));
List<User> users = mongoTemplate.find(query,User.class); 
```

下面是查询的结果，按照`age`进行了很好的排序:

```java
[
    {
        "_id" : ObjectId("55c0e5e5511f0a164a581908"),
        "_class" : "org.baeldung.model.User",
        "name" : "Antony",
        "age" : 33
    },
    {
        "_id" : ObjectId("55c0e5e5511f0a164a581909"),
        "_class" : "org.baeldung.model.User",
        "name" : "Alice",
        "age" : 35
    },
    {
        "_id" : ObjectId("55c0e5e5511f0a164a581907"),
        "_class" : "org.baeldung.model.User",
        "name" : "Eric",
        "age" : 45
    }
]
```

### 2.5。 `Pageable`

让我们看一个使用分页的简单例子。

以下是数据库的状态:

```java
[
    {
        "_id" : ObjectId("55c0e5e5511f0a164a581907"),
        "_class" : "org.baeldung.model.User",
        "name" : "Eric",
        "age" : 45
    },
    {
        "_id" : ObjectId("55c0e5e5511f0a164a581908"),
        "_class" : "org.baeldung.model.User",
        "name" : "Antony",
        "age" : 33
    },
    {
        "_id" : ObjectId("55c0e5e5511f0a164a581909"),
        "_class" : "org.baeldung.model.User",
        "name" : "Alice",
        "age" : 35
    }
] 
```

下面是查询逻辑，简单地请求一个大小为 2 的页面:

```java
final Pageable pageableRequest = PageRequest.of(0, 2);
Query query = new Query();
query.with(pageableRequest); 
```

结果，正如所料，这两个文档:

```java
[
    {
        "_id" : ObjectId("55c0e5e5511f0a164a581907"),
        "_class" : "org.baeldung.model.User",
        "name" : "Eric",
        "age" : 45
    },
    {
        "_id" : ObjectId("55c0e5e5511f0a164a581908"),
        "_class" : "org.baeldung.model.User",
        "name" : "Antony",
        "age" : 33
    }
]
```

## 3。生成的查询方法

现在让我们探索 Spring Data 通常提供的更常见的查询类型，即根据方法名自动生成的查询。

为了利用这些类型的查询，我们唯一需要做的就是在存储库接口上声明方法:

```java
public interface UserRepository 
  extends MongoRepository<User, String>, QueryDslPredicateExecutor<User> {
    ...
}
```

### 3.1。`FindByX`

我们将从简单的开始，探索 findBy 类型的查询。在这种情况下，我们将使用按名称查找:

```java
List<User> findByName(String name);
```

就像上一节 2.1 中一样，查询将得到相同的结果，查找具有给定名称的所有用户:

```java
List<User> users = userRepository.findByName("Eric"); 
```

### 3.2。`StartingWith`和 `endingWith`

在 2.2 节中，我们探索了一个基于`regex`的查询。Starts 和 ends with 当然没有那么强大，但仍然非常有用，尤其是如果我们不需要实际实现它们的话。

这里有一个简单的操作示例:

```java
List<User> findByNameStartingWith(String regexp);
```

```java
List<User> findByNameEndingWith(String regexp);
```

当然，实际使用它的例子非常简单:

```java
List<User> users = userRepository.findByNameStartingWith("A"); 
```

```java
List<User> users = userRepository.findByNameEndingWith("c");
```

结果完全一样。

### 3.3。`Between`

类似于第 2.3 节，这将返回年龄在`ageGT`和`ageLT:` 之间的所有用户

```java
List<User> findByAgeBetween(int ageGT, int ageLT);
```

调用方法将导致找到完全相同的文档:

```java
List<User> users = userRepository.findByAgeBetween(20, 50); 
```

### 3.4。`Like`和`OrderBy`

这次让我们看一个更高级的例子，为生成的查询组合了两种类型的修饰符。

我们将查找姓名中包含字母`A,`的所有用户，并且我们还将按年龄对结果进行升序排序:

```java
List<User> users = userRepository.findByNameLikeOrderByAgeAsc("A"); 
```

对于我们在 2.4 节中使用的数据库，结果将是:

```java
[
    {
        "_id" : ObjectId("55c0e5e5511f0a164a581908"),
        "_class" : "org.baeldung.model.User",
        "name" : "Antony",
        "age" : 33
    },
    {
        "_id" : ObjectId("55c0e5e5511f0a164a581909"),
        "_class" : "org.baeldung.model.User",
        "name" : "Alice",
        "age" : 35
    }
]
```

## 4。JSON 查询方法

如果我们不能借助方法名或标准来表示查询，我们可以做一些更低级的事情，**使用`@Query`注释**。

有了这个注释，我们可以将原始查询指定为 Mongo JSON 查询字符串。

### 4.1。`FindBy`

让我们从简单开始，先看看我们如何表示**一个`find by`类型的方法**:

```java
@Query("{ 'name' : ?0 }")
List<User> findUsersByName(String name); 
```

此方法应该按名称返回用户。占位符 `?0`引用了该方法的第一个参数。

```java
List<User> users = userRepository.findUsersByName("Eric");
```

### 4.2。`$regex`

我们还可以看一下**一个正则表达式驱动的查询，**当然会产生与 2.2 和 3.2 节相同的结果:

```java
@Query("{ 'name' : { $regex: ?0 } }")
List<User> findUsersByRegexpName(String regexp);
```

用法也完全一样:

```java
List<User> users = userRepository.findUsersByRegexpName("^A"); 
```

```java
List<User> users = userRepository.findUsersByRegexpName("c$");
```

### 4.3。`$lt`和`$gt`

现在让我们实现 lt 和`gt`查询:

```java
@Query("{ 'age' : { $gt: ?0, $lt: ?1 } }")
List<User> findUsersByAgeBetween(int ageGT, int ageLT);
```

现在这个方法有两个参数，我们在原始查询中通过索引引用每个参数，`?0`和`?1:`

```java
List<User> users = userRepository.findUsersByAgeBetween(20, 50);
```

## 5。QueryDSL 查询

`MongoRepository`很好地支持了 [QueryDSL](https://web.archive.org/web/20220625163339/http://www.querydsl.com/) 项目，所以我们也可以在这里利用这个漂亮的、类型安全的 API。

### 5.1。美芬依赖

首先，让我们确保在 pom 中定义了正确的 Maven 依赖项:

```java
<dependency>
    <groupId>com.mysema.querydsl</groupId>
    <artifactId>querydsl-mongodb</artifactId>
    <version>4.3.1</version>
</dependency>
<dependency>
    <groupId>com.mysema.querydsl</groupId>
    <artifactId>querydsl-apt</artifactId>
    <version>4.3.1</version>
</dependency>
```

### 5.2。 `Q`-班级

QueryDSL 使用 Q-class 来创建查询，但是由于我们并不真的想要手工创建这些查询，**我们需要以某种方式生成它们**。

我们将使用 apt-maven-plugin 来完成这项工作:

```java
<plugin>    
    <groupId>com.mysema.maven</groupId>
    <artifactId>apt-maven-plugin</artifactId>
    <version>1.1.3</version>
    <executions>
        <execution>
            <goals>
                <goal>process</goal>
            </goals>
            <configuration>
                <outputDirectory>target/generated-sources/java</outputDirectory>
                <processor>
                  org.springframework.data.mongodb.repository.support.MongoAnnotationProcessor
                </processor>
            </configuration>
        </execution>
     </executions>
</plugin>
```

让我们看看`User`类，特别关注`@QueryEntity`注释:

```java
@QueryEntity 
@Document
public class User {

    @Id
    private String id;
    private String name;
    private Integer age;

    // standard getters and setters
}
```

在运行了 Maven 生命周期的`process` 目标(或之后的任何其他目标)之后，apt 插件**将在`target/generated-sources/java/{your package structure}`下生成新的类**:

```java
/**
 * QUser is a Querydsl query type for User
 */
@Generated("com.mysema.query.codegen.EntitySerializer")
public class QUser extends EntityPathBase<User> {

    private static final long serialVersionUID = ...;

    public static final QUser user = new QUser("user");

    public final NumberPath<Integer> age = createNumber("age", Integer.class);

    public final StringPath id = createString("id");

    public final StringPath name = createString("name");

    public QUser(String variable) {
        super(User.class, forVariable(variable));
    }

    public QUser(Path<? extends User> path) {
        super(path.getType(), path.getMetadata());
    }

    public QUser(PathMetadata<?> metadata) {
        super(User.class, metadata);
    }
}
```

正是因为有了这个类，我们才不需要创建查询。

顺便提一下，如果我们使用 Eclipse，引入这个插件将在 pom 中生成以下警告:

> `You need to run build with JDK or have tools.jar on the classpath. If this occurs during eclipse build make sure you run eclipse under JDK as well (com.mysema.maven:apt-maven-plugin:1.1.3:process:default:generate-sources`

Maven `install`工作正常，并且生成了`QUser`类，但是 pom 中突出显示了一个插件。

一个快速解决方法是手动指向`eclipse.ini`中的 JDK:

```java
...
-vm
{path_to_jdk}\jdk{your_version}\bin\javaw.exe
```

### 5.3。`The New Repository`

现在我们需要在我们的存储库中实际启用 QueryDSL 支持，这可以通过简单地用**扩展`QueryDslPredicateExecutor` 接口**来完成:

```java
public interface UserRepository extends 
  MongoRepository<User, String>, QuerydslPredicateExecutor<User>
```

### 5.4。`Eq`

启用支持后，**现在让我们实现与我们之前演示的相同的查询**。

我们从简单的等式开始:

```java
QUser qUser = new QUser("user");
Predicate predicate = qUser.name.eq("Eric");
List<User> users = (List<User>) userRepository.findAll(predicate);
```

### 5.5。`StartingWith`和`EndingWith`

类似地，让我们实现前面的查询，并查找名称以`A`开头的用户:

```java
QUser qUser = new QUser("user");
Predicate predicate = qUser.name.startsWith("A");
List<User> users = (List<User>) userRepository.findAll(predicate); 
```

以及以`c`结尾:

```java
QUser qUser = new QUser("user");
Predicate predicate = qUser.name.endsWith("c");
List<User> users = (List<User>) userRepository.findAll(predicate); 
```

结果与第 2.2、3.2 和 4.2 节中的结果相同。

### 5.6。`Between`

下一个查询将返回年龄在 20 到 50 岁之间的用户，类似于前面的部分:

```java
QUser qUser = new QUser("user");
Predicate predicate = qUser.age.between(20, 50);
List<User> users = (List<User>) userRepository.findAll(predicate);
```

## 6。结论

在本文中，我们探索了使用 Spring 数据 MongoDB 进行查询的许多方法。

后退一步，看看我们查询 MongoDB 的所有强大方法是很有趣的，从有限的控制一直到原始查询的完全控制。

所有这些例子和代码片段的实现都可以在 GitHub 项目中找到。