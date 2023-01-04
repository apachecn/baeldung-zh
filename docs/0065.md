# Mockito 和 JUnit 5–使用 ExtendWith

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/mockito-junit-5-extension>

## 1。概述

在这个快速教程中，我们将展示如何将 Mockito 与 JUnit 5 扩展模型集成。要了解更多关于 JUnit 5 扩展模型的信息，请看这篇[文章](/web/20221206183317/http://www.baeldung.com/junit-5-extensions)。

首先，我们将展示如何创建一个扩展，为任何用`@Mock`注释的类属性或方法参数自动创建模拟对象。

然后，我们将在 JUnit 5 测试类中使用我们的 Mockito 扩展。

## 2。Maven 依赖关系

### 2.1。所需的依赖关系

让我们将 JUnit 5 (jupiter)和`mockito`依赖项添加到我们的`pom.xml`中:

```
<dependency>
    <groupId>org.junit.jupiter</groupId>
    <artifactId>junit-jupiter-engine</artifactId>
    <version>5.8.1</version>
    <scope>test</scope>
</dependency>
<dependency>
    <groupId>org.mockito</groupId>
    <artifactId>mockito-core</artifactId>
    <version>4.6.1</version>
    <scope>test</scope>
</dependency>
```

### 2.2。Surefire 插件

让我们配置 Maven Surefire 插件，使用新的 JUnit 平台启动器运行我们的测试类:

```
<plugin>
    <artifactId>maven-surefire-plugin</artifactId>
    <version>2.22.2</version>
    <dependencies>
        <dependency>
             <groupId>org.junit.platform</groupId>
             <artifactId>junit-platform-surefire-provider</artifactId>
             <version>1.3.2</version>
         </dependency>
     </dependencies>
</plugin> 
```

最新版本的`[junit-jupiter-engine](https://web.archive.org/web/20221206183317/https://search.maven.org/search?q=junit-jupiter-engine)`、[、`mockito-core`、](https://web.archive.org/web/20221206183317/https://search.maven.org/classic/#search%7Cga%7C1%7Cmockito-core)、`[junit-platform-surefire-provider](https://web.archive.org/web/20221206183317/https://search.maven.org/search?q=a:junit-platform-surefire-provider)`可以从 Maven Central 下载。

## 3。摩奇托扩展

`Mockito`提供了库中 JUnit5 扩展的实现—`[mockito-junit-jupiter](https://web.archive.org/web/20221206183317/https://search.maven.org/search?q=a:mockito-junit-jupiter)`。

我们将把这种依赖性包含在我们的`pom.xml`中:

```
<dependency>
    <groupId>org.mockito</groupId>
    <artifactId>mockito-junit-jupiter</artifactId>
    <version>4.6.1</version>
    <scope>test</scope>
</dependency>
```

## 4。构建测试类

让我们构建我们的测试类，并将 Mockito 扩展附加到它:

```
@ExtendWith(MockitoExtension.class)
class UserServiceUnitTest {

    UserService userService;

    // ...
}
```

我们可以使用`@Mock`注释为一个实例变量注入一个 mock，我们可以在测试类的任何地方使用它:

```
@Mock UserRepository userRepository;
```

同样，我们可以将模拟对象注入到方法参数中:

```
@BeforeEach
void init(@Mock SettingRepository settingRepository) {
    userService = new DefaultUserService(userRepository, settingRepository, mailClient);

    Mockito.lenient().when(settingRepository.getUserMinAge()).thenReturn(10);

    when(settingRepository.getUserNameMinLength()).thenReturn(4);

    Mockito.lenient()
        .when(userRepository.isUsernameAlreadyExists(any(String.class)))
            .thenReturn(false);
}
```

请注意这里`Mockito.lenient()`的用法。当一个初始化的 mock 在执行过程中没有被一个测试方法调用时,`Mockito`抛出一个`UnsupportedStubbingException` 。通过在初始化模拟时使用这种方法，我们可以避免这种严格的存根检查。

我们甚至可以将模拟对象注入到测试方法参数中:

```
@Test
void givenValidUser_whenSaveUser_thenSucceed(@Mock MailClient mailClient) {
    // Given
    user = new User("Jerry", 12);
    when(userRepository.insert(any(User.class))).then(new Answer<User>() {
        int sequence = 1;

        @Override
        public User answer(InvocationOnMock invocation) throws Throwable {
            User user = (User) invocation.getArgument(0);
            user.setId(sequence++);
            return user;
        }
    });

    userService = new DefaultUserService(userRepository, settingRepository, mailClient);

    // When
    User insertedUser = userService.register(user);

    // Then
    verify(userRepository).insert(user);
    assertNotNull(user.getId());
    verify(mailClient).sendUserRegistrationMail(insertedUser);
}
```

注意，我们作为测试参数注入的`MailClient` mock 将不会是我们在`init`方法中注入的同一个实例。

## 5。结论

Junit 5 为扩展提供了一个很好的模型。我们演示了一个简单的 Mockito 扩展，它简化了我们的模拟创建逻辑。

本文中使用的所有代码都可以在我们的 [GitHub 项目](https://web.archive.org/web/20221206183317/https://github.com/eugenp/tutorials/tree/master/testing-modules/mockito-simple)中找到。