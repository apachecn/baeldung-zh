# 解耦 Java 模块的设计策略

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-modules-decoupling-design-strategies>

## 1。概述

Java 平台模块系统(JPMS)提供了更强的封装、更高的可靠性和更好的关注点分离。

但是所有这些便利的功能都是有代价的。由于模块化的应用程序是建立在模块网络之上的，而这些模块又依赖于其他模块才能正常工作，因此在许多情况下，这些模块之间是紧密耦合的。

这可能会让我们认为模块化和松耦合是不能在同一个系统中共存的特性。但实际上，他们可以！

在本教程中，我们将深入探讨两种众所周知的设计模式，它们可以用来轻松地解耦 Java 模块。

## 2。父模块

为了展示我们将用于解耦 Java 模块的设计模式，我们将构建一个演示多模块 Maven 项目。

为了保持代码简单，项目最初将包含两个 Maven 模块，并且每个 Maven 模块将被包装到一个 Java 模块中。

第一个模块将包括一个服务接口，以及两个实现——服务提供者。第二个模块将使用提供者来解析一个`String`值。

让我们从创建名为`demoproject`的项目根目录开始，我们将定义项目的父 POM:

```java
<packaging>pom</packaging>

<modules>
    <module>servicemodule</module>
    <module>consumermodule</module>
</modules>

<build>
    <pluginManagement>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <version>3.8.1</version>
                <configuration>
                    <source>11</source>
                    <target>11</target>
                </configuration>
            </plugin>
        </plugins>
    </pluginManagement>
</build>
```

在父 POM 的定义中有几个值得强调的细节。

首先，**文件包括我们上面提到的两个子模块**，即`servicemodule`和`consumermodule`(我们将在后面详细讨论)。

接下来，由于我们使用的是 Java 11，**我们的系统至少需要 Maven 3.5.0 版本`,` ，因为 Maven 支持 Java 9 及更高版本**。

