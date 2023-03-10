# @DirtiesContext 快速指南

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-dirtiescontext>

## 1.概观

在这个快速教程中，我们将学习`[@DirtiesContext](https://web.archive.org/web/20221217195746/https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/test/annotation/DirtiesContext.html)`注释。我们还将展示使用注释进行测试的标准方法。

## 2.`@DirtiesContext`

`@DirtiesContext`是一个**弹簧检测标注**。它表明相关的测试或类修改了`ApplicationContext`。它告诉测试框架关闭并为以后的测试重新创建上下文。

我们可以注释一个测试方法或者整个类。通过设置`MethodMode`或`ClassMode`，**我们可以控制 Spring 何时标记关闭**的上下文。

如果我们将`@DirtiesContext`放在一个类上，那么这个注释会应用到这个类中具有给定`ClassMode.` 的每个方法上

## 3.不清除 Spring 上下文的测试

假设我们有一个`User`:

```java
public class User {
    String firstName;
    String lastName;
}
```

我们还有一个非常简单的`UserCache:`

```java
@Component
public class UserCache {

    @Getter
    private Set<String> userList = new HashSet<>();

    public boolean addUser(String user) {
        return userList.add(user);
    }

    public void printUserList(String message) {
        System.out.println(message + ": " + userList);
    }

}
```

我们创建一个集成测试来加载和测试整个应用程序:

```java
@TestMethodOrder(OrderAnnotation.class)
@ExtendWith(SpringExtension.class)
@SpringBootTest(classes = SpringDataRestApplication.class)
class DirtiesContextIntegrationTest {

    @Autowired
    protected UserCache userCache;

    ...
}
```

第一种方法`addJaneDoeAndPrintCache`，向缓存中添加一个条目:

```java
@Test
@Order(1)
void addJaneDoeAndPrintCache() {
    userCache.addUser("Jane Doe");
    userCache.printUserList("addJaneDoeAndPrintCache");
}
```

将用户添加到缓存后，它会打印缓存的内容:

```java
addJaneDoeAndPrintCache: [Jane Doe]
```

接下来，`printCache`再次打印用户缓存:

```java
@Test
@Order(2)
void printCache() {
    userCache.printUserList("printCache");
}
```

它包含在之前的测试中添加的名称:

```java
printCache: [Jane Doe]
```

假设后来的一个测试依赖于一个空的缓存来存放一些断言。先前插入的名称可能会导致不期望的行为。

## 4.使用`@DirtiesContext`

现在我们用默认的`MethodMode`、`[AFTER_METHOD](https://web.archive.org/web/20221217195746/https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/test/annotation/DirtiesContext.MethodMode.html#AFTER_METHOD)`来显示`@DirtiesContext`。这意味着在相应的测试方法完成后，Spring 将标记上下文以进行闭包。

为了隔离测试的变更，我们添加了`@DirtiesContext`。让我们看看它是如何工作的。

`addJohnDoeAndPrintCache`测试方法将用户添加到缓存中。我们还添加了`@DirtiesContext`注释，它表示上下文应该在测试方法结束时关闭:

```java
@DirtiesContext(methodMode = MethodMode.AFTER_METHOD)
@Test
@Order(3)
void addJohnDoeAndPrintCache() {
    userCache.addUser("John Doe");
    userCache.printUserList("addJohnDoeAndPrintCache");
}
```

现在的输出是:

```java
addJohnDoeAndPrintCache: [John Doe, Jane Doe]
```

最后，` printCacheAgain`再次打印缓存:

```java
@Test
@Order(4)
void printCacheAgain() {
    userCache.printUserList("printCacheAgain");
}
```

运行完整的测试类，我们看到 Spring 上下文在`addJohnDoeAndPrintCache`和`printCacheAgain`之间重新加载。因此缓存重新初始化，输出为空:

```java
printCacheAgain: []
```

## 5.其他支持的测试阶段

上例显示了电流测试方法阶段后的**。让我们快速总结一下各个阶段:**

### 5.1.班级水平

**测试类的`ClassMode` 选项定义了上下文何时被重置**:

*   `BEFORE_CLASS:` 当前测试类之前
*   `BEFORE_EACH_TEST_METHOD:` 当前测试类中的每个测试方法之前
*   `AFTER_EACH_TEST_METHOD:` 当前测试类中的每个测试方法后
*   `AFTER_CLASS:` 当前测试类后

### 5.2.方法级别

**单个方法的`MethodMode`选项定义了上下文重置的时间**:

*   `BEFORE_METHOD:` 当前测试方法之前
*   `AFTER_METHOD`:当前测试方法后

## 6.结论

在本文中，我们介绍了`@DirtiesContext`测试注释。

与往常一样，GitHub 上的示例代码[可用。](https://web.archive.org/web/20221217195746/https://github.com/eugenp/tutorials/tree/master/testing-modules/spring-testing)