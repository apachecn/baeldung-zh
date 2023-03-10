# 谷歌指南

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/guice>

## 1。简介

在本教程中，我们将研究 Google Guice 的基础知识。然后，我们将看看在 Guice 中完成基本依赖注入(DI)任务的一些方法。

我们还将比较和对比 Guice 方法和那些更成熟的 DI 框架，比如 Spring、Contexts 和 Dependency Injection (CDI)。

本教程假设读者已经理解了[依赖注入模式](/web/20220930092237/https://www.baeldung.com/inversion-control-and-dependency-injection-in-spring)的基础。

## 2。设置

为了在我们的 Maven 项目中使用 Google Guice，我们需要向我们的`pom.xml`添加以下依赖项:

```java
<dependency>
    <groupId>com.google.inject</groupId>
    <artifactId>guice</artifactId>
    <version>4.1.0</version>
</dependency> 
```

这里还有一组 Guice 扩展(我们稍后会谈到)，以及作为第三方模块的来扩展 Guice 的功能(主要是通过提供与更成熟的 Java 框架的集成)。

## 3。使用 Guice 的基本依赖注入

### 3.1。我们的示例应用程序

我们将在一个场景中设计支持服务台业务中三种通信方式的类:电子邮件、SMS 和 IM。

首先，让我们考虑这个类:

```java
public class Communication {

    @Inject 
    private Logger logger;

    @Inject
    private Communicator communicator;

    public Communication(Boolean keepRecords) {
        if (keepRecords) {
            System.out.println("Message logging enabled");
        }
    }

    public boolean sendMessage(String message) {
        return communicator.sendMessage(message);
    }

}
```

这个`Communication` 类是沟通的基本单位。此类的一个实例用于通过可用的通信信道发送消息。如上所示，`Communication`有一个`Communicator,`，我们将使用它来进行实际的消息传输。

果汁的基本切入点是`Injector:`

```java
public static void main(String[] args){
    Injector injector = Guice.createInjector(new BasicModule());
    Communication comms = injector.getInstance(Communication.class);
} 
```

这个 main 方法检索我们的`Communication`类的一个实例。它还介绍了 Guice 的一个基本概念:`Module`(在本例中使用了`BasicModule`)。**`Module`是定义绑定**(或者在 Spring 中称为连线)的基本单位。

**Guice 已经采用了代码优先的方法来进行依赖注入和管理，**所以我们不会处理太多现成的 XML。

在上面的例子中，`Communication`的依赖树将使用一个名为`just-in-time binding`的特性被隐式地注入，假设这些类有默认的无参数构造函数。从一开始这就是 Guice 的一个特性，从 4.3 版开始只在 Spring 中可用。

### 3.2。Guice 基本绑定

捆绑之于导线，犹如电线之于弹簧。通过绑定，我们**定义 Guice 如何将依赖关系**注入到一个类中。

在 `com.google.inject.AbstractModule`的实现中定义了绑定:

```java
public class BasicModule extends AbstractModule {

    @Override
    protected void configure() {
        bind(Communicator.class).to(DefaultCommunicatorImpl.class);
    }
}
```

这个模块实现指定在任何找到`Communicator`变量的地方都要注入`Default` `CommunicatorImpl` 的实例。

### 3.3.命名绑定

**这种机制的另一个化身是`named binding`** 。考虑以下变量声明:

```java
@Inject @Named("DefaultCommunicator")
Communicator communicator; 
```

为此，我们将有以下绑定定义:

```java
@Override
protected void configure() {
    bind(Communicator.class)
      .annotatedWith(Names.named("DefaultCommunicator"))
      .to(DefaultCommunicatorImpl.class);
} 
```

这个绑定将为用`@Named(“DefaultCommunicator”)` 注释标注的变量提供一个 `Communicator`的实例。

我们还可以看到，`@Inject` 和`@Named`注释似乎是来自 Jakarta EE 的 CDI 的贷款注释，事实也的确如此。它们在`com.google.inject.*` 包中，当使用 IDE 时，我们应该小心从正确的包中导入。

虽然我们只是说使用 Guice 提供的`@Inject` 和`@Named`，但是值得注意的是 Guice 确实提供了对`javax.inject.Inject` 和`javax.inject.Named,` 以及其他 Jakarta EE 注释的支持。

### 3.4.构造函数绑定

**我们还可以使用`constructor binding`** 注入一个没有默认无参数构造函数的依赖项:

```java
public class BasicModule extends AbstractModule {

    @Override
    protected void configure() {
        bind(Boolean.class).toInstance(true);
        bind(Communication.class).toConstructor(
          Communication.class.getConstructor(Boolean.TYPE));
} 
```

上面的代码片段将使用带有一个`boolean` 参数的构造函数注入一个`Communication`实例。我们通过**定义`Boolean`类的`untargeted binding`** 向构造函数提供`true`参数。

此外，这个`untargeted binding`将被急切地提供给绑定中接受`boolean`参数的任何构造函数。用这种方法，我们可以注入`Communication`的所有依赖项。

**另一种特定于构造函数的绑定方法是`instance binding`** ，我们在绑定中直接提供一个实例:

```java
public class BasicModule extends AbstractModule {

    @Override
    protected void configure() {
        bind(Communication.class)
          .toInstance(new Communication(true));
    }    
}
```

这个绑定将在我们声明一个`Communication`变量的地方提供一个`Communication` 类的实例。

然而，在这种情况下，类的依赖树不会自动连接。此外，我们应该限制这种模式的使用，因为这种模式不需要任何繁重的初始化或依赖注入。

## 4。依赖注入的类型

Guice 还支持我们在 DI 模式中所期望的标准类型的注入。在`Communicator` 类中，我们需要注入不同类型的`CommunicationMode`。

### 4.1。现场注射

```java
@Inject @Named("SMSComms")
CommunicationMode smsComms;
```

我们可以使用可选的`@Named`注释作为限定符来实现基于名称的目标注入。

### 4.2。方法注入

这里我们将使用一个 setter 方法来实现注入:

```java
@Inject
public void setEmailCommunicator(@Named("EmailComms") CommunicationMode emailComms) {
    this.emailComms = emailComms;
} 
```

### 4.3。构造函数注入

我们还可以使用构造函数注入依赖关系:

```java
@Inject
public Communication(@Named("IMComms") CommunicationMode imComms) {
    this.imComms= imComms;
} 
```

### 4.4。隐式注射

Guice 还将隐式注入一些通用组件，比如`Injector`和一个`java.util.Logger`实例，等等。请注意，我们在所有示例中都使用了记录器，但是我们不会为它们找到实际的绑定。

## 5。Guice 中的作用域

Guice 支持我们在其他 DI 框架中已经习惯的作用域和作用域机制。Guice 默认提供已定义依赖项的新实例。

### 5.1。单例

让我们在应用程序中注入一个 singleton:

```java
bind(Communicator.class).annotatedWith(Names.named("AnotherCommunicator"))
  .to(Communicator.class).in(Scopes.SINGLETON); 
```

`in(Scopes.SINGLETON)`指定任何带有`@Named(“AnotherCommunicator”)`注释的`Communicator` 字段都将被注入一个单例。默认情况下，这个单例是延迟启动的。

### 5.2。急切的单例

然后我们将注入一个热切的单例:

```java
bind(Communicator.class).annotatedWith(Names.named("AnotherCommunicator"))
  .to(Communicator.class)
  .asEagerSingleton(); 
```

`asEagerSingleton()`调用将单例定义为急切实例化。

除了这两个作用域，Guice 还支持自定义作用域，以及 Jakarta EE 提供的仅用于 web 的`@RequestScoped` 和`@SessionScoped`注释(这些注释没有 Guice 提供的版本)。

## 6。Guice 中的面向方面编程

Guice 符合 AOPAlliance 的面向方面编程规范。我们可以实现典型的日志拦截器，在我们的例子中，我们只用四个步骤就可以用它来跟踪消息发送。

**第一步——实现 AOPAlliance 的** `**[MethodInterceptor](https://web.archive.org/web/20220930092237/http://aopalliance.sourceforge.net/doc/org/aopalliance/intercept/MethodInterceptor.html)**`:

```java
public class MessageLogger implements MethodInterceptor {

    @Inject
    Logger logger;

    @Override
    public Object invoke(MethodInvocation invocation) throws Throwable {
        Object[] objectArray = invocation.getArguments();
        for (Object object : objectArray) {
            logger.info("Sending message: " + object.toString());
        }
        return invocation.proceed();
    }
} 
```

**步骤 2–定义一个普通的 Java 注释**:

```java
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface MessageSentLoggable {
} 
```

**步骤 3–为匹配器定义绑定:**

`Matcher` 是一个 Guice 类，我们将使用它来指定我们的 AOP 注释将应用到的组件。在这种情况下，我们希望注释应用于`CommunicationMode:`的实现

```java
public class AOPModule extends AbstractModule {

    @Override
    protected void configure() {
        bindInterceptor(
            Matchers.any(),
            Matchers.annotatedWith(MessageSentLoggable.class),
            new MessageLogger()
        );
    }
} 
```

这里我们指定了一个`Matcher`，它将把我们的`MessageLogger`拦截器应用到`any`类，该类的方法应用了`MessageSentLoggable`注释。

**步骤 4–将我们的注释应用到我们的通信模式，并加载我们的模块**

```java
@Override
@MessageSentLoggable
public boolean sendMessage(String message) {
    logger.info("SMS message sent");
    return true;
}

public static void main(String[] args) {
    Injector injector = Guice.createInjector(new BasicModule(), new AOPModule());
    Communication comms = injector.getInstance(Communication.class);
}
```

## 7 .**。结论**

看了基本的 Guice 功能，我们可以看到 Guice 的灵感来自 Spring。

除了对 [JSR-330](https://web.archive.org/web/20220930092237/https://github.com/google/guice/wiki/JSR330) 的支持，Guice 的目标是成为一个专注于注入的 DI 框架(而 Spring 为了编程方便提供了一个完整的生态系统，不一定仅仅是 DI ),目标是那些想要 DI 灵活性的开发者。

Guice 也是高度可扩展的，允许程序员编写可移植的插件，从而灵活和创造性地使用该框架。除此之外，Guice 已经为最流行的框架和平台提供了广泛的集成，比如 Servlets、JSF、JPA 和 OSGi 等等。

本文中使用的所有源代码都可以在我们的 [GitHub 项目](https://web.archive.org/web/20220930092237/https://github.com/eugenp/tutorials/tree/master/guice)中获得。