最后，我们还需要至少 3.8.0 版本的 [Maven 编译器插件](/web/20221206202751/https://www.baeldung.com/maven-compiler-plugin)。因此，为了确保我们是最新的，我们将检查 [Maven Central](https://web.archive.org/web/20221206202751/https://search.maven.org/classic/#search%7Cga%7C1%7Cg%3A%22org.apache.maven.plugins%22%20AND%20a%3A%22maven-compiler-plugin%22) 以获得最新版本的 Maven 编译器插件。

## 3.服务模块

出于演示的目的，让我们使用一种快速而简单的方法来实现`servicemodule`模块，这样我们就可以清楚地发现这个设计中出现的缺陷。

让我们将**服务接口和服务提供者公开**，将它们放在同一个包中，并导出它们。这似乎是一个相当好的设计选择，但是正如我们马上会看到的，**它极大地增加了项目模块之间的耦合程度。**

在项目的根目录下，我们将创建`servicemodule/src/main/java`目录。然后，我们需要定义包`com.baeldung.servicemodule`，并在其中放置下面的`TextService`接口:

```java
public interface TextService {

    String processText(String text);

} 
```

`TextService`接口非常简单，所以现在让我们定义服务提供者。

在同一个包中，让我们添加一个`Lowercase` 实现:

```java
public class LowercaseTextService implements TextService {

    @Override
    public String processText(String text) {
        return text.toLowerCase();
    }

} 
```

现在，让我们添加一个`Uppercase` 实现:

```java
public class UppercaseTextService implements TextService {

    @Override
    public String processText(String text) {
        return text.toUpperCase();
    }

} 
```

最后，在`servicemodule/src/main/java`目录下，让我们包含模块描述符`module-info.java`:

```java
module com.baeldung.servicemodule {
    exports com.baeldung.servicemodule;
} 
```

## 4。消费者模块

现在我们需要创建一个消费者模块，它使用我们之前创建的服务提供者之一。

让我们添加下面的`com.baeldung.consumermodule.` `Application`类:

```java
public class Application {
    public static void main(String args[]) {
        TextService textService = new LowercaseTextService();
        System.out.println(textService.processText("Hello from Baeldung!"));
    }
}
```

现在，让我们将模块描述符`module-info.java,` 包含在源根中，它应该是`consumermodule/src/main/java`:

```java
module com.baeldung.consumermodule {
    requires com.baeldung.servicemodule;
} 
```

最后，让我们编译源文件并运行应用程序，无论是从 IDE 中还是从命令控制台。

正如我们所料，我们应该会看到以下输出:

```java
hello from baeldung!
```

这肯定是可行的，但是有一个重要的警告值得注意:**我们不必要地将服务提供者耦合到消费者模块**。

由于我们让提供者对外界可见，消费者模块知道他们。

此外，这与使软件组件依赖于抽象相冲突。

## 5。服务提供商工厂

通过只导出服务接口，我们可以很容易地**去除模块之间的耦合。相比之下，服务提供者没有被导出，因此对消费者模块保持隐藏。消费者模块只看到服务接口类型。**

为此，我们需要:

1.  将服务接口放在一个单独的包中，该包被导出到外部世界
2.  将服务提供程序放在不同的包中，该包不会被导出
3.  创建一个导出的工厂类。消费者模块使用工厂类来查找服务提供者

我们可以用设计模式的形式概念化上面的步骤:**公共服务接口，私有服务提供者，公共服务提供者工厂**。

### 5.1。公共服务接口

为了清楚地了解这种模式是如何工作的，让我们将服务接口和服务提供者放在不同的包中。接口将被导出，但提供者实现不会。

所以，让我们把`TextService` 移到一个新的包中，我们称之为`com.baeldung.servicemodule.external`。

### 5.2。私人服务提供商

然后，让我们同样将我们的`LowercaseTextService` 和`UppercaseTextService` 移动到`com.baeldung.servicemodule.internal.`

### 5.3。公共服务提供商工厂

由于服务提供者类现在是私有的，不能从其他模块访问，**我们将使用一个公共工厂类来提供一个简单的机制，消费者模块可以使用它来获取服务提供者的实例**。

在`com.baeldung.servicemodule.external`包中，让我们定义下面的`TextServiceFactory`类:

```java
public class TextServiceFactory {

    private TextServiceFactory() {}

    public static TextService getTextService(String name) {
        return name.equalsIgnoreCase("lowercase") ? new LowercaseTextService(): new UppercaseTextService();
    }

}
```

当然，我们可以让工厂类稍微复杂一些。为了简单起见，服务提供者只是基于传递给`getTextService()`方法的`String`值来创建的。

现在，让我们替换我们的`module-info.java`文件，只导出我们的`external `包:

```java
module com.baeldung.servicemodule {
    exports com.baeldung.servicemodule.external;
}
```

**注意，我们只导出了服务接口和工厂类**。这些实现是私有的，因此它们对其他模块不可见。

### 5.4。应用程序类

现在，让我们重构*应用程序*类，这样它就可以使用服务提供者工厂类:

```java
public static void main(String args[]) {
    TextService textService = TextServiceFactory.getTextService("lowercase");
    System.out.println(textService.processText("Hello from Baeldung!"));
} 
```

正如预期的那样，如果我们运行应用程序，我们应该看到控制台上打印出相同的文本:

```java
hello from baeldung!
```

通过将服务接口公共化，将服务提供者私有化，我们可以通过一个简单的工厂类有效地分离服务和消费者模块。

当然，没有一种模式是万能的。和往常一样，我们应该首先分析我们的用例是否合适。

## 6。服务和消费者模块

JPMS 通过 `provides…with`和`uses`指令为开箱即用的服务和消费者模块提供支持。

因此，我们可以使用这个功能来解耦模块，而不必创建额外的工厂类。

要使服务和消费者模块协同工作，我们需要做以下工作:

1.  将服务接口放在一个模块中，该模块导出接口
2.  将服务提供者放在另一个模块中–提供者被导出
3.  在提供者的模块描述符中指定我们想要用`provides…with`指令提供一个`TextService`实现
4.  将`Application`类放在它自己的模块——消费者模块中
5.  在消费者模块的模块描述符中指定该模块是带有`uses`指令的消费者模块
6.  使用消费者模块中的[服务加载器 API](/web/20221206202751/https://www.baeldung.com/java-spi) 来查找服务提供者

这种方法非常强大，因为它利用了服务和消费者模块带来的所有功能。但这也有点棘手。

一方面，我们使消费者模块只依赖于服务接口，而不依赖于服务提供者。另一方面，**我们甚至可以根本不定义服务提供者，应用程序仍然会编译**。

### 6.1。父模块

为了实现这个模式，我们还需要重构父 POM 和现有的模块。

由于服务接口、服务提供者和消费者现在将生活在不同的模块中，我们首先需要修改父 POM 的`<modules>`部分，以反映这种新的结构:

```java
<modules>
    <module>servicemodule</module>
    <module>providermodule</module>
    <module>consumermodule</module>
</modules>
```

### 6.2。服务模块

我们的`TextService` 界面将返回到`com.baeldung.servicemodule.`

我们将相应地更改模块描述符:

```java
module com.baeldung.servicemodule {
    exports com.baeldung.servicemodule;
}
```

### 6.3。供应商模块

如上所述，提供者模块是用于我们的实现的，所以现在让我们将`LowerCaseTextService`和 U `ppercaseTextService`放在这里。我们会把它们放在一个叫做`com.baeldung.providermodule.`的包里

最后，我们来添加一个`module-info.java`文件:

```java
module com.baeldung.providermodule {
    requires com.baeldung.servicemodule;
    provides com.baeldung.servicemodule.TextService with com.baeldung.providermodule.LowercaseTextService;
} 
```

### 6.4。消费者模块

现在，让我们重构消费者模块。首先，我们将把`Application`放回`com.baeldung.consumermodule`包中。

接下来，我们将重构`Application`类的`main()`方法，这样它就可以使用 [`ServiceLoader`](https://web.archive.org/web/20221206202751/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/ServiceLoader.html) 类来发现适当的实现:

```java
public static void main(String[] args) {
    ServiceLoader<TextService> services = ServiceLoader.load(TextService.class);
    for (final TextService service: services) {
        System.out.println("The service " + service.getClass().getSimpleName() + 
            " says: " + service.parseText("Hello from Baeldung!"));
    }
}
```

最后，我们将重构`module-info.java`文件:

```java
module com.baeldung.consumermodule {
    requires com.baeldung.servicemodule;
    uses com.baeldung.servicemodule.TextService;
} 
```

现在，让我们运行应用程序。正如所料，我们应该看到控制台输出了以下文本:

```java
The service LowercaseTextService says: hello from baeldung! 
```

正如我们所看到的，实现这种模式比使用工厂类的模式稍微复杂一些。尽管如此，额外的努力会得到更灵活、松耦合设计的高度回报。

消费者模块依赖于抽象，在运行时很容易加入不同的服务提供者。

## 7。结论

在本教程中，我们学习了如何实现两种模式来分离 Java 模块。

这两种方法都使得消费者模块依赖于抽象，这在软件组件的设计中总是一个期望的特性。

当然，各有利弊。对于第一个，我们得到了一个很好的解耦，但是我们必须创建一个额外的工厂类。

对于第二个，为了使模块解耦，我们必须创建一个额外的抽象模块，并使用服务加载器 API 添加一个新的间接级[。](https://web.archive.org/web/20221206202751/https://en.wikipedia.org/wiki/Indirection)

像往常一样，本教程中显示的所有示例都可以在 GitHub 上获得[。](https://web.archive.org/web/20221206202751/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-jpms)