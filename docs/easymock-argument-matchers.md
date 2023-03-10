# EasyMock 参数匹配器

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/easymock-argument-matchers>

## 1。概述

在本教程中，我们将探索 EasyMock 参数匹配器。我们将讨论不同类型的预定义匹配器，以及如何创建自定义匹配器。

我们已经在 EasyMock 简介文章中介绍了 EasyMock 的基础知识，所以你可能需要先阅读它来熟悉 EasyMock。

## 2。简单的嘲讽例子

在我们开始探索不同的匹配器之前，让我们看一下我们的上下文。在本教程中，我们将在示例中使用一个非常基本的用户服务。

下面是我们简单的`IUserService`界面:

```java
public interface IUserService {
    public boolean addUser(User user);
    public List<User> findByEmail(String email);
    public List<User> findByAge(double age);  
}
```

以及相关的`User`型号:

```java
public class User {
    private long id;
    private String firstName;
    private String lastName;
    private double age;
    private String email;

    // standard constructor, getters, setters
}
```

因此，我们将从简单地模仿我们的`IUserService`开始，在我们的例子中使用它:

```java
private IUserService userService = mock(IUserService.class);
```

现在，让我们探索 EasyMock 参数匹配器。

## 3。相等匹配器

首先，我们将使用`eq()`匹配器来匹配新添加的`User`:

```java
@Test
public void givenUserService_whenAddNewUser_thenOK() {        
    expect(userService.addUser(eq(new User()))).andReturn(true);
    replay(userService);

    boolean result = userService.addUser(new User());
    verify(userService);
    assertTrue(result);
}
```

这个匹配器对原语和对象都可用，**对对象**使用`equals()`方法。

类似地，我们可以使用`same()`匹配器来匹配特定的`User`:

```java
@Test
public void givenUserService_whenAddSpecificUser_thenOK() {
    User user = new User();

    expect(userService.addUser(same(user))).andReturn(true);
    replay(userService);

    boolean result = userService.addUser(user);
    verify(userService);
    assertTrue(result);
}
```

**`same()`匹配器使用`==”`** 来比较参数，这意味着它在我们的例子中比较`User`实例。

如果我们不使用任何匹配器，默认情况下使用`equals().`来比较参数

对于数组，我们也有基于`Arrays.equals()`方法的`aryEq()`匹配器。

## 4。任何匹配器

有多个任意匹配器，如`anyInt()`、`anyBoolean()`、`anyDouble()`、…等。**它们指定参数应该具有给定的类型。**

让我们看一个使用`anyString()`来匹配期望的`email`为任何`String`值的例子:

```java
@Test
public void givenUserService_whenSearchForUserByEmail_thenFound() {
    expect(userService.findByEmail(anyString()))
      .andReturn(Collections.emptyList());
    replay(userService);

    List<User> result = userService.findByEmail("[[email protected]](/web/20220926185918/https://www.baeldung.com/cdn-cgi/l/email-protection)");
    verify(userService);
    assertEquals(0,result.size());
}
```

我们还可以使用`isA()`将一个参数匹配为一个特定类的实例:

```java
@Test
public void givenUserService_whenAddUser_thenOK() {
    expect(userService.addUser(isA(User.class))).andReturn(true);
    replay(userService);

    boolean result = userService.addUser(new User());
    verify(userService);
    assertTrue(result);
}
```

这里，我们断言我们期望`addUser()`方法参数的类型是`User.`

## 5\. Null Matchers

接下来，**我们可以使用`isNull()`和`notNull()`匹配器来匹配`null`值。**

在下面的例子中，如果添加的`User`值为空，我们将使用`isNull()`匹配器进行匹配:

```java
@Test
public void givenUserService_whenAddNull_thenFail() {
    expect(userService.addUser(isNull())).andReturn(false);
    replay(userService);

    boolean result = userService.addUser(null);
    verify(userService);
    assertFalse(result);
}
```

我们还可以用类似的方式`notNull()`来匹配添加的用户值是否不为空:

```java
@Test
public void givenUserService_whenAddNotNull_thenOK() {
    expect(userService.addUser(notNull())).andReturn(true);
    replay(userService);

    boolean result = userService.addUser(new User());
    verify(userService);
    assertTrue(result);
}
```

## 6。字符串匹配器

有多个有用的匹配器可以和`String`参数一起使用。

首先，我们将使用`startsWith()`匹配器来匹配用户的电子邮件前缀:

```java
@Test
public void whenSearchForUserByEmailStartsWith_thenFound() {        
    expect(userService.findByEmail(startsWith("test")))
      .andReturn(Collections.emptyList());
    replay(userService);

    List<User> result = userService.findByEmail("[[email protected]](/web/20220926185918/https://www.baeldung.com/cdn-cgi/l/email-protection)");
    verify(userService);
    assertEquals(0,result.size());
}
```

类似地，我们将使用`endsWith()`匹配器作为电子邮件后缀:

```java
@Test
public void givenUserService_whenSearchForUserByEmailEndsWith_thenFound() {        
    expect(userService.findByEmail(endsWith(".com")))
      .andReturn(Collections.emptyList());
    replay(userService);

    List<User> result = userService.findByEmail("[[email protected]](/web/20220926185918/https://www.baeldung.com/cdn-cgi/l/email-protection)");
    verify(userService);
    assertEquals(0,result.size());
}
```

