# DTO 模式(数据传输对象)

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-dto-pattern>

## 1.概观

在本教程中，我们将讨论 [DTO 模式](https://web.archive.org/web/20220928221252/https://martinfowler.com/eaaCatalog/dataTransferObject.html)，它是什么，以及如何和何时使用它。到最后，我们会知道如何正确使用它。

## 2.模式

dto 或数据传输对象是在进程间传送数据的对象，目的是减少方法调用的次数。这个模式是由马丁·福勒在他的书《T2·EAA》中首次提出的。

Fowler 解释说，该模式的主要目的是通过在单个调用中批量处理多个参数来减少与服务器的往返。这减少了此类远程操作中的网络开销。

另一个好处是序列化逻辑的封装(将对象结构和数据转换为可以存储和传输的特定格式的机制)。它提供了序列化细微差别的单点更改。它还将领域模型从表示层中分离出来，允许两者独立地改变。

## 3.怎么用？

dto 通常被创建为[POJO](/web/20220928221252/https://www.baeldung.com/java-pojo-class)。它们**是不包含业务逻辑的平面数据结构。**它们只包含存储、访问器和最终与序列化或解析相关的方法。

数据从[域模型](https://web.archive.org/web/20220928221252/https://martinfowler.com/eaaCatalog/domainModel.html)映射到 dto，通常是通过表示层或外观层中的映射器组件。

下图说明了组件之间的交互: [![](img/82b37e0f644587d14e9a29a57d0adda3.png)](/web/20220928221252/https://www.baeldung.com/wp-content/uploads/2021/08/layers-4.svg)

## 4.什么时候用？

dto 在具有远程调用的系统中很方便，因为它们有助于减少远程调用的数量。

当领域模型由许多不同的对象组成，并且表示模型一次需要它们的所有数据时，dto 也有帮助，或者它们甚至可以减少客户机和服务器之间的往返。

**使用 dto，我们可以从我们的领域模型**中构建不同的视图，允许我们创建相同领域的其他表示，但根据客户的需求优化它们，而不影响我们的领域设计。这种灵活性是解决复杂问题的有力工具。

## 5.用例

为了演示模式的实现，我们将使用一个简单的应用程序，它有两个主要的域模型，在本例中是`User`和`Role`。为了关注这个模式，让我们看两个功能性的例子——用户检索和新用户的创建。

### 5.1.DTO vs 域名

以下是两种模型的定义:

```java
public class User {

    private String id;
    private String name;
    private String password;
    private List<Role> roles;

    public User(String name, String password, List<Role> roles) {
        this.name = Objects.requireNonNull(name);
        this.password = this.encrypt(password);
        this.roles = Objects.requireNonNull(roles);
    }

    // Getters and Setters

   String encrypt(String password) {
       // encryption logic
   }
}
```

```java
public class Role {

    private String id;
    private String name;

    // Constructors, getters and setters
}
```

现在让我们看看 dto，这样我们就可以将它们与领域模型进行比较。

此时，需要注意的是，DTO 代表从 API 客户端发送或接收的模型。

因此，细微的区别是要么将发送到服务器的请求打包在一起，要么优化客户端的响应:

```java
public class UserDTO {
    private String name;
    private List<String> roles;

    // standard getters and setters
}
```

上面的 DTO 只向客户端提供相关信息，例如，出于安全原因，隐藏了密码。

下一个 DTO 对创建用户所需的所有数据进行分组，并在一个请求中将其发送到服务器，这优化了与 API 的交互:

```java
public class UserCreationDTO {

    private String name;
    private String password;
    private List<String> roles;

    // standard getters and setters
}
```

### 5.2.连接两边

接下来，连接两个类的层使用映射器组件将数据从一端传递到另一端，反之亦然。

这通常发生在表示层:

```java
@RestController
@RequestMapping("/users")
class UserController {

    private UserService userService;
    private RoleService roleService;
    private Mapper mapper;

    // Constructor

    @GetMapping
    @ResponseBody
    public List<UserDTO> getUsers() {
        return userService.getAll()
          .stream()
          .map(mapper::toDto)
          .collect(toList());
    }

    @PostMapping
    @ResponseBody
    public UserIdDTO create(@RequestBody UserCreationDTO userDTO) {
        User user = mapper.toUser(userDTO);

        userDTO.getRoles()
          .stream()
          .map(role -> roleService.getOrCreate(role))
          .forEach(user::addRole);

        userService.save(user);

        return new UserIdDTO(user.getId());
    }

} 
```

最后，我们有**传输数据的`Mapper`组件，确保 DTO 和域模型都不需要知道对方**:

```java
@Component
class Mapper {
    public UserDTO toDto(User user) {
        String name = user.getName();
        List<String> roles = user
          .getRoles()
          .stream()
          .map(Role::getName)
          .collect(toList());

        return new UserDTO(name, roles);
    }

    public User toUser(UserCreationDTO userDTO) {
        return new User(userDTO.getName(), userDTO.getPassword(), new ArrayList<>());
    }
}
```

## 6.常见错误

尽管 DTO 模式是一种简单的设计模式，但在应用程序实现这种技术时，我们可能会犯一些错误。

第一个错误是为每个场合创建不同的 dto。这将增加我们需要维护的类和映射器的数量。尽量保持简洁，并评估添加一个或重用现有组件的利弊。

我们还想避免在许多场景中使用单一的类。这种做法可能会导致大型合同中许多属性经常不被使用。

另一个常见的错误是将业务逻辑添加到那些类中，这是不应该发生的。该模式的目的是优化数据传输和合同结构。因此，所有的业务逻辑都应该位于域层。

最后，我们有所谓的**[local dto](https://web.archive.org/web/20220928221252/https://martinfowler.com/bliki/LocalDTO.html)，其中 dto 跨域传递数据。问题还是在于所有映射的维护成本。**

支持这种方法的一个最常见的论点是领域模型的封装。但是这里的问题是将我们的领域模型与持久性模型结合起来。通过分离它们，暴露领域模型的风险几乎消失了。

其他模式也有类似的结果，但它们通常用于更复杂的场景，如 [CQRS](https://web.archive.org/web/20220928221252/https://cqrs.files.wordpress.com/2010/11/cqrs_documents.pdf) 、[数据映射器](https://web.archive.org/web/20220928221252/https://martinfowler.com/eaaCatalog/dataMapper.html)、[命令查询分离](https://web.archive.org/web/20220928221252/https://martinfowler.com/bliki/CommandQuerySeparation.html)等。

## 7.结论

在本文中，我们看到了 [DTO 模式](https://web.archive.org/web/20220928221252/https://martinfowler.com/eaaCatalog/dataTransferObject.html)的定义，它为什么存在以及如何实现。

我们还看到了一些与其实现相关的常见错误以及避免这些错误的方法。

像往常一样，这个例子的源代码可以在 GitHub 的[上找到。](https://web.archive.org/web/20220928221252/https://github.com/eugenp/tutorials/tree/master/patterns-modules/design-patterns-architectural)