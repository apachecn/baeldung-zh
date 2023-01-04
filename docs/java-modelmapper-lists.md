# 使用模型映射器映射列表

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-modelmapper-lists>

## 1.概观

在本教程中，我们将解释如何使用 [ModelMapper](https://web.archive.org/web/20221208143854/http://modelmapper.org/getting-started/) 框架来映射不同元素类型的列表。 **T his 涉及使用 Java 中的泛型类型作为解决方案，将不同类型的数据从一个列表转换到另一个列表** 。

## 2.模型映射器

ModelMapper 的主要作用是通过确定如何将一个对象模型映射到另一个称为数据转换对象(d to)的对象模型来映射对象。

为了使用[模型映射器](https://web.archive.org/web/20221208143854/https://search.maven.org/artifact/org.modelmapper/modelmapper)，我们首先将依赖项添加到我们的`pom.xml`:

```
<dependency> 
    <groupId>org.modelmapper</groupId>
    <artifactId>modelmapper</artifactId>
    <version>2.3.7</version>
</dependency>
```

### 2.1.配置

模型映射器提供了多种配置来简化映射过程。 我们通过启用或禁用配置 中的适当属性来自定义配置。**将`fieldMatchingEnabled`属性设置为` true`并允许私有字段匹配** : 是一种常见的做法

```
modelMapper.getConfiguration()
  .setFieldMatchingEnabled(true)
  .setFieldAccessLevel(Configuration.AccessLevel.PRIVATE); 
```

通过这样做，ModelMapper 可以比较映射类(对象)中的私有字段。在这种配置中，并不严格要求两个类中的所有字段都具有相同的名称。 允许几种[匹配策略](https://web.archive.org/web/20221208143854/http://modelmapper.org/user-manual/configuration/)。**默认情况下，标准匹配策略要求所有源和目标属性必须以任意顺序匹配。这是我们场景**的理想选择。

### 2.2.类型令牌

ModelMapper 使用 [TypeToken](https://web.archive.org/web/20221208143854/http://modelmapper.org/javadoc/org/modelmapper/TypeToken.html) 来映射泛型类型。为了理解为什么这是必要的，让我们看看当我们将一个`Integer`列表映射到一个`Character`列表:时会发生什么

```
List<Integer> integers = new ArrayList<Integer>();
integers.add(1);
integers.add(2);
integers.add(3);

List<Character> characters = new ArrayList<Character>();
modelMapper.map(integers, characters);
```

此外，如果我们打印出`characters`列表的元素，我们会看到一个空列表。这是由于在运行时执行期间发生类型擦除。

如果我们将`map`调用改为使用`TypeToken`，我们可以为`List<Character>` : 创建一个类型文字

```
List<Character> characters 
    = modelMapper.map(integers, new TypeToken<List<Character>>() {}.getType());
```

**在编译时，`TokenType` 匿名内例保留了`List<Character>`** 参数类型，这次我们的转换成功了。

## 3.使用自定义类型映射

Java 中的列表可以使用自定义元素类型进行映射。

例如，假设我们想要将一个由`User`实体组成的列表映射到一个列表`UserDTO`。为了实现这一点，我们将为每个元素调用`map`:

```
List<UserDTO> dtos = users
  .stream()
  .map(user -> modelMapper.map(user, UserDTO.class))
  .collect(Collectors.toList());
```

当然，通过更多的工作，我们可以创建一个通用的参数化方法:

```
<S, T> List<T> mapList(List<S> source, Class<T> targetClass) {
    return source
      .stream()
      .map(element -> modelMapper.map(element, targetClass))
      .collect(Collectors.toList());
}
```

因此，我们可以改为做:

```
List<UserDTO> userDtoList = mapList(users, UserDTO.class);
```

## 4.类型映射和属性映射

列表或集合等特定属性可以添加到`User-UserDTO`模型中。`[TypeMap](https://web.archive.org/web/20221208143854/http://modelmapper.org/javadoc/org/modelmapper/TypeMap.html)` 提供了明确定义这些属性映射的方法。`TypeMap`对象存储特定类型(类)的映射信息:

```
TypeMap<UserList, UserListDTO> typeMap = modelMapper.createTypeMap(UserList.class, UserListDTO.class);
```

`UserList` 类包含一个`User` s. **的集合在这里，w** **e 想把这个集合中的用户名列表映射到`UserListDTO`类**的属性列表 **。为此，** 我们将创建第一个`UsersListConverter`类，并将其`List <User>`和`List <String>` 作为 参数类型传递给

```
public class UsersListConverter extends AbstractConverter<List<User>, List<String>> {

    @Override
    protected List<String> convert(List<User> users) {

        return users
          .stream()
          .map(User::getUsername)
          .collect(Collectors.toList());
    }
}
```

从创建的`TypeMap`对象我们 通过调用`UsersListConverter`类的实例显式添加[属性映射](https://web.archive.org/web/20221208143854/http://modelmapper.org/user-manual/property-mapping/)

```
 typeMap.addMappings(mapper -> mapper.using(new UsersListConverter())
   .map(UserList::getUsers, UserListDTO::setUsernames));
```

在`addMappings`方法中，表达式映射允许我们用 lambda 表达式定义源到目的地的属性。最后，它将用户列表转换成用户名列表。

## 5.结论

在本教程中，我们解释了如何通过在 ModelMapper `.` 中操作泛型类型来映射列表。我们可以利用`TypeToken,` 泛型类型映射和属性映射来创建对象列表类型并进行复杂映射。

本文的完整源代码可以在 GitHub 上的[处获得。](https://web.archive.org/web/20221208143854/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-collections-conversions-2)