更一般地，我们可以使用`contains()`将电子邮件与给定的子字符串进行匹配:

```java
@Test
public void givenUserService_whenSearchForUserByEmailContains_thenFound() {        
    expect(userService.findByEmail(contains("@")))
      .andReturn(Collections.emptyList());
    replay(userService);

    List<User> result = userService.findByEmail("[[email protected]](/web/20220926185918/https://www.baeldung.com/cdn-cgi/l/email-protection)");
    verify(userService);
    assertEquals(0,result.size());
}
```

或者甚至使用`matches()`将我们的电子邮件匹配到特定的正则表达式:

```java
@Test
public void givenUserService_whenSearchForUserByEmailMatches_thenFound() {        
    expect(userService.findByEmail(matches(".+\\@.+\\..+")))
      .andReturn(Collections.emptyList());
    replay(userService);

    List<User> result = userService.findByEmail("[[email protected]](/web/20220926185918/https://www.baeldung.com/cdn-cgi/l/email-protection)");
    verify(userService);
    assertEquals(0,result.size());
}
```

## 7。数字匹配器

我们也有一些可以使用的数值匹配器。

让我们看一个使用`lt()`匹配器来匹配小于 100 的年龄参数的例子:

```java
@Test
public void givenUserService_whenSearchForUserByAgeLessThan_thenFound() {    
    expect(userService.findByAge(lt(100.0)))
      .andReturn(Collections.emptyList());
    replay(userService);

    List<User> result = userService.findByAge(20);        
    verify(userService);
    assertEquals(0,result.size());
}
```

类似地，我们也使用`geq()`来匹配年龄参数，使其大于或等于 10:

```java
@Test
public void givenUserService_whenSearchForUserByAgeGreaterThan_thenFound() {    
    expect(userService.findByAge(geq(10.0)))
      .andReturn(Collections.emptyList());
    replay(userService);

    List<User> result = userService.findByAge(20);        
    verify(userService);
    assertEquals(0,result.size());
}
```

可用的号码匹配器有:

*   `lt()`–小于给定值
*   `leq()`–小于或等于
*   `gt()`–大于
*   `geq()`–大于或等于

## 8。联合收割机匹配器

**我们还可以使用`and()`、`or()`和`not()`匹配器组合多个匹配器。**

让我们看看如何组合两个匹配器来验证年龄值既大于 10 又小于 100:

```java
@Test
public void givenUserService_whenSearchForUserByAgeRange_thenFound() {
    expect(userService.findByAge(and(gt(10.0),lt(100.0))))
      .andReturn(Collections.emptyList());
    replay(userService);

    List<User> result = userService.findByAge(20);        
    verify(userService);
    assertEquals(0,result.size());
}
```

我们可以看的另一个例子是将`not()`和`endsWith()`结合起来匹配不以“.”结尾的电子邮件。com”:

```java
@Test
public void givenUserService_whenSearchForUserByEmailNotEndsWith_thenFound() {
    expect(userService.findByEmail(not(endsWith(".com"))))
      .andReturn(Collections.emptyList());
    replay(userService);

    List<User> result = userService.findByEmail("[[email protected]](/web/20220926185918/https://www.baeldung.com/cdn-cgi/l/email-protection)");
    verify(userService);
    assertEquals(0,result.size());
}
```

## 9。自定义匹配器

最后，我们将讨论如何创建一个定制的 EasyMock 匹配器。

目标是创建一个简单的`minCharCount()`匹配器来匹配长度大于或等于给定值的字符串:

```java
@Test
public void givenUserService_whenSearchForUserByEmailCharCount_thenFound() {        
    expect(userService.findByEmail(minCharCount(5)))
      .andReturn(Collections.emptyList());
    replay(userService);

    List<User> result = userService.findByEmail("[[email protected]](/web/20220926185918/https://www.baeldung.com/cdn-cgi/l/email-protection)");
    verify(userService);
    assertEquals(0,result.size());
}
```

要创建自定义参数匹配器，我们需要:

*   创建一个实现`IArgumentMatcher`接口的新类
*   使用新的匹配器名称创建一个静态方法，并使用`reportMatcher()`注册上述类的一个实例

让我们看看在我们的`minCharCount()`方法中声明匿名类的两个步骤:

```java
public static String minCharCount(int value){
    EasyMock.reportMatcher(new IArgumentMatcher() {
        @Override
        public boolean matches(Object argument) {
            return argument instanceof String 
              && ((String) argument).length() >= value;
        }

        @Override
        public void appendTo(StringBuffer buffer) {
            buffer.append("charCount(\"" + value + "\")");
        }
    });    
    return null;
}
```

另外，请注意，`IArgumentMatcher`接口有两个方法:`matches()`和`appendTo(). `

第一个方法包含我们的匹配器的参数验证和逻辑，而第二个方法用于追加匹配器`String`表示，以便在失败时打印。

## 10。结论

我们讨论了不同数据类型的 EasyMock 预定义参数匹配器，以及如何创建自定义匹配器。

GitHub 上的[提供了示例的完整源代码。](https://web.archive.org/web/20220926185918/https://github.com/eugenp/tutorials/tree/master/testing-modules/mocks)