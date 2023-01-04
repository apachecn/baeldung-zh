# AspectJ 简介

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/aspectj>

## 1。简介

本文是对 AspectJ 的快速实用的介绍。

首先，我们将展示如何启用面向方面的编程，然后我们将关注编译时、编译后和加载时编织之间的区别。

让我们先简单介绍一下面向方面编程(AOP)和 AspectJ 的基础知识。

## 2。概述

AOP 是一种编程范式，旨在通过允许横切关注点的分离来增加模块化。它通过向现有代码添加额外的行为来实现这一点，而无需修改代码本身。相反，我们单独声明要修改的代码。

AspectJ 使用 Java 编程语言的扩展实现关注点和横切关注点的编织。

## 3。Maven 依赖关系

AspectJ 根据其用途提供不同的库。我们可以在 Maven 中央存储库中的 group [org.aspectj](https://web.archive.org/web/20220523154131/https://search.maven.org/classic/#search%7Cga%7C1%7Cg%3A%22org.aspectj%22) 下找到 Maven 依赖项。

在本文中，我们关注使用编译时、编译后和加载时织入器创建方面和织入器所需的依赖关系。

### 3.1。AspectJ 运行时

当运行一个 AspectJ 程序时，类路径应该包含类和方面以及 AspectJ 运行时库`aspectjrt.jar` `:`

```java
<dependency>
    <groupId>org.aspectj</groupId>
    <artifactId>aspectjrt</artifactId>
    <version>1.8.9</version>
</dependency>
```

这种依赖性在 [Maven Central](https://web.archive.org/web/20220523154131/https://search.maven.org/classic/#search%7Cga%7C1%7Cg%3A%22org.aspectj%22%20AND%20a%3A%22aspectjrt%22) 上可用。

### 3.2。`AspectJWeaver`

除了 AspectJ 运行时依赖，我们还需要包含`aspectjweaver.jar` 来在加载时向 Java 类引入通知:

```java
<dependency>
    <groupId>org.aspectj</groupId>
    <artifactId>aspectjweaver</artifactId>
    <version>1.8.9</version>
</dependency>
```

依赖关系在 [Maven Central](https://web.archive.org/web/20220523154131/https://search.maven.org/classic/#artifactdetails%7Corg.aspectj%7Caspectjweaver%7C1.8.9%7Cjar) 上也是可用的。

## 4。特征创建

AspectJ 提供了 AOP 的一个实现，并且有**三个核心概念:**

*   连接点
*   切入点
*   建议

我们将通过创建一个验证用户帐户余额的简单程序来演示这些概念。

首先，让我们创建一个具有给定余额和撤销方法的`Account`类:

```java
public class Account {
    int balance = 20;

    public boolean withdraw(int amount) {
        if (balance < amount) {
            return false;
        } 
        balance = balance - amount;
        return true;
    }
}
```

我们将创建一个`AccountAspect.aj`文件来记录帐户信息并验证帐户余额(注意，AspectJ 文件以“`.aj`”文件扩展名结尾):

```java
public aspect AccountAspect {
    final int MIN_BALANCE = 10;

    pointcut callWithDraw(int amount, Account acc) : 
     call(boolean Account.withdraw(int)) && args(amount) && target(acc);

    before(int amount, Account acc) : callWithDraw(amount, acc) {
    }

    boolean around(int amount, Account acc) : 
      callWithDraw(amount, acc) {
        if (acc.balance < amount) {
            return false;
        }
        return proceed(amount, acc);
    }

    after(int amount, Account balance) : callWithDraw(amount, balance) {
    }
}
```

正如我们所看到的，我们已经向取款方法添加了一个`pointcut`，并创建了三个引用已定义的`pointcut`的`advises`。

为了理解下文，我们引入以下定义:

*   **方面**:横切多个对象的关注点的模块化。每个方面都专注于特定的横切功能
*   **连接点**:脚本执行过程中的一个点，比如方法或属性访问的执行
*   **建议**:某个方面在特定连接点采取的动作
*   **切入点**:匹配连接点的正则表达式。一个通知与一个切入点表达式相关联，并在任何匹配切入点的连接点上运行

关于这些概念及其具体语义的更多细节，我们可能想要查看下面的[链接](https://web.archive.org/web/20220523154131/https://eclipse.org/aspectj/doc/next/progguide/semantics-pointcuts.html)。

接下来，我们需要将方面编织到我们的代码中。下面几节讨论三种不同类型的编织:编译时编织、编译后编织和 AspectJ 中的加载时编织。

## 5。编译时编织

最简单的编织方法是编译时编织。当我们既有方面的源代码，又有我们在其中使用方面的代码时，AspectJ 编译器将从源代码进行编译，并产生一个编织的类文件作为输出。之后，在执行代码时，编织过程输出类作为普通的 Java 类被加载到 JVM 中。

我们可以下载 [AspectJ 开发工具](https://web.archive.org/web/20220523154131/https://eclipse.org/aspectj/downloads.php#ides),因为它包含一个捆绑的 AspectJ 编译器。AJDT 最重要的特性之一是一个可视化横切关注点的工具，这有助于调试切入点规范。甚至在部署代码之前，我们就可以看到组合的效果。

我们使用 Mojo 的 AspectJ Maven 插件，通过 AspectJ 编译器将 AspectJ 方面编织到我们的类中。

```java
<plugin>
    <groupId>org.codehaus.mojo</groupId>
    <artifactId>aspectj-maven-plugin</artifactId>
    <version>1.7</version>
    <configuration>
        <complianceLevel>1.8</complianceLevel>
        <source>1.8</source>
        <target>1.8</target>
        <showWeaveInfo>true</showWeaveInfo>
        <verbose>true</verbose>
        <Xlint>ignore</Xlint>
        <encoding>UTF-8 </encoding>
    </configuration>
    <executions>
        <execution>
            <goals>
                <!-- use this goal to weave all your main classes -->
                <goal>compile</goal>
                <!-- use this goal to weave all your test classes -->
                <goal>test-compile</goal>
            </goals>
        </execution>
    </executions>
</plugin>
```

关于 AspectJ 编译器的选项引用的更多细节，我们可能想看看下面的[链接](https://web.archive.org/web/20220523154131/https://www.mojohaus.org/aspectj-maven-plugin/ajc_reference/standard_opts.html)。

让我们为我们的 Account 类添加一些测试用例:

```java
public class AccountTest {
    private Account account;

    @Before
    public void before() {
        account = new Account();
    }

    @Test
    public void given20AndMin10_whenWithdraw5_thenSuccess() {
        assertTrue(account.withdraw(5));
    }

    @Test
    public void given20AndMin10_whenWithdraw100_thenFail() {
        assertFalse(account.withdraw(100));
    }
}
```

当我们运行测试用例时，控制台中显示的以下文本意味着我们成功地编织了源代码:

```java
[INFO] Join point 'method-call
(boolean com.baeldung.aspectj.Account.withdraw(int))' in Type
'com.baeldung.aspectj.test.AccountTest' (AccountTest.java:20)
advised by around advice from 'com.baeldung.aspectj.AccountAspect'
(AccountAspect.class:18(from AccountAspect.aj))

[INFO] Join point 'method-call
(boolean com.baeldung.aspectj.Account.withdraw(int))' in Type 
'com.baeldung.aspectj.test.AccountTest' (AccountTest.java:20) 
advised by before advice from 'com.baeldung.aspectj.AccountAspect' 
(AccountAspect.class:13(from AccountAspect.aj))

[INFO] Join point 'method-call
(boolean com.baeldung.aspectj.Account.withdraw(int))' in Type 
'com.baeldung.aspectj.test.AccountTest' (AccountTest.java:20) 
advised by after advice from 'com.baeldung.aspectj.AccountAspect'
(AccountAspect.class:26(from AccountAspect.aj))

2016-11-15 22:53:51 [main] INFO  com.baeldung.aspectj.AccountAspect 
-  Balance before withdrawal: 20
2016-11-15 22:53:51 [main] INFO  com.baeldung.aspectj.AccountAspect 
-  Withdraw ammout: 5
2016-11-15 22:53:51 [main] INFO  com.baeldung.aspectj.AccountAspect 
- Balance after withdrawal : 15
2016-11-15 22:53:51 [main] INFO  com.baeldung.aspectj.AccountAspect 
-  Balance before withdrawal: 20
2016-11-15 22:53:51 [main] INFO  com.baeldung.aspectj.AccountAspect 
-  Withdraw ammout: 100
2016-11-15 22:53:51 [main] INFO  com.baeldung.aspectj.AccountAspect 
- Withdrawal Rejected!
2016-11-15 22:53:51 [main] INFO  com.baeldung.aspectj.AccountAspect 
- Balance after withdrawal : 20
```

## 6。编译后编织

编译后编织(有时也称为二进制编织)用于编织现有的类文件和 JAR 文件。与编译时编织一样，用于编织的方面可以是源代码或二进制形式，并且它们本身可以由方面编织。

为了用 Mojo 的 AspectJ Maven 插件做到这一点，我们需要设置所有我们想在插件配置中编织的 JAR 文件:

```java
<configuration>
    <weaveDependencies>
        <weaveDependency>  
            <groupId>org.agroup</groupId>
            <artifactId>to-weave</artifactId>
        </weaveDependency>
        <weaveDependency>
            <groupId>org.anothergroup</groupId>
            <artifactId>gen</artifactId>
        </weaveDependency>
    </weaveDependencies>
</configuration>
```

包含要编织的类的 JAR 文件必须在 Maven 项目中作为`<dependencies/>`列出，在 AspectJ Maven 插件的`<configuration>`中作为`<weaveDependencies/>`列出。

## 7 .**。加载时编织**

加载时编织只是简单的二进制编织，直到类加载器加载一个类文件并为 JVM 定义该类。

为了支持这一点，需要一个或多个“编织类装入器”。这些要么由运行时环境显式提供，要么使用“编织代理”来启用。

### 7.1。启用加载时编织

可以使用 AspectJ 代理启用 AspectJ 加载时编织，该代理可以参与类加载过程，并在 VM 中定义类型之前编织任何类型。我们为 JVM ``-javaagent:pathto/aspectjweaver.jar``指定`javaagent`选项，或者使用 Maven 插件来配置`javaagent` :

```java
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-surefire-plugin</artifactId>
    <version>2.22.2</version>
    <configuration>
        <argLine>
            -javaagent:"${settings.localRepository}"/org/aspectj/
            aspectjweaver/${aspectj.version}/
            aspectjweaver-${aspectj.version}.jar
        </argLine>
        <useSystemClassLoader>true</useSystemClassLoader>
        <forkMode>always</forkMode>
    </configuration>
</plugin>
```

### 7.2。配置编织器

AspectJ 的加载时编织代理是通过使用`aop.xml`文件来配置的。它在`META-INF`目录中的类路径上查找一个或多个`aop.xml` 文件，并聚合内容以确定 weaver 配置。

一个 `aop.xml`文件包含两个关键部分:

*   **方面**:为编织者定义一个或多个方面，并控制在编织过程中使用哪些方面。`aspects`元素可以选择包含一个或多个`include`和`exclude`元素(默认情况下，所有定义的方面都用于编织)
*   **weaver** :为 Weaver 定义 Weaver 选项，并指定应该编织的类型集。如果没有指定 include 元素，那么所有对 weaver 可见的类型都将被编织

让我们为 weaver 配置一个方面:

```java
<aspectj>
    <aspects>
        <aspect name="com.baeldung.aspectj.AccountAspect"/>
        <weaver options="-verbose -showWeaveInfo">
            <include within="com.baeldung.aspectj.*"/>
        </weaver>
    </aspects>
</aspectj>
```

我们可以看到，我们配置了一个指向`AccountAspect`的方面，只有`com.baeldung.aspectj`包中的源代码会被 AspectJ 织入。

## 8。注释方面

除了熟悉的基于 AspectJ 代码的方面声明风格之外，AspectJ 5 还支持基于注释的方面声明风格。我们非正式地将支持这种开发风格的注释集称为“`@AspectJ`”注释。

让我们创建一个注释:

```java
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface Secured {
    public boolean isLocked() default false; 
}
```

我们使用`@Secured`注释来启用或禁用一个方法:

```java
public class SecuredMethod {

    @Secured(isLocked = true)
    public void lockedMethod() {
    }

    @Secured(isLocked = false)
    public void unlockedMethod() {
    }
}
```

接下来，我们使用 AspectJ 注释样式添加一个方面，并根据@Secured 注释的属性检查权限:

```java
@Aspect
public class SecuredMethodAspect {
    @Pointcut("@annotation(secured)")
    public void callAt(Secured secured) {
    }

    @Around("callAt(secured)")
    public Object around(ProceedingJoinPoint pjp, 
      Secured secured) throws Throwable {
        return secured.isLocked() ? null : pjp.proceed();
    }
}
```

关于 AspectJ 注释风格的更多细节，我们可以查看下面的[链接](//web.archive.org/web/20220523154131/https://eclipse.org/aspectj/doc/released/adk15notebook/annotations-aspectmembers.html)。

接下来，我们使用加载时织入器织入我们的类和方面，并将`aop.xml`放在`META-INF` 文件夹下:

```java
<aspectj>
    <aspects>
        <aspect name="com.baeldung.aspectj.SecuredMethodAspect"/>
        <weaver options="-verbose -showWeaveInfo">
            <include within="com.baeldung.aspectj.*"/>
        </weaver>
    </aspects>
</aspectj>
```

最后，我们添加单元测试并检查结果:

```java
@Test
public void testMethod() throws Exception {
	SecuredMethod service = new SecuredMethod();
	service.unlockedMethod();
	service.lockedMethod();
}
```

当我们运行测试用例时，我们可以检查控制台输出，以验证我们成功地在源代码中编织了我们的方面和类:

```java
[INFO] Join point 'method-call
(void com.baeldung.aspectj.SecuredMethod.unlockedMethod())'
in Type 'com.baeldung.aspectj.test.SecuredMethodTest'
(SecuredMethodTest.java:11)
advised by around advice from 'com.baeldung.aspectj.SecuredMethodAspect'
(SecuredMethodAspect.class(from SecuredMethodAspect.java))

2016-11-15 22:53:51 [main] INFO com.baeldung.aspectj.SecuredMethod 
- unlockedMethod
2016-11-15 22:53:51 [main] INFO c.b.aspectj.SecuredMethodAspect - 
public void com.baeldung.aspectj.SecuredMethod.lockedMethod() is locked
```

## 9。结论

在本文中，我们讨论了关于 AspectJ 的介绍性概念。具体可以看一下 [AspectJ 主页。](https://web.archive.org/web/20220523154131/https://eclipse.org/aspectj/)

你可以在 GitHub 上找到这篇文章[的源代码。](https://web.archive.org/web/20220523154131/https://github.com/eugenp/tutorials/tree/master/spring-aop)