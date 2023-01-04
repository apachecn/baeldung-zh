# 番石榴 18:有什么新鲜事？

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/whats-new-in-guava-18>

## 1。概述

Google Guava 为库提供了简化 Java 开发的工具。在本教程中，我们将看看在[番石榴 18 版本](https://web.archive.org/web/20221208143856/https://github.com/google/guava/wiki/Release18)中引入的新功能。

## 2。`MoreObjects`公用事业类

Guava 18 增加了`MoreObjects`类，它包含了在`java.util.Objects`中没有对等物的方法。

从版本 18 开始，它只包含了`toStringHelper`方法的实现，可以用来帮助您构建自己的`toString`方法。

*   `toStringHelper(Class<?> clazz)`
*   `toStringHelper(Object self)`
*   `toStringHelper(String className)`

通常， `toString()`在你需要输出一个对象的一些信息时使用`.` 通常它应该包含关于对象当前状态的细节。通过使用`toStringHelper`的一个实现，您可以轻松地构建一个有用的`toString()`消息。

假设我们有一个`User` 对象，它包含一些需要在调用`toString()`时写入的字段。我们可以使用`MoreObjects.toStringHelper(Object self)` 方法轻松做到这一点。

```
public class User {

    private long id;
    private String name;

    public User(long id, String name) {
        this.id = id;
        this.name = name;
    }

    @Override
    public String toString() {
        return MoreObjects.toStringHelper(this)
            .add("id", id)
            .add("name", name)
            .toString();
    }
} 
```

这里我们使用了`toStringHelper(Object self)`方法。通过这个设置，我们可以创建一个示例用户来查看调用`toString()`时的输出。

```
User user = new User(12L, "John Doe");
String userState = user.toString();
// userState: User{ id=12,name=John Doe } 
```

如果配置类似，其他两个实现将返回相同的`String`:

**T2`toStringHelper(Class<?> clazz)`**

```
@Override
public String toString() {
    return MoreObjects.toStringHelper(User.class)
        .add("id", id)
        .add("name", name)
        .toString();
}
```

**T2`toStringHelper(String className)`**

```
@Override
public String toString() {
    return MoreObjects.toStringHelper("User")
        .add("id", id)
        .add("name", name)
        .toString();
}
```

如果您在`User`类的扩展上调用`toString()`,这些方法之间的差异是显而易见的。例如，如果你有两种`User`:`Administrator`和`Player`，它们会产生不同的输出。

```
public class Player extends User {
    public Player(long id, String name) {
        super(id, name);
    }
}

public class Administrator extends User {
    public Administrator(long id, String name) {
        super(id, name);
    }
}
```

如果你在 `User` 类中使用`toStringHelper(Object self)` ，那么你的`Player.toString()` 将返回“`Player{id=12, name=John Doe}`”。但是，如果使用`toStringHelper(String className)` 或`toStringHelper(Class<?> clazz)`， `Player.toString()` 将返回`User{id=12, name=John Doe}`。列出的类名将是父类而不是子类。

## 3。`FluentIterable`新方法

### 3.1。概述

`FluentIterable`用于以链式方式操作`Iterable`实例。让我们看看如何使用它。

假设您有一个在上面的例子中定义的`User`对象的列表，并且您希望过滤该列表以仅包括 18 岁或以上的用户。

```
List<User> users = new ArrayList<>();
users.add(new User(1L, "John", 45));
users.add(new User(2L, "Michelle", 27));
users.add(new User(3L, "Max", 16));
users.add(new User(4L, "Sue", 10));
users.add(new User(5L, "Bill", 65));

Predicate<User> byAge = user -> user.getAge() >= 18;

List<String> results = FluentIterable.from(users)
                           .filter(byAge)
                           .transform(Functions.toStringFunction())
                           .toList(); 
```

结果列表将包含 John、Michelle 和 Bill 的信息。

### 3.2。 `FluentIterable.of(E[])`

用这种方法。您可以从数组`Object`中创建一个`FluentIterable`。

```
User[] usersArray = { new User(1L, "John", 45), new User(2L, "Max", 15) } ;
FluentIterable<User> users = FluentIterable.of(usersArray); 
```

您现在可以使用`FluentIterable`界面中提供的方法。

### 3.3。 `FluentIterable.append(E…)`

您可以通过向现有的`FluentIterable`添加更多元素来创建新的`FluentIterable`。

```
User[] usersArray = {new User(1L, "John", 45), new User(2L, "Max", 15)};

FluentIterable<User> users = FluentIterable.of(usersArray).append(
                                 new User(3L, "Sue", 23),
                                 new User(4L, "Bill", 17)
                             ); 
```

正如所料，结果`FluentIterable`的大小是 4。

### 3.4。 `FluentIterable.append(Iterable<? extends E>)`

这个方法的行为与前面的例子相同，但是允许您将任何现有的`Iterable`实现的全部内容添加到一个`FluentIterable`中。

```
User[] usersArray = { new User(1L, "John", 45), new User(2L, "Max", 15) };

List<User> usersList = new ArrayList<>();
usersList.add(new User(3L, "Diana", 32));

FluentIterable<User> users = FluentIterable.of(usersArray).append(usersList); 
```

正如所料，结果`FluentIterable`的大小是 3。

### 3.5。 `FluentIterable.join(Joiner)`

`FluentIterable.join(…)` 方法产生一个表示`FluentIterable`全部内容的`String`，由一个给定的`String`连接。

```
User[] usersArray = { new User(1L, "John", 45), new User(2L, "Max", 15) };
FluentIterable<User> users = FluentIterable.of(usersArray);
String usersString = users.join(Joiner.on("; ")); 
```

`usersString`变量将包含对`FluentIterable`的每个元素调用`toString()` 方法的输出，用“；”分隔。`Joiner`类提供了几个连接字符串的选项。

## 4。`Hashing.crc32c`

散列函数是可用于将任意大小的数据映射到固定大小的数据的任何函数。它被用于许多领域，如加密和检查传输数据中的错误。

`Hashing.crc32c`方法返回实现 [CRC32C 算法](https://web.archive.org/web/20221208143856/https://en.wikipedia.org/wiki/Cyclic_redundancy_check)的`HashFunction`。

```
int receivedData = 123;
HashCode hashCode = Hashing.crc32c().hashInt(receivedData);
// hashCode: 495be649
```

## 5。`InetAddresses.decrement(InetAddress)`

这个方法返回一个新的`InetAddress`，它将比它的输入“小一”。

```
InetAddress address = InetAddress.getByName("127.0.0.5");
InetAddress decrementedAddress = InetAddresses.decrement(address);
// decrementedAddress: 127.0.0.4
```

## 6。`MoreExecutors` 中新增执行人

### 6.1。线程回顾

在 Java 中，你可以使用多线程来执行工作。为此，Java 有`Thread`和`Runnable`类。

```
ConcurrentHashMap<String, Boolean> threadExecutions = new ConcurrentHashMap<>();
Runnable logThreadRun = () -> threadExecutions.put(Thread.currentThread().getName(), true);

Thread t = new Thread(logThreadRun);
t.run();

Boolean isThreadExecuted = threadExecutions.get("main");
```

不出所料，`isThreadExecuted` 将是`true`。你也可以看到这个`Runnable`将只在`main` 线程中运行。如果要使用多线程，可以针对不同的目的使用不同的`Executors`。

```
ExecutorService executorService = Executors.newFixedThreadPool(2);
executorService.submit(logThreadRun);
executorService.submit(logThreadRun);
executorService.shutdown();

Boolean isThread1Executed = threadExecutions.get("pool-1-thread-1");
Boolean isThread2Executed = threadExecutions.get("pool-1-thread-2");
// isThread1Executed: true
// isThread2Executed: true
```

在这个例子中，所有提交的工作都在`ThreadPool` 线程中执行。

Guava 在其`MoreExecutors`类中提供了不同的方法。

### 6.2。`MoreExecutors.directExecutor()`

这是一个轻量级执行器，可以在调用`execute`方法的线程上运行任务。

```
Executor executor = MoreExecutors.directExecutor();
executor.execute(logThreadRun);

Boolean isThreadExecuted = threadExecutions.get("main");
// isThreadExecuted: true
```

### 6.3。`MoreExecutors.newDirectExecutorService()`

这个方法返回一个`ListeningExecutorService`的实例。它是`Executor`的一个更重的实现，有许多有用的方法。这类似于以前版本的番石榴不赞成使用的`sameThreadExecutor()`方法。

这个`ExecutorService` 将在调用`execute()`方法的线程上运行任务。

```
ListeningExecutorService executor = MoreExecutors.newDirectExecutorService();
executor.execute(logThreadRun); 
```

这个执行器有很多有用的方法比如`invokeAll, invokeAny, awaitTermination, submit, isShutdown, isTerminated, shutdown, shutdownNow`。

## 7。结论

Guava 18 对其不断增长的有用功能库进行了一些添加和改进。它非常值得考虑用于你的下一个项目。本文中的代码示例可从 GitHub 库中的[获得。](https://web.archive.org/web/20221208143856/https://github.com/eugenp/tutorials/tree/master/guava-modules/guava-18)