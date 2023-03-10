# 用 Spring RestTemplate 获取 JSON 对象列表

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-resttemplate-json-list>

## 1.概观

为了获取信息，我们的服务经常需要与其他 REST 服务进行通信。

在 Spring 中，我们可以使用 [`RestTemplate`](/web/20221126231529/https://www.baeldung.com/rest-template) 来执行同步 HTTP 请求。数据通常以 JSON 的形式返回，`RestTemplate`可以为我们转换。

在本教程中，我们将探索如何在 Java 中将 JSON 数组转换成三种不同的对象结构:`Object`的`Array`、[的`Array`和 POJO 的`List`。](/web/20221126231529/https://www.baeldung.com/java-pojo-class#what-is-a-pojo)

## 2.JSON、POJO 和服务

假设我们有一个端点`http://localhost:8080/users`返回一个用户列表，如下所示:

```java
[{
  "id": 1,
  "name": "user1",
}, {
  "id": 2,
  "name": "user2"
}]
```

我们将需要相应的`User `类来处理数据:

```java
public class User {
    private int id;
    private String name;

    // getters and setters..
}
```

对于我们的接口实现，我们写了一个`UserConsumerServiceImpl `，用`RestTemplate `作为它的依赖:

```java
public class UserConsumerServiceImpl implements UserConsumerService {

    private final RestTemplate restTemplate;

    public UserConsumerServiceImpl(RestTemplate restTemplate) {
        this.restTemplate = restTemplate;
    }

...
}
```

## 3.映射 JSON 对象列表

当对 REST 请求的响应是 JSON 数组时，有几种方法可以将其转换成 Java 集合。

让我们看看这些选项，看看它们让我们处理返回的数据有多容易。我们将研究提取由 REST 服务返回的一些用户对象的用户名。

### 3.1.`RestTemplate`用对象数组

首先，让我们用`RestTemplate.getForEntity`进行调用，并使用一个`Object[]`类型的 `ResponseEntity`来收集响应:

```java
ResponseEntity<Object[]> responseEntity =
   restTemplate.getForEntity(BASE_URL, Object[].class);
```

接下来，我们可以将身体提取到我们的数组`Object`:

```java
Object[] objects = responseEntity.getBody();
```

这里实际的`Object`只是一些包含我们的数据但不使用我们的`User`类型的任意结构。让我们把它转换成我们的`User`对象。

为此，我们需要一个`ObjectMapper`:

```java
ObjectMapper mapper = new ObjectMapper();
```

我们可以内联声明它，尽管这通常是作为类的`private static final`成员来完成的。

最后，我们准备提取用户名:

```java
return Arrays.stream(objects)
  .map(object -> mapper.convertValue(object, User.class))
  .map(User::getName)
  .collect(Collectors.toList());
```

使用这种方法，我们可以在 Java 中将一个数组`anything`读入一个数组`Object`。例如，如果我们只想计算结果，这可能会很方便。

然而，它不适合于进一步加工。我们必须付出额外的努力将它转换成我们可以使用的类型。

**当我们要求[杰克森反序列化器](/web/20221126231529/https://www.baeldung.com/jackson-object-mapper-tutorial)产生`Object`作为目标类型时，它实际上将 JSON 反序列化成一系列`LinkedHashMap`对象**。用`convertValue`进行后期处理是一种低效的开销。

如果我们一开始就向杰克逊提供我们想要的类型，我们就可以避免这种情况。

### 3.2.`RestTemplate`带用户阵列

我们可以提供`User[] `到`RestTemplate`，而不是`Object[]`:

```java
 ResponseEntity<User[]> responseEntity = 
    restTemplate.getForEntity(BASE_URL, User[].class); 
  User[] userArray = responseEntity.getBody();
  return Arrays.stream(userArray) 
    .map(User::getName) 
    .collect(Collectors.toList());
```

我们可以看到我们不再需要`ObjectMapper.convertValue`。在`ResponseEntity`里面有`User`对象。但是我们仍然需要做一些额外的转换来使用 Java `Stream` API，并让我们的代码处理列表。

### 3.3.`RestTemplate`带用户列表和`ParameterizedTypeReference`

如果我们需要杰克森产生一个` User`的`List`而不是一个数组的便利，我们需要描述我们想要创建的`List`。为此，我们不得不使用`RestTemplate.` `exchange`。

这个方法使用一个由[匿名内部类](/web/20221126231529/https://www.baeldung.com/java-anonymous-classes)产生的`ParameterizedTypeReference` :

```java
ResponseEntity<List<User>> responseEntity = 
  restTemplate.exchange(
    BASE_URL,
    HttpMethod.GET,
    null,
    new ParameterizedTypeReference<List<User>>() {}
  );
List<User> users = responseEntity.getBody();
return users.stream()
  .map(User::getName)
  .collect(Collectors.toList());
```

这就产生了我们想要使用的`List`。

让我们仔细看看为什么我们需要使用 `ParameterizedTypeReference`。

在前两个例子中，Spring 可以很容易地将 JSON 反序列化为一个`User.class`类型的令牌，其中的类型信息在运行时完全可用。

然而，对于泛型，如果我们试图使用`List<User>.class`，就会发生类型擦除。所以，杰克逊将无法确定`<>`内部的类型。

我们可以通过使用名为`ParameterizedTypeReference`的超类型令牌来克服这个问题。将其实例化为匿名内部类— `new ParameterizedTypeReference<List<User>>() {}` —利用了泛型类的子类包含编译时类型信息的事实，这些信息不会被类型擦除，并且可以通过反射来使用。

## 4.结论

在本文中，我们看到了使用`RestTemplate`处理 JSON 对象的三种不同方式。我们看到了如何指定`Object`的数组类型和我们自己的自定义类。

然后我们学习了如何通过使用`ParameterizedTypeReference`来提供类型信息以产生一个`List`。

和往常一样，这篇文章的代码可以在 GitHub 上找到[。](https://web.archive.org/web/20221126231529/https://github.com/eugenp/tutorials/tree/master/spring-web-modules/spring-resttemplate-2)