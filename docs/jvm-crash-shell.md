# 速成入门

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/jvm-crash-shell>

## 1.介绍

CRaSH 是一个可重用的 shell，部署在 JVM 中，帮助我们与 JVM 进行交互。

在本教程中，我们将看到如何安装崩溃作为一个独立的应用程序。此外，我们将嵌入一个 Spring Web 应用程序，并创建一些自定义命令。

## 2.独立安装

让我们从 [CRaSH 的官网下载发行版，将 CRaSH 安装为一个独立的应用程序。](https://web.archive.org/web/20220629004727/https://www.crashub.org/)

崩溃目录结构包含三个重要目录`cmd, bin,`和`conf:`

[![](img/6a65e9d36b3709ebff632d15f525752b.png)](/web/20220629004727/https://www.baeldung.com/wp-content/uploads/2020/02/Screenshot-2020-02-19-at-22.45.54.png)

`bin` 目录包含启动崩溃的独立 CLI 脚本。

`cmd `目录包含所有它支持的现成命令。此外，这也是我们可以放置自定义命令的地方。我们将在本文的后面部分对此进行研究。

要启动 CLI，我们转到`bin` ，用`crash.bat` 或`crash.sh:`启动独立实例

[![](img/6745a6ffe794bf7b60321b94bb4fed0c.png)](/web/20220629004727/https://www.baeldung.com/wp-content/uploads/2020/02/Screenshot-2020-02-19-at-22.46.30.png)

## 3.在 Spring Web 应用程序中嵌入崩溃

让我们将 CRaSH 嵌入到 Spring web 应用程序中。首先，我们需要一些依赖关系:

```java
<dependency>
    <groupId>org.crashub</groupId>
    <artifactId>crash.embed.spring</artifactId>
    <version>1.3.2</version>
</dependency>
<dependency>
    <groupId>org.crashub</groupId>
    <artifactId>crash.cli</artifactId>
    <version>1.3.2</version>
</dependency>
<dependency>
    <groupId>org.crashub</groupId>
    <artifactId>crash.connectors.telnet</artifactId>
    <version>1.3.2</version>
</dependency>
```

我们可以在 [Maven Central](https://web.archive.org/web/20220629004727/https://search.maven.org/search?q=g:org.crashub) 查看最新版本。

CRaSH 同时支持 Java 和 Groovy，所以我们需要添加 Groovy 来让 Groovy 脚本工作:

```java
<dependency>
    <groupId>org.codehaus.groovy</groupId>
    <artifactId>groovy</artifactId>
    <version>3.0.0-rc-3</version>
</dependency>
```

它的最新版本也在 [Maven Central](https://web.archive.org/web/20220629004727/https://search.maven.org/search?q=g:org.codehaus.groovy%20a:groovy) 中。

接下来，我们需要在我们的`web.xml:`中添加一个监听器

```java
<listener>
    <listener-class>org.crsh.plugin.WebPluginLifeCycle</listener-class>
</listener>
```

现在监听器已经准备好了，让我们在`WEB-INF`目录中添加属性和命令。我们将创建一个名为`crash`的目录，并将命令和属性放入其中:

[![](img/85202130bdf1cb1e612d8cf5ff7c1eeb.png)](/web/20220629004727/https://www.baeldung.com/wp-content/uploads/2020/02/Screenshot-2020-02-19-at-22.47.01.png)

部署应用程序后，我们可以通过 telnet 连接到 shell:

```java
telnet localhost 5000 
```

我们可以使用`crash.telnet.port` 属性更改`crash.properties `文件中的 telnet 端口。

或者，我们也可以创建一个 Spring bean 来配置属性并覆盖命令的目录位置:

```java
<bean class="org.crsh.spring.SpringWebBootstrap">
    <property name="cmdMountPointConfig" value="war:/WEB-INF/crash/commands/" />
    <property name="confMountPointConfig" value="war:/WEB-INF/crash/" />
    <property name="config">
        <props>
             <prop key="crash.telnet.port">5000</prop>
         </props>
     </property>
</bean>
```

## 4.克拉舍和 Spring Boot

Spring Boot 曾经通过远程外壳将 CRaSH 作为嵌入式产品出售:

```java
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-remote-shell</artifactId>
</dependency>
```

不幸的是，该支持现在已被否决。如果我们仍然想将 shell 与 Spring Boot 应用程序一起使用，我们可以使用附加模式。在附加模式下，崩溃挂钩到 Spring Boot 应用程序的 JVM，而不是它自己的 JVM:

```java
crash.sh <PID>
```

这里，< `PID>` 那个 JVM 实例的进程 id。我们可以使用 `jps` 命令来检索在主机上运行的 JVM 的进程 id。

## 5.创建自定义命令

现在，让我们为我们的崩溃外壳创建一个自定义命令。有两种方法可以创建和使用命令；一个使用 Groovy，另一个使用 Java。我们会逐一调查。

### 5.1.Groovy 命令

首先，让我们用 Groovy 创建一个简单的命令:

```java
class message {

    @Usage("show my own message")
    @Command
    Object main(@Usage("custom message") @Option(names=["m","message"]) String message) {
        if (message == null) {
            message = "No message given...";
        }
        return message;
    }
}
```

`@Command`注释将方法标记为命令，`@Usage`用于显示命令的用法和参数，最后，`@Option`用于将任何参数传递给命令。

让我们测试一下这个命令:

[![](img/4e59346be0b52c561fc1d9cd8297f832.png)](/web/20220629004727/https://www.baeldung.com/wp-content/uploads/2020/02/Screenshot-2020-02-19-at-22.49.27.png)

### 5.2.Java 命令

让我们用 Java 创建相同的命令:

```java
public class message2 extends BaseCommand {
    @Usage("show my own message using java")
    @Command
    public Object main(@Usage("custom message") 
      @Option(names = { "m", "message" }) String message) {
        if (message == null) {
            message = "No message given...";
        }
        return message;
    }
}
```

该命令类似于 Groovy 的命令，但是这里我们需要扩展`org.crsh.command.BaseCommand.`

那么，让我们再测试一次:

[![](img/d60e10baadcb188e1c63927bb2577ba1.png)](/web/20220629004727/https://www.baeldung.com/wp-content/uploads/2020/02/Screenshot-2020-02-19-at-22.49.55.png)

## 6.结论

在本教程中，我们将 CRaSH 作为一个独立的应用程序来安装，并将其嵌入到 Spring web 应用程序中。此外，我们用 Groovy 和 Java 创建了自定义命令。

和往常一样，这段代码可以在 GitHub 上找到。