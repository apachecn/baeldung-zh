# 在 Java 中将匿名类转换成 Lambda

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-from-anonymous-class-to-lambda>

## 1.概观

在本教程中，我们将学习如何用 Java 将匿名类转换成 lambda 表达式。

首先，我们将从什么是匿名类的背景开始。然后，我们将使用实际例子演示如何回答我们的核心问题。

## 2.Java 中的匿名类

简而言之，匿名类，顾名思义，就是没有名字的[内部类](/web/20221212232312/https://www.baeldung.com/java-nested-classes#non-static-nested-classes)。由于它没有名字，**我们需要在一个表达式**中同时声明和实例化它。

根据设计，匿名类扩展一个类或实现一个接口。

例如，我们可以使用`Runnable`作为匿名类，在 Java 中创建新线程。语法类似于构造函数的调用，只是我们需要将类定义放在块中:

```
Thread thread = new Thread(new Runnable() {
    @Override
    public void run() {
        ...
    }
});
```

现在我们知道了什么是匿名类，让我们看看如何使用一个 [lambda 表达式](/web/20221212232312/https://www.baeldung.com/java-8-lambda-expressions-tips)重写它。

## 3.匿名类作为 Lambda 表达式

Lambda 表达式提供了一种便捷的方式来更简洁直接地定义匿名类。

然而，这只有在匿名类只有一个方法时才有可能。所以，让我们看看如何一步一步地将匿名类转换成 lambda 表达式。

### 3.1.定义匿名类

例如，让我们考虑一下`Sender`接口:

```
public interface Sender {
    String send(String message);
}
```

正如我们所看到的，该接口只有一个声明的方法。这种类型的接口被称为[功能接口](/web/20221212232312/https://www.baeldung.com/java-8-functional-interfaces)。

接下来，让我们创建`SenderService`接口:

```
public interface SenderService {
    String callSender(Sender sender);
}
```

由于`callSender()`方法接受一个`Sender`对象作为参数，我们可以将其作为匿名类传递。

现在，我们将创建`SenderService`接口的两个实现。

首先，让我们创建`EmailSenderService`类:

```
public class EmailSenderService implements SenderService {

    @Override
    public String callSender(Sender sender) {
        return sender.send("Email Notification");
    }
}
```

接下来，让我们创建`SmsSenderService`类:

```
public class SmsSenderService implements SenderService {

    @Override
    public String callSender(Sender sender) {
        return sender.send("SMS Notification");
    }
}
```

既然我们已经把这些部分放在了一起，让我们创建第一个测试用例，其中我们将`Sender`对象作为匿名类传递:

```
@Test
public void whenPassingAnonymousClass_thenSuccess() {
    SenderService emailSenderService = new EmailSenderService();

    String emailNotif = emailSenderService.callSender(new Sender() {
        @Override
        public String send(String message) {
            return message;
        }
    });

    assertEquals(emailNotif, "Email Notification");
}
```

如上所示，我们将`Sender`对象作为匿名类传递，并覆盖了`send()`方法。

### 3.2.转换匿名类

现在，让我们尝试使用 lambda 表达式以更简洁的方式重写前面的测试用例。

由于`send()`是唯一定义的方法，编译器知道调用什么方法，所以没有必要显式覆盖它。

为了转换匿名类，**我们需要省略`new`关键字和方法名**:

```
@Test
public void whenPassingLambdaExpression_thenSuccess() {
    SenderService smsSenderService = new SmsSenderService();

    String smsNotif = smsSenderService.callSender((String message) -> {
        return message;
    });

    assertEquals(smsNotif, "SMS Notification");
}
```

正如我们所看到的，**我们用一个箭头代替了匿名类，这个箭头位于`send()`参数和它的主体**之间。

我们可以进一步增强:**我们可以通过移除参数类型和`return`语句**将 lambda 语句改为 lambda 表达式:

```
String smsNotif = smsSenderService.callSender(message -> message);
```

正如我们所看到的，我们不必指定参数类型，因为编译器可以隐式地推断出它。

## 4.结论

在本文中，我们学习了如何在 Java 中用 lambda 表达式替换匿名类。

一路上，我们解释了什么是匿名类，以及如何将它转换成 lambda 表达式。

和往常一样，本文中使用的代码可以在 GitHub 上找到[。](https://web.archive.org/web/20221212232312/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-lambdas)