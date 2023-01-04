# Java 命名和目录接口概述

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/jndi>

## 1.介绍

Java 命名和目录接口(JNDI)提供了命名和/或目录服务作为 Java API 的一致使用。该接口可用于绑定对象、查找或查询对象，以及检测相同对象上的变化。

虽然 JNDI 的使用包括各种各样的[支持的命名和目录服务](https://web.archive.org/web/20221206020025/https://docs.oracle.com/javase/8/docs/technotes/guides/jndi/index.html)，但在本教程中，我们将重点关注 JDBC，同时探索 JNDI 的 API。

## 2.JNDI 描述

任何与 JNDI 的合作都需要**理解底层服务**以及**一个可访问的实现。**例如，数据库连接服务需要特定的属性和异常处理。

然而，JNDI 的抽象将连接配置从应用程序中分离出来。

让我们探索一下包含 JNDI 核心功能的`Name`和`Context`。

### 2.1.`Name`界面

```java
Name objectName = new CompositeName("java:comp/env/jdbc");
```

`Name`接口提供了管理组件名称和 JNDI 名称语法的能力。字符串的第一个标记代表全局上下文，之后添加的每个字符串代表下一个子上下文:

```java
Enumeration<String> elements = objectName.getAll();
while(elements.hasMoreElements()) {
  System.out.println(elements.nextElement());
}
```

我们的输出看起来像:

```java
java:comp
env
jdbc
```

正如我们所见，`/` 是`Name` 子上下文的分隔符。现在，让我们添加一个子上下文:

```java
objectName.add("example");
```

然后我们测试我们的加法:

```java
assertEquals("example", objectName.get(objectName.size() - 1));
```

### 2.2.`Context`界面

`Context` 包含命名和目录服务`.` 的属性。这里，为了方便起见，让我们使用 Spring 的一些助手代码来构建一个`Context`:

```java
SimpleNamingContextBuilder builder = new SimpleNamingContextBuilder(); 
builder.activate();
```

Spring 的`SimpleNamingContextBuilder`创建一个 JNDI 提供者，然后用 [`NamingManager`](https://web.archive.org/web/20221206020025/https://docs.oracle.com/en/java/javase/11/docs/api/java.naming/javax/naming/spi/NamingManager.html) 激活构建者:

```java
JndiTemplate jndiTemplate = new JndiTemplate();
ctx = (InitialContext) jndiTemplate.getContext();
```

最后，`JndiTemplate `帮助我们访问`InitialContext`。

## 3.JNDI 对象绑定和查找

现在我们已经看到了如何使用`Name`和`Context`，让我们使用 JNDI 来存储一个 JDBC `DataSource`:

```java
ds = new DriverManagerDataSource("jdbc:h2:mem:mydb");
```

### 3.1.绑定 JNDI 对象

因为我们有一个上下文，让我们将对象绑定到它:

```java
ctx.bind("java:comp/env/jdbc/datasource", ds);
```

通常，服务应该在目录上下文中存储对象引用、序列化数据或属性。这完全取决于应用程序的需求。

注意，以这种方式使用 JNDI 并不常见。通常，JNDI 与应用程序运行时外部管理的数据进行交互。

然而，如果应用程序已经可以创建或找到它的`DataSource`，那么使用 Spring 连接它可能会更容易。相反，如果我们的应用程序之外的东西绑定了 JNDI 的对象，那么应用程序就可以使用它们。

### 3.2.查找 JNDI 对象

让我们来看看我们的`DataSource`:

```java
DataSource ds = (DataSource) ctx.lookup("java:comp/env/jdbc/datasource");
```

然后让我们测试一下，确保`DataSource` 如预期的那样:

```java
assertNotNull(ds.getConnection());
```

### 4.常见的 JNDI 例外

使用 JNDI 有时可能会导致运行时异常。下面是一些常见的。

### 4.1.`NameNotFoundException`

```java
ctx.lookup("badJndiName");
```

由于该名称未在此上下文中绑定，因此我们看到以下堆栈跟踪:

```java
javax.naming.NameNotFoundException: Name [badJndiName] not bound; 0 bindings: []
  at org.springframework.mock.jndi.SimpleNamingContext.lookup(SimpleNamingContext.java:140)
  at java.naming/javax.naming.InitialContext.lookup(InitialContext.java:409)
```

我们应该注意，堆栈跟踪包含所有绑定的对象，这对于跟踪异常发生的原因很有用。

### 4.2.`NoInitialContextException`

任何与`InitialContext`的交互都会抛出`NoInitialContextException`:

```java
assertThrows(NoInitialContextException.class, () -> {
  JndiTemplate jndiTemplate = new JndiTemplate();
  InitialContext ctx = (InitialContext) jndiTemplate.getContext();
  ctx.lookup("java:comp/env/jdbc/datasource");
}).printStackTrace();
```

我们应该注意，JNDI 的这种用法是有效的，就像我们前面使用它一样。但是，这一次没有 JNDI 上下文提供程序，将会抛出一个异常:

```java
javax.naming.NoInitialContextException: Need to specify class name in environment or system property, 
  or in an application resource file: java.naming.factory.initial
    at java.naming/javax.naming.spi.NamingManager.getInitialContext(NamingManager.java:685)
```

## 5.JNDI 在现代应用架构中的作用

虽然 JNDI 在轻量级、容器化的 Java 应用程序(如 Spring Boot)中的作用较小，但它还有其他用途。三种仍然使用 JNDI 的 Java 技术是 [JDBC](/web/20221206020025/https://www.baeldung.com/java-jdbc) 、 [EJB](/web/20221206020025/https://www.baeldung.com/ejb-intro) 和 [JMS](/web/20221206020025/https://www.baeldung.com/spring-jms) 。它们在 Java 企业应用程序中都有广泛的用途。

例如，一个单独的 DevOps 团队可能管理环境变量，如所有环境中敏感数据库连接的用户名和密码。可以在 web 应用程序容器中创建 JNDI 资源，将 JNDI 用作在所有环境中工作的一致抽象层。

这种设置允许开发人员创建和控制用于开发目的的本地定义，同时通过相同的 JNDI 名称连接到生产环境中的敏感资源。

## 6.结论

在本教程中，我们看到了使用 Java 命名和目录接口来连接、绑定和查找对象。我们还看了 JNDI 抛出的常见异常。

最后，我们研究了 JNDI 如何融入现代应用程序架构。

和往常一样，代码[可以在 GitHub](https://web.archive.org/web/20221206020025/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-jndi) 上获得。