# 在 Spring AOP 中获取建议的方法信息

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-aop-get-advised-method-info>

## 1.介绍

在本教程中，我们将向您展示如何使用一个 [Spring AOP](/web/20220701023000/https://www.baeldung.com/spring-aop) 方面获得关于一个方法的签名、参数和注释的所有信息。

## 2.Maven 依赖性

让我们从在`pom.xml`中添加 [Spring Boot AOP Starter](https://web.archive.org/web/20220701023000/https://search.maven.org/classic/#search%7Cga%7C1%7C%20(g%3A%22org.springframework.boot%22%20AND%20a%3A%22spring-boot-starter-parent%22)%20OR%20%20(g%3A%22org.springframework.boot%22%20AND%20a%3A%22spring-boot-starter%22)%20OR%20%20(g%3A%22org.springframework.boot%22%20AND%20a%3A%22spring-boot-starter-aop%22)) 库依赖开始:

```java
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-aop</artifactId>
</dependency>
```

## 3.创建我们的切入点注释

让我们创建一个`AccountOperation`注释。为了澄清，我们将使用它作为我们方面的切入点:

```java
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
public @interface AccountOperation {
    String operation();
} 
```

**注意，创建注释对于定义切入点来说并不是强制性的。**换句话说，我们可以定义其他的切入点类型，比如类中的某些方法，以某个前缀开头的方法，等等。，通过使用 Spring AOP 提供的切入点定义语言。

## 4.创建我们的示例服务

### 4.1.帐户类别

让我们用`accountNumber`和`balance`属性创建一个`Account` POJO。我们将在服务方法中使用它作为方法参数:

```java
public class Account {

    private String accountNumber;
    private double balance;

    // getter / setters / toString
}
```

### 4.2.服务级别

现在让我们用两个用`@AccountOperation` 注释的方法创建`BankAccountService` 类，这样我们就可以获得我们方面中方法的信息。注意`withdraw` 方法抛出一个检查过的异常`WithdrawLimitException` 来演示我们如何获得关于一个方法抛出的异常的信息。

另外，注意`getBalance` 方法没有`AccountOperation`注释，所以它不会被方面拦截:

```java
@Component
public class BankAccountService {

    @AccountOperation(operation = "deposit")
    public void deposit(Account account, Double amount) {
        account.setBalance(account.getBalance() + amount);
    }

    @AccountOperation(operation = "withdraw")
    public void withdraw(Account account, Double amount) throws WithdrawLimitException {

        if(amount > 500.0) {
            throw new WithdrawLimitException("Withdraw limit exceeded.");
        }

        account.setBalance(account.getBalance() - amount);
    }

    public double getBalance() {
        return RandomUtils.nextDouble();
    }
}
```

## 5.定义我们的方面

让我们创建一个`BankAccountAspect` 来从我们的`BankAccountService:`中调用的相关方法中获取所有必要的信息

```java
@Aspect
@Component
public class BankAccountAspect {

    @Before(value = "@annotation(com.baeldung.method.info.AccountOperation)")
    public void getAccountOperationInfo(JoinPoint joinPoint) {

        // Method Information
        MethodSignature signature = (MethodSignature) joinPoint.getSignature();

        System.out.println("full method description: " + signature.getMethod());
        System.out.println("method name: " + signature.getMethod().getName());
        System.out.println("declaring type: " + signature.getDeclaringType());

        // Method args
        System.out.println("Method args names:");
        Arrays.stream(signature.getParameterNames())
          .forEach(s -> System.out.println("arg name: " + s));

        System.out.println("Method args types:");
        Arrays.stream(signature.getParameterTypes())
          .forEach(s -> System.out.println("arg type: " + s));

        System.out.println("Method args values:");
        Arrays.stream(joinPoint.getArgs())
          .forEach(o -> System.out.println("arg value: " + o.toString()));

        // Additional Information
        System.out.println("returning type: " + signature.getReturnType());
        System.out.println("method modifier: " + Modifier.toString(signature.getModifiers()));
        Arrays.stream(signature.getExceptionTypes())
          .forEach(aClass -> System.out.println("exception type: " + aClass));

        // Method annotation
        Method method = signature.getMethod();
        AccountOperation accountOperation = method.getAnnotation(AccountOperation.class);
        System.out.println("Account operation annotation: " + accountOperation);
        System.out.println("Account operation value: " + accountOperation.operation());
    }
} 
```

注意，我们将切入点定义为一个注释，所以由于我们的`BankAccountService`中的`getBalance` 方法没有用`AccountOperation,`进行注释，所以方面不会拦截它。

现在让我们详细分析我们方面的每个部分，并看看当调用`BankAccountService`方法时我们在控制台中得到什么。

### 5.1.获取有关方法签名的信息

为了能够获得我们的方法签名信息，我们需要从 [`JoinPoint`](https://web.archive.org/web/20220701023000/https://javadoc.io/doc/org.aspectj/aspectjweaver/latest/org/aspectj/lang/JoinPoint.html) 对象中检索 [`MethodSignature`](https://web.archive.org/web/20220701023000/https://www.javadoc.io/doc/org.aspectj/aspectjrt/latest/org/aspectj/lang/reflect/MethodSignature.html) :

```java
MethodSignature signature = (MethodSignature) joinPoint.getSignature();

System.out.println("full method description: " + signature.getMethod());
System.out.println("method name: " + signature.getMethod().getName());
System.out.println("declaring type: " + signature.getDeclaringType());
```

现在让我们调用服务的`withdraw()`方法:

```java
@Test
void withdraw() {
    bankAccountService.withdraw(account, 500.0);
    assertTrue(account.getBalance() == 1500.0);
}
```

运行`withdraw()` 测试后，我们现在可以在控制台上看到以下结果:

```java
full method description: public void com.baeldung.method.info.BankAccountService.withdraw(com.baeldung.method.info.Account,java.lang.Double) throws com.baeldung.method.info.WithdrawLimitException
method name: withdraw
declaring type: class com.baeldung.method.info.BankAccountService
```

### 5.2.获取关于参数的信息

为了检索关于方法参数的信息，我们可以使用`MethodSignature` 对象:

```java
System.out.println("Method args names:");
Arrays.stream(signature.getParameterNames()).forEach(s -> System.out.println("arg name: " + s));

System.out.println("Method args types:");
Arrays.stream(signature.getParameterTypes()).forEach(s -> System.out.println("arg type: " + s));

System.out.println("Method args values:");
Arrays.stream(joinPoint.getArgs()).forEach(o -> System.out.println("arg value: " + o.toString()));
```

让我们通过调用`BankAccountService`中的`deposit`方法来尝试一下:

```java
@Test
void deposit() {
    bankAccountService.deposit(account, 500.0);
    assertTrue(account.getBalance() == 2500.0);
}
```

这是我们在控制台上看到的:

```java
Method args names:
arg name: account
arg name: amount
Method args types:
arg type: class com.baeldung.method.info.Account
arg type: class java.lang.Double
Method args values:
arg value: Account{accountNumber='12345', balance=2000.0}
arg value: 500.0
```

### 5.3.获取关于方法注释的信息

我们可以通过使用`Method`类的`getAnnotation()`方法来获得关于注释的信息:

```java
Method method = signature.getMethod();
AccountOperation accountOperation = method.getAnnotation(AccountOperation.class);
System.out.println("Account operation annotation: " + accountOperation);
System.out.println("Account operation value: " + accountOperation.operation());
```

现在让我们重新运行我们的`withdraw()` 测试，检查我们得到了什么:

```java
Account operation annotation: @com.baeldung.method.info.AccountOperation(operation=withdraw)
Account operation value: withdraw
```

### 5.4.获取附加信息

我们可以获得一些关于我们的方法的附加信息，比如它们的返回类型，它们的修饰符，以及它们抛出了什么异常，如果有的话:

```java
System.out.println("returning type: " + signature.getReturnType());
System.out.println("method modifier: " + Modifier.toString(signature.getModifiers()));
Arrays.stream(signature.getExceptionTypes())
  .forEach(aClass -> System.out.println("exception type: " + aClass));
```

现在让我们创建一个新的测试`withdrawWhenLimitReached` ,让`withdraw()`方法超过它定义的撤销限制:

```java
@Test 
void withdrawWhenLimitReached() 
{ 
    Assertions.assertThatExceptionOfType(WithdrawLimitException.class)
      .isThrownBy(() -> bankAccountService.withdraw(account, 600.0)); 
    assertTrue(account.getBalance() == 2000.0); 
}
```

现在让我们检查控制台输出:

```java
returning type: void
method modifier: public
exception type: class com.baeldung.method.info.WithdrawLimitException 
```

我们最后的测试将有助于演示`getBalance()` 方法。正如我们之前所说的，它不会被方面截获，因为在方法声明中没有`AccountOperation `注释:

```java
@Test
void getBalance() {
    bankAccountService.getBalance();
}
```

当运行这个测试时，控制台中没有输出，正如我们所预料的那样。

## 6.结论

在本文中，我们看到了如何使用 Spring AOP 方面获得关于一个方法的所有可用信息。我们通过定义一个切入点，将信息打印到控制台中，并检查运行测试的结果来做到这一点。

GitHub 上的[提供了我们应用程序的源代码。](https://web.archive.org/web/20220701023000/https://github.com/eugenp/tutorials/tree/master/spring-aop-2)