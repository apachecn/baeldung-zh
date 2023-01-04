# Java 中何时使用 Callable 和 Supplier

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-callable-vs-supplier>

## 1.概观

在本教程中，我们将讨论`Callable`和`Supplier` [功能接口](/web/20221220042828/https://www.baeldung.com/java-8-functional-interfaces)，这两个接口结构相似，但用途不同。

两者都返回一个类型值，并且不带任何参数。执行上下文是决定差异的判别式。

在本教程中，我们将关注异步任务的上下文。

## 2.模型

在我们开始之前，让我们定义一个类:

```
public class User {

    private String name;
    private String surname;
    private LocalDate birthDate;
    private Integer age;
    private Boolean canDriveACar = false;

    // standard constructors, getters and setters
}
```

## 3.`Callable`

`Callable`是 Java 版本 5 中引入的接口，在版本 8 中演化为功能接口。

**它的 SAM(单一抽象方法)是返回一个泛型值的方法`call()`，可能抛出一个异常:**

```
V call() throws Exception;
```

它被设计成封装一个应该由另一个线程执行的任务，比如 [`Runnable`](/web/20221220042828/https://www.baeldung.com/java-runnable-callable) 接口。这是因为`Callable`实例可以通过`[ExecutorService](/web/20221220042828/https://www.baeldung.com/java-executor-service-tutorial)`来执行。

因此，让我们定义一个实现:

```
public class AgeCalculatorCallable implements Callable<Integer> {

    private final LocalDate birthDate;

    @Override
    public Integer call() throws Exception {
        return Period.between(birthDate, LocalDate.now()).getYears();
    }

    // standard constructors, getters and setters
}
```

当`call()`方法返回一个值时，主线程检索它来执行它的逻辑。为此，我们可以使用 [`Future`](/web/20221220042828/https://www.baeldung.com/java-future) ，一个在另一个线程上执行的任务完成时跟踪并获取值的对象。

### 3.1.单一任务

让我们定义一个只执行一个异步任务的方法:

```
public User execute(User user) {
    ExecutorService executorService = Executors.newCachedThreadPool();
    try {
        Future<Integer> ageFuture = executorService.submit(new AgeCalculatorCallable(user.getBirthDate()));
        user.setAge(age.get());
    } catch (ExecutionException | InterruptedException e) {
        throw new RuntimeException(e.getCause());
    }
    return user;
}
```

我们可以通过 lambda 表达式重写`submit()` 的内部块:

```
Future<Integer> ageFuture = executorService.submit(
  () -> Period.between(user.getBirthDate(), LocalDate.now()).getYears());
```

当我们试图通过调用 `get()`方法来访问返回值时，我们必须处理两个被检查的异常:

*   `InterruptedException`:当线程睡眠、活动或被占用时发生中断时抛出
*   `ExecutionException`:通过抛出异常中止任务时抛出。换句话说，这是一个包装器异常，中止任务的真正异常是原因所在(可以使用`getCause`()方法来检查)。

### 3.2.任务链

执行属于链的任务取决于先前任务的状态。如果其中一个失败，当前任务将无法执行。

所以让我们定义一个新的`Callable`:

```
public class CarDriverValidatorCallable implements Callable<Boolean> {

    private final Integer age;

    @Override
    public Boolean call() throws Exception {
        return age > 18;
    }
    // standard constructors, getters and setters
}
```

接下来，让我们定义一个任务链，其中第二个任务将前一个任务的结果作为输入参数:

```
public User execute(User user) {
    ExecutorService executorService = Executors.newCachedThreadPool();
    try {
        Future<Integer> ageFuture = executorService.submit(new AgeCalculatorCallable(user.getBirthDate()));
        Integer age = ageFuture.get();
        Future<Boolean> canDriveACarFuture = executorService.submit(new CarDriverValidatorCallable(age));
        Boolean canDriveACar = canDriveACarFuture.get();
        user.setAge(age);
        user.setCanDriveACar(canDriveACar);
    } catch (ExecutionException | InterruptedException e) {
        throw new RuntimeException(e.getCause());
    }
    return user;
}
```

在任务链中使用`Callable`和`Future`有一些问题:

*   链中的每个任务都遵循“提交-获取”模式。在一个长链中，这会产生冗长的代码。
*   当链容忍任务失败时，我们应该创建一个专用的`try` / `catch`块。
*   当被调用时，`get()`方法等待，直到`Callable`返回值。所以这个链的总执行时间等于所有任务的执行时间之和。但是如果下一个任务只依赖于前一个任务的正确执行，那么这个链式过程就会明显变慢。

## 4.`Supplier`

`Supplier`是 SAM(单一抽象方法)为 `get()`的函数接口。

**它不接受任何参数，返回一个值，并且只抛出未检查的异常:**

```
T get();
```

这个接口最常见的一个用例是推迟一些代码的执行。

[`Optional`](/web/20221220042828/https://www.baeldung.com/java-optional) 类有一些方法接受一个`Supplier`作为参数，比如`Optional.or()`、`Optional.orElseGet().`

所以只有当`Optional`为空时，才会执行`Supplier`。

我们还可以在异步计算环境中使用它，特别是在 [CompletableFuture](/web/20221220042828/https://www.baeldung.com/java-completablefuture) API 中。

有些方法接受一个`Supplier`作为参数，比如`supplyAsync()` 方法。

### 4.1.单一任务

让我们定义一个只执行一个异步任务的方法:

```
public User execute(User user) {
    ExecutorService executorService = Executors.newCachedThreadPool();
    CompletableFuture<Integer> ageFut = CompletableFuture.supplyAsync(() -> Period.between(user.getBirthDate(), LocalDate.now())
      .getYears(), executorService)
      .exceptionally(throwable -> {throw new RuntimeException(throwable);});
    user.setAge(ageFut.join());
    return user;
}
```

在这种情况下，lambda 表达式定义了`Supplier`，但是我们也可以定义一个实现类。感谢`CompletableFuture,`,我们为异步操作定义了一个模板，使它更容易理解和修改。

`join()`方法提供了`Supplier.`的返回值

### 4.2.任务链

我们还可以在`Supplier`接口和`CompletableFuture`的支持下开发一系列任务:

```
public User execute(User user) {
    ExecutorService executorService = Executors.newCachedThreadPool();
    CompletableFuture<Integer> ageFut = CompletableFuture.supplyAsync(() -> Period.between(user.getBirthDate(), LocalDate.now())
      .getYears(), executorService);
    CompletableFuture<Boolean> canDriveACarFut = ageFut.thenComposeAsync(age -> CompletableFuture.supplyAsync(() -> age > 18, executorService))
      .exceptionally((ex) -> false);
    user.setAge(ageFut.join());
    user.setCanDriveACar(canDriveACarFut.join());
    return user;
}
```

用`CompletableFuture`–`Supplier`方法定义一系列异步任务可能会解决之前用`Future`–`Callable`方法引入的一些问题:

*   链中的每个任务都是独立的。因此，如果任务执行失败，我们可以通过`exceptionally()`块来处理它。
*   方法不需要在编译时处理检查过的异常。
*   我们可以设计一个异步任务模板，改进每个任务的状态处理。

## 5.结论

在本文中，我们讨论了`Callable`和`Supplier`接口之间的差异，重点是异步任务的上下文。

**接口设计层面的主要区别是`Callable`抛出的检查异常。**

`Callable`不是指功能上下文。它是随着时间的推移而适应的，函数式编程和检查异常并不和谐。

所以任何函数 API(比如`CompletableFuture` API)总是接受`Supplier`而不是`Callable`。

和往常一样，例子的完整源代码可以在 GitHub 上找到[。](https://web.archive.org/web/20221220042828/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-lambdas)