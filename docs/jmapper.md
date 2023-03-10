# JMapper 指南

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/jmapper>

## 1。概述

在本教程中，我们将探索一个快速易用的映射框架。

我们将讨论配置 JMapper 的不同方法，如何执行定制转换，以及关系映射。

## 2。Maven 配置

首先，我们需要将 [JMapper 依赖关系](https://web.archive.org/web/20221127215441/https://search.maven.org/classic/#search%7Cga%7C1%7Cjmapper-core)添加到我们的`pom.xml`:

```java
<dependency>
    <groupId>com.googlecode.jmapper-framework</groupId>
    <artifactId>jmapper-core</artifactId>
    <version>1.6.0.1</version>
</dependency>
```

## 3。源模型和目的模型

在我们开始配置之前，让我们看一下我们将在本教程中使用的简单 beans。

首先，这是我们的源 bean——一个基本的`User`:

```java
public class User {
    private long id;    
    private String email;
    private LocalDate birthDate;
}
```

还有我们的目的地比恩，`UserDto:`

```java
public class UserDto {
    private long id;
    private String username;
}
```

**我们将使用这个库将属性从我们的源 bean `User`映射到我们的目的 bean `UserDto`。**

配置 JMapper 有三种方式:通过使用 API、注释和 XML 配置。

在接下来的几节中，我们将逐一介绍这些内容。

## 4。使用 API

让我们看看如何使用 API 配置`JMapper`。

这里，我们不需要向源类和目的类添加任何配置。相反，所有的**配置都可以使用** `**JMapperAPI**,`来完成，这使得它成为最灵活的配置方法:

```java
@Test
public void givenUser_whenUseApi_thenConverted(){
    JMapperAPI jmapperApi = new JMapperAPI() 
      .add(mappedClass(UserDto.class)
        .add(attribute("id").value("id"))
        .add(attribute("username").value("email")));

    JMapper<UserDto, User> userMapper = new JMapper<>
      (UserDto.class, User.class, jmapperApi);
    User user = new User(1L,"[[email protected]](/web/20221127215441/https://www.baeldung.com/cdn-cgi/l/email-protection)", LocalDate.of(1980,8,20));
    UserDto result = userMapper.getDestination(user);

    assertEquals(user.getId(), result.getId());
    assertEquals(user.getEmail(), result.getUsername());
}
```

这里，我们使用`mappedClass()`方法来定义我们的映射类`UserDto.`，然后，我们使用`attribute()`方法来定义每个属性及其映射值。

接下来，我们基于配置创建了一个`JMapper`对象，并使用它的`getDestination()`方法来获得`UserDto`结果。

## 5。使用注释

让我们看看**如何使用`@JMap`注释来配置我们的映射**:

```java
public class UserDto {  
    @JMap
    private long id;

    @JMap("email")
    private String username;
}
```

下面是我们如何使用我们的`JMapper`:

```java
@Test
public void givenUser_whenUseAnnotation_thenConverted(){
    JMapper<UserDto, User> userMapper = new JMapper<>(UserDto.class, User.class);
    User user = new User(1L,"[[email protected]](/web/20221127215441/https://www.baeldung.com/cdn-cgi/l/email-protection)", LocalDate.of(1980,8,20));
    UserDto result = userMapper.getDestination(user);

    assertEquals(user.getId(), result.getId());
    assertEquals(user.getEmail(), result.getUsername());        
}
```

注意，对于`id`属性，我们不需要提供目标字段名，因为它与源 bean 同名，而对于`username`字段，我们提到它对应于`User`类中的`email`字段。

然后，**我们只需要将源 beanss 和目的 bean 传递给我们的`JMapper`**——不需要进一步的配置。

总的来说，这种方法很方便，因为它使用的代码最少。

## 6。使用 XML 配置

我们还可以使用 XML 配置来定义我们的映射。

下面是我们在`user_jmapper.xml`的示例 XML 配置:

```java
<jmapper>
  <class name="com.baeldung.jmapper.UserDto">
    <attribute name="id">
      <value name="id"/>
    </attribute>
    <attribute name="username">
      <value name="email"/>
    </attribute>
  </class>
</jmapper>
```

我们需要将我们的 XML 配置传递给`JMapper`:

```java
@Test
public void givenUser_whenUseXml_thenConverted(){
    JMapper<UserDto, User> userMapper = new JMapper<>
      (UserDto.class, User.class,"user_jmapper.xml");
    User user = new User(1L,"[[email protected]](/web/20221127215441/https://www.baeldung.com/cdn-cgi/l/email-protection)", LocalDate.of(1980,8,20));
    UserDto result = userMapper.getDestination(user);

    assertEquals(user.getId(), result.getId());
    assertEquals(user.getEmail(), result.getUsername());            
}
```

我们也可以将 XML 配置作为一个`String`直接传递给`JMapper`，而不是一个文件名。

## 7。全球地图

如果在源 beanss 和目标 bean 中有多个同名字段，我们可以利用全局映射。

例如，如果我们有一个包含两个字段`id`和`email`的`UserDto1`:

```java
public class UserDto1 {  
    private long id;
    private String email;

    // standard constructor, getters, setters
}
```

全局映射将更容易使用，因为它们被映射到在`User`源 bean 中具有相同名称的字段。

### 7.1。使用 API

对于`JMapperAPI`配置，我们将使用`global()`:

```java
@Test
public void givenUser_whenUseApiGlobal_thenConverted() {
    JMapperAPI jmapperApi = new JMapperAPI()
      .add(mappedClass(UserDto.class).add(global())) ;
    JMapper<UserDto1, User> userMapper1 = new JMapper<>
      (UserDto1.class, User.class,jmapperApi);
    User user = new User(1L,"[[email protected]](/web/20221127215441/https://www.baeldung.com/cdn-cgi/l/email-protection)", LocalDate.of(1980,8,20));
    UserDto1 result = userMapper1.getDestination(user);

    assertEquals(user.getId(), result.getId());
    assertEquals(user.getEmail(), result.getEmail());
}
```

### 7.2。使用注释

对于注释配置，我们将在类级别使用`@JGlobalMap` :

```java
@JGlobalMap
public class UserDto1 {  
    private long id;
    private String email;
}
```

这里有一个简单的测试:

```java
@Test
public void whenUseGlobalMapAnnotation_thenConverted(){
    JMapper<UserDto1, User> userMapper= new JMapper<>(
      UserDto1.class, User.class);
    User user = new User(
      1L,"[[email protected]](/web/20221127215441/https://www.baeldung.com/cdn-cgi/l/email-protection)", LocalDate.of(1980,8,20));
    UserDto1 result = userMapper.getDestination(user);

    assertEquals(user.getId(), result.getId());
    assertEquals(user.getEmail(), result.getEmail());        
}
```

### 7.3。XML 配置

对于 XML 配置，我们有`<global/>`元素:

```java
<jmapper>
  <class name="com.baeldung.jmapper.UserDto1">
    <global/>
  </class>
</jmapper>
```

然后传递 XML 文件名:

```java
@Test
public void givenUser_whenUseXmlGlobal_thenConverted(){
    JMapper<UserDto1, User> userMapper = new JMapper<>
      (UserDto1.class, User.class,"user_jmapper1.xml");
    User user = new User(1L,"[[email protected]](/web/20221127215441/https://www.baeldung.com/cdn-cgi/l/email-protection)", LocalDate.of(1980,8,20));
    UserDto1 result = userMapper.getDestination(user);

    assertEquals(user.getId(), result.getId());
    assertEquals(user.getEmail(), result.getEmail());            
}
```

## 8。自定义转换

现在，让我们看看如何使用`JMapper`应用自定义转换。

在我们的`UserDto`中有一个新字段`age`，我们需要从`User` `birthDate`属性中计算它:

```java
public class UserDto {
    @JMap
    private long id;

    @JMap("email")
    private String username;

    @JMap("birthDate")
    private int age;

    @JMapConversion(from={"birthDate"}, to={"age"})
    public int conversion(LocalDate birthDate){
        return Period.between(birthDate, LocalDate.now())
          .getYears();
    }
}
```

因此，**我们使用`@JMapConversion`来应用从`User's birthDate`到`UserDto's` `age`属性的复杂转换**。因此，当我们将`User`映射到`UserDto`时，将计算`age`字段:

```java
@Test
public void whenUseAnnotationExplicitConversion_thenConverted(){
    JMapper<UserDto, User> userMapper = new JMapper<>(
      UserDto.class, User.class);
    User user = new User(
      1L,"[[email protected]](/web/20221127215441/https://www.baeldung.com/cdn-cgi/l/email-protection)", LocalDate.of(1980,8,20));
    UserDto result = userMapper.getDestination(user);

    assertEquals(user.getId(), result.getId());
    assertEquals(user.getEmail(), result.getUsername());     
    assertTrue(result.getAge() > 0);
}
```

## 9。关系映射

最后，我们将讨论关系映射。使用这个方法，我们每次都需要使用一个目标类来定义我们的`JMapper`。

如果我们已经知道目标类，我们可以为每个映射字段定义它们，并使用`RelationalJMapper`。

在这个例子中，我们有一个源 bean `User`:

```java
public class User {
    private long id;    
    private String email;
}
```

和两个目的地 bean`UserDto1`:

```java
public class UserDto1 {  
    private long id;
    private String username;
}
```

和`UserDto2`:

```java
public class UserDto2 {
    private long id;
    private String email;
}
```

让我们看看如何利用我们的`RelationalJMapper.`

### 9.1。使用 API

对于我们的 API 配置，我们可以使用`targetClasses()`为每个属性定义目标类:

```java
@Test
public void givenUser_whenUseApi_thenConverted(){
    JMapperAPI jmapperApi = new JMapperAPI()
      .add(mappedClass(User.class)
      .add(attribute("id")
        .value("id")
        .targetClasses(UserDto1.class,UserDto2.class))
      .add(attribute("email")
        .targetAttributes("username","email")
        .targetClasses(UserDto1.class,UserDto2.class)));

    RelationalJMapper<User> relationalMapper = new RelationalJMapper<>
      (User.class,jmapperApi);
    User user = new User(1L,"[[email protected]](/web/20221127215441/https://www.baeldung.com/cdn-cgi/l/email-protection)");
    UserDto1 result1 = relationalMapper
      .oneToMany(UserDto1.class, user);
    UserDto2 result2 = relationalMapper
      .oneToMany(UserDto2.class, user);

    assertEquals(user.getId(), result1.getId());
    assertEquals(user.getEmail(), result1.getUsername());
    assertEquals(user.getId(), result2.getId());
    assertEquals(user.getEmail(), result2.getEmail());            
}
```

注意，对于每个目标类，我们需要定义目标属性名。

`RelationalJMapper`只接受一个类——映射类。

### 9.2。使用注释

对于注释方法，我们也将定义`classes`:

```java
public class User {
    @JMap(classes = {UserDto1.class, UserDto2.class})
    private long id;    

    @JMap(
      attributes = {"username", "email"}, 
      classes = {UserDto1.class, UserDto2.class})
    private String email;
}
```

像往常一样，当我们使用注释时，不需要进一步的配置:

```java
@Test
public void givenUser_whenUseAnnotation_thenConverted(){
    RelationalJMapper<User> relationalMapper
      = new RelationalJMapper<>(User.class);
    User user = new User(1L,"[[email protected]](/web/20221127215441/https://www.baeldung.com/cdn-cgi/l/email-protection)");
    UserDto1 result1 = relationalMapper
      .oneToMany(UserDto1.class, user);
    UserDto2 result2= relationalMapper
      .oneToMany(UserDto2.class, user);

    assertEquals(user.getId(), result1.getId());
    assertEquals(user.getEmail(), result1.getUsername());  
    assertEquals(user.getId(), result2.getId());
    assertEquals(user.getEmail(), result2.getEmail());          
}
```

### 9.3。XML 配置

对于 XML 配置，我们使用`<classes>`来定义每个属性的目标类。

这是我们的`user_jmapper2.xml`:

```java
<jmapper>
  <class name="com.baeldung.jmapper.relational.User">
    <attribute name="id">
      <value name="id"/>
      <classes>
        <class name="com.baeldung.jmapper.relational.UserDto1"/>
        <class name="com.baeldung.jmapper.relational.UserDto2"/>
      </classes>
    </attribute>
    <attribute name="email">
      <attributes>
        <attribute name="username"/>
        <attribute name="email"/>
      </attributes>
      <classes>
        <class name="com.baeldung.jmapper.relational.UserDto1"/>
        <class name="com.baeldung.jmapper.relational.UserDto2"/>
      </classes>      
    </attribute>
  </class>
</jmapper>
```

然后将 XML 配置文件传递给`RelationalJMapper`:

```java
@Test
public void givenUser_whenUseXml_thenConverted(){
    RelationalJMapper<User> relationalMapper
     = new RelationalJMapper<>(User.class,"user_jmapper2.xml");
    User user = new User(1L,"[[email protected]](/web/20221127215441/https://www.baeldung.com/cdn-cgi/l/email-protection)");
    UserDto1 result1 = relationalMapper
      .oneToMany(UserDto1.class, user);
    UserDto2 result2 = relationalMapper
      .oneToMany(UserDto2.class, user);

    assertEquals(user.getId(), result1.getId());
    assertEquals(user.getEmail(), result1.getUsername());
    assertEquals(user.getId(), result2.getId());
    assertEquals(user.getEmail(), result2.getEmail());         
}
```

## 10。结论

在本教程中，我们学习了配置 JMapper 的不同方法以及如何执行自定义转换。

示例的完整源代码可以在 GitHub 的[中找到。](https://web.archive.org/web/20221127215441/https://github.com/eugenp/tutorials/tree/master/libraries-data)