# Java 9 模块化指南

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-9-modularity>

## 1。概述

Java 9 在包之上引入了一个新的抽象层次，正式名称是 Java 平台模块系统(Java Platform Module System，JPMS)，简称“模块”。

在本教程中，我们将通过新的系统，并讨论其各个方面。

我们还将构建一个简单的项目来演示我们将在本指南中学习的所有概念。

## 2。什么是模块？

首先，我们需要了解模块是什么，然后才能了解如何使用它们。

模块是一组紧密相关的包和资源以及一个新的模块描述符文件。

换句话说，它是一个“Java 包的包”抽象，允许我们使我们的代码更加可重用。

### 2.1。包装

模块中的包与我们从 Java 诞生以来一直使用的 Java 包是一样的。

当我们创建一个模块时，我们在内部将代码组织在包中，就像我们以前对任何其他项目所做的那样。

除了组织我们的代码之外，包还用于确定哪些代码可以在模块之外公开访问。在本文的后面，我们将花更多的时间来讨论这个问题。

### 2.2。资源

每个模块负责其资源，如媒体或配置文件。

以前，我们将所有资源放入项目的根级别，并手动管理哪些资源属于应用程序的不同部分。

有了模块，我们可以将需要的图像和 XML 文件与需要它的模块一起发送，使我们的项目更容易管理。

### 2.3。模块描述符

当我们创建一个模块时，我们包括一个描述符文件，它定义了新模块的几个方面:

*   **名称**–我们模块的名称
*   **依赖关系**–该模块所依赖的其他模块的列表
*   **公共包**–我们希望从模块外部访问的所有包的列表
*   **提供的服务**–我们可以提供其他模块可以使用的服务实现
*   **消费的服务**–允许当前模块成为服务的消费者
*   **反射权限**–明确允许其他类使用反射来访问包的私有成员

模块命名规则类似于我们命名包的方式(允许点号，不允许破折号)。使用项目风格(my.module)或反向 DNS ( `com.baeldung.mymodule`)风格的名称是很常见的。在本指南中，我们将使用项目风格。

我们需要列出所有我们想要公开的包，因为默认情况下所有的包都是模块私有的。

反思也是如此。默认情况下，我们不能在从另一个模块导入的类上使用反射。

在本文的后面，我们将看看如何使用模块描述符文件的例子。

### 2.4。模块类型

新模块系统中有四种类型的模块:

*   **系统模块**–这些是我们运行上面的`list-modules`命令时列出的模块。它们包括 Java SE 和 JDK 模块。
*   **应用模块**——当我们决定使用模块时，这些模块通常是我们想要构建的。它们在汇编 JAR 中包含的编译后的`module-info.class` 文件中被命名和定义。
*   **自动模块**–我们可以通过将现有的 JAR 文件添加到模块路径中来包含非官方模块。模块的名称将来自于 JAR 的名称。自动模块将对该路径加载的所有其他模块拥有完全读取权限。
*   **未命名模块**–当一个类或 JAR 加载到类路径而不是模块路径时，它会自动添加到未命名模块中。它是一个包罗万象的模块，用来保持与以前编写的 Java 代码的向后兼容性。

### 2.5。分布

模块可以通过两种方式分发:作为一个 JAR 文件或者作为一个“分解”的编译项目。当然，这与任何其他 Java 项目都是一样的，所以这不足为奇。

我们可以创建由一个“主应用程序”和几个库模块组成的多模块项目。

我们必须小心，因为每个 JAR 文件只能有一个模块。

当我们建立我们的构建文件时，我们需要确保将项目中的每个模块捆绑成一个单独的 jar。

## 3。默认模块

当我们安装 Java 9 时，我们可以看到 JDK 现在有了一个新的结构。

他们已经把所有的原始包转移到新的模块系统中。

我们可以通过在命令行中键入以下命令来查看这些模块:

```java
java --list-modules
```

这些模块分为四大组:`java, javafx, jdk, `和`Oracle`。

模块是核心 SE 语言规范的实现类。

模块是 FX UI 库。

**JDK 本身需要的任何东西都保存在`jdk`模块中。**

最后，**任何 Oracle 特有的东西都在`oracle`模块中。**

## 4。模块声明

**要建立一个模块，我们需要在包的根目录下放一个名为`module-info.java`的特殊文件。**

这个文件被称为模块描述符，包含构建和使用新模块所需的所有数据。

我们用声明来构造模块，声明的主体要么为空，要么由模块指令组成:

```java
module myModuleName {
    // all directives are optional
}
```

我们以关键字`module`开始模块声明，然后是模块名。

这个模块将使用这个声明，但是我们通常需要更多的信息。

这就是模块指令的用武之地。

### 4.1。需要

我们的第一个指令是`requires`。这个模块指令允许我们声明模块依赖关系:

```java
module my.module {
    requires module.name;
}
```

现在，`my.module`对`module.name`既有运行时依赖，也有编译时依赖**。**

当我们使用这个指令时，我们的模块可以访问从依赖项导出的所有公共类型。

### 4.2。需要静态

有时，我们编写的代码引用了另一个模块，但是我们库的用户永远也不想使用它。

例如，我们可以编写一个实用函数，当另一个日志模块存在时，它可以很好地显示我们的内部状态。但是，并不是我们库的每个消费者都想要这个功能，他们也不想包含额外的日志库。

在这些情况下，我们希望使用一个可选的依赖项。通过使用`requires static`指令，我们创建了一个仅编译时依赖关系:

```java
module my.module {
    requires static module.name;
}
```

### 4.3。需要可传递的

我们通常与图书馆合作，以使我们的生活更容易。

但是，我们需要确保任何引入我们代码的模块也会引入这些额外的“可传递”依赖项，否则它们将不起作用。

幸运的是，我们可以使用`requires transitive`指令来强制任何下游消费者也读取我们所需的依赖项:

```java
module my.module {
    requires transitive module.name;
}
```

现在，当一个开发者`requires my.module`时，他们也不需要说`requires module.name`来让我们的模块继续工作。

### 4.4。出口

默认情况下，一个模块不会向其他模块公开它的任何 API。这个`strong encapsulation`是最初创建模块系统的关键动力之一。

我们的代码要安全得多，但是现在我们需要明确地向世界开放我们的 API，如果我们想让它可用的话。

**我们使用`exports`指令来公开命名包的所有公共成员:**

```java
module my.module {
    exports com.my.package.name;
}
```

现在，当有人做`requires my.module`时，他们将可以访问我们的`com.my.package.name`包中的公共类型，但不能访问任何其他包。

### 4.5。出口…至

**我们可以使用`exports…to`向世界开放我们的公共类。**

但是，如果我们不想让全世界都访问我们的 API 呢？

**我们可以使用`exports…to`指令来限制哪些模块可以访问我们的 API。**

类似于`exports`指令，我们将一个包声明为 exported。但是，我们也列出了我们允许哪些模块作为`requires`导入这个包。让我们看看这个是什么样子的:

```java
module my.module {
    export com.my.package.name to com.specific.package;
}
```

### 4.6。用途

一个`service`是一个特定接口或抽象类的实现，可以被其他类`consumed`使用。

**我们用`uses`指令指定模块消费的服务。**

注意**我们`use`的类名是服务的接口或者抽象类，而不是实现类**:

```java
module my.module {
    uses class.name;
}
```

这里我们应该注意到,`requires`指令和`uses`指令是有区别的。

我们可能`require`一个提供我们想要消费的服务的模块，但是那个服务实现了一个来自它的可传递依赖的接口。

为了以防万一，我们没有强迫我们的模块要求`all`传递依赖，而是使用`uses`指令将所需的接口添加到模块路径中。

### 4.7。为…提供

**一个模块也可以是其他模块可以消费的`service provider`。**

指令的第一部分是关键字`provides`。这里是我们放置接口或抽象类名的地方。

接下来，我们有了`with`指令，其中我们提供了实现类名，或者是`implements`接口或者是`extends`抽象类。

这是它看起来的样子:

```java
module my.module {
    provides MyInterface with MyInterfaceImpl;
}
```

### 4.8。打开

我们前面提到过，封装是这个模块系统设计的驱动因素。

在 Java 9 之前，可以使用反射来检查包中的每个类型和成员，甚至是`private`类型和成员。没有什么是真正封装的，这可能会给库的开发人员带来各种各样的问题。

因为 Java 9 强制执行`strong encapsulation`，**，我们现在必须显式地授予其他模块对我们的类进行反射的权限。**

如果我们想继续像旧版本的 Java 那样允许完全反射，我们可以简单地将整个模块:

```java
open module my.module {
}
```

### 4.9。打开

如果我们需要允许私有类型的反射，但是我们不想暴露所有的代码，**我们可以使用`opens`指令来暴露特定的包。**

但是请记住，这将向整个世界打开这个包，所以请确保这是您想要的:

```java
module my.module {
  opens com.my.package;
}
```

### 4.10。打开…至

好吧，有时候反思是很好的，但是我们仍然希望从`encapsulation`中获得尽可能多的安全。**我们可以有选择地向预先批准的模块列表打开我们的包，在这种情况下，使用`opens…to`指令**:

```java
module my.module {
    opens com.my.package to moduleOne, moduleTwo, etc.;
}
```

## 5。命令行选项

到目前为止，对 Java 9 模块的支持已经被添加到 Maven 和 Gradle 中，所以您不需要手工构建大量的项目。不过知道`how`从命令行使用模块系统还是很有价值的。

我们将在下面的完整示例中使用命令行来帮助巩固整个系统在我们头脑中的工作方式。

*   `**module-path**`**–**我们使用`–module-path`选项来指定模块路径。这是包含模块的一个或多个目录的列表。
*   `**add-reads**`–不依赖于模块声明文件，我们可以使用等同于`requires`指令的命令行；`–add-reads`。
*   `**add-exports**`**–`exports`指令的**命令行替换。
*   `**add-opens**` `– `替换模块声明文件中的`open`子句。
*   `**add-modules**` `– `将模块列表添加到默认的模块集中
*   `**list-modules**` `– `打印所有模块及其版本字符串的列表
*   `**patch-module**`–在模块中添加或覆盖类
*   `**illegal-access=permit|warn|deny**`–要么通过显示单个全局警告来放松强封装，要么显示每个警告，要么出现错误而失败。默认为`permit`。

## 6。能见度

我们应该花点时间讨论一下代码的可见性。

许多库依靠反射来施展他们的魔法(想到 JUnit 和 Spring)。

默认情况下，在 Java 9 中，我们可以访问导出包中的公共类、方法和字段。即使我们使用反射来访问非公共成员并调用`setAccessible(true), `,我们也无法访问这些成员。

我们可以使用`open`、`opens`和`opens…to`选项来授予反射的仅运行时访问权。注意，**这只是运行时的！**

我们将无法针对私有类型进行编译，而且我们也永远不需要这样做。

如果我们必须访问一个模块进行反射，而我们不是该模块的所有者(也就是说，我们不能使用`opens…to`指令)，那么可以使用命令行`–add-opens`选项来允许自己的模块在运行时反射访问锁定的模块。

这里唯一的警告是，您需要访问用于运行模块的命令行参数，这样才能工作。

## 7。将所有这些放在一起

现在我们知道了什么是模块以及如何使用它们，让我们继续构建一个简单的项目来演示我们刚刚学到的所有概念。

为了简单起见，我们不会使用 Maven 或 Gradle。相反，我们将依赖命令行工具来构建我们的模块。

### 7.1。设置我们的项目

首先，我们需要建立我们的项目结构。我们将创建几个目录来组织我们的文件。

首先创建项目文件夹:

```java
mkdir module-project
cd module-project
```

这是我们整个项目的基础，所以在这里添加文件，如 Maven 或 Gradle 构建文件，其他源目录和资源。

我们还放了一个目录来保存我们所有的项目特定模块。

接下来，我们创建一个模块目录:

```java
mkdir simple-modules
```

下面是我们的项目结构:

```java
module-project
|- // src if we use the default package
|- // build files also go at this level
|- simple-modules
  |- hello.modules
    |- com
      |- baeldung
        |- modules
          |- hello
  |- main.app
    |- com
      |- baeldung
        |- modules
          |- main
```

### 7.2。我们的第一个模块

现在我们已经有了基本的结构，让我们添加第一个模块。

在`simple-modules `目录下，创建一个名为`hello.modules`的新目录。

我们可以给它起任何我们想要的名字，但是要遵循包命名规则(例如，用句点来分隔单词，等等。).如果我们愿意，我们甚至可以使用主包的名称作为模块名称，但是通常，我们希望坚持使用我们用来创建这个模块的 JAR 的名称。

在我们的新模块下，我们可以创建我们想要的包。在我们的例子中，我们将创建一个包结构:

```java
com.baeldung.modules.hello
```

接下来，在这个包中创建一个名为`HelloModules.java`的新类。我们将保持代码简单:

```java
package com.baeldung.modules.hello;

public class HelloModules {
    public static void doSomething() {
        System.out.println("Hello, Modules!");
    }
}
```

最后，在`hello.modules`根目录中，添加我们的模块描述符；`module-info.java`:

```java
module hello.modules {
    exports com.baeldung.modules.hello;
}
```

为了使这个例子简单，我们所做的就是导出`com.baeldung.modules.hello `包的所有公共成员。

### 7.3。我们的第二个模块

我们的第一个模块很棒，但是它什么都不做。

我们现在可以创建使用它的第二个模块。

在我们的`simple-modules`目录下，创建另一个名为`main.app`的模块目录。这次我们将从模块描述符开始:

```java
module main.app {
    requires hello.modules;
}
```

我们不需要向外界暴露任何东西。相反，我们需要做的就是依赖我们的第一个模块，这样我们就可以访问它导出的公共类。

现在我们可以创建一个使用它的应用程序。

创建新的包结构:`com.baeldung.modules.main`。

现在，创建一个名为`MainApp.java.`的新类文件

```java
package com.baeldung.modules.main;

import com.baeldung.modules.hello.HelloModules;

public class MainApp {
    public static void main(String[] args) {
        HelloModules.doSomething();
    }
}
```

这就是我们演示模块所需的全部代码。我们的下一步是从命令行构建并运行这段代码。

### 7.4。构建我们的模块

为了构建我们的项目，我们可以创建一个简单的 bash 脚本，并将其放在项目的根目录下。

创建一个名为`compile-simple-modules.sh`的文件:

```java
#!/usr/bin/env bash
javac -d outDir --module-source-path simple-modules $(find simple-modules -name "*.java")
```

该命令有两部分，即`javac`和`find`命令。

`find`命令只是输出所有。我们的简单模块目录下的文件。然后我们可以将这个列表直接输入到 Java 编译器中。

与旧版本的 Java 不同，我们唯一要做的是提供一个`module-source-path`参数来通知编译器它正在构建模块。

一旦我们运行这个命令，我们将有一个`outDir`文件夹，里面有两个编译好的模块。

### 7.5。运行我们的代码

现在，我们终于可以运行代码来验证模块是否正常工作了。

在项目的根目录下创建另一个文件:`run-simple-module-app.sh`。

```java
#!/usr/bin/env bash
java --module-path outDir -m main.app/com.baeldung.modules.main.MainApp
```

要运行一个模块，我们必须至少提供`module-path`和主类。如果一切正常，您应该会看到:

```java
>$ ./run-simple-module-app.sh 
Hello, Modules!
```

### 7.6。添加服务

现在我们已经对如何构建一个模块有了基本的了解，让我们把它变得稍微复杂一点。

我们将看到如何使用`provides…with`和`uses`指令。

首先在`hello.modules`模块中定义一个名为`HelloInterface` `.java`的新文件:

```java
public interface HelloInterface {
    void sayHello();
}
```

为了简单起见，我们将使用现有的`HelloModules.java`类来实现这个接口:

```java
public class HelloModules implements HelloInterface {
    public static void doSomething() {
        System.out.println("Hello, Modules!");
    }

    public void sayHello() {
        System.out.println("Hello!");
    }
}
```

这就是我们创建一个`service`所需要做的一切。

现在，我们需要告诉世界，我们的模块提供这种服务。

将以下内容添加到我们的`module-info.java`:

```java
provides com.baeldung.modules.hello.HelloInterface with com.baeldung.modules.hello.HelloModules;
```

正如我们所看到的，我们声明了接口和实现它的类。

接下来，我们需要消耗这个`service`。在我们的`main.app`模块中，让我们将以下内容添加到我们的`module-info.java`中:

```java
uses com.baeldung.modules.hello.HelloInterface;
```

最后，在我们的 main 方法中，我们可以通过一个 [ServiceLoader](https://web.archive.org/web/20220926202208/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/ServiceLoader.html) 来使用这个服务:

```java
Iterable<HelloInterface> services = ServiceLoader.load(HelloInterface.class);
HelloInterface service = services.iterator().next();
service.sayHello();
```

编译并运行:

```java
#> ./run-simple-module-app.sh 
Hello, Modules!
Hello!
```

我们使用这些指令来更加明确地说明如何使用我们的代码。

我们可以将实现放在私有包中，而将接口放在公共包中。

这使得我们的代码更加安全，而几乎没有额外的开销。

继续尝试其他一些指令，以了解更多关于模块及其工作方式的信息。

## 8.向未命名的模块添加模块

**未命名模块的概念类似于默认包。**因此，它不被认为是一个真正的模块，但可以被视为默认模块。

如果一个类不是一个已命名模块的成员，那么它将被自动视为这个未命名模块的一部分。

有时，为了确保特定的平台、库或服务提供者模块出现在模块图中，我们需要将模块添加到默认的根集中。例如，当我们试图用 Java 9 编译器运行 Java 8 程序时，我们可能需要添加模块。

一般来说，**将命名模块添加到默认根模块集中的选项是** `**–add-modules <module>**(,<module>)*`，其中`<module>`是模块名。

例如，要提供对所有`java.xml.bind`模块的访问，语法应该是:

```java
--add-modules java.xml.bind
```

为了在 Maven 中使用它，我们可以将其嵌入到`maven-compiler-plugin`:

```java
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-compiler-plugin</artifactId>
    <version>3.8.0</version>
    <configuration>
        <source>9</source>
        <target>9</target>
        <compilerArgs>
            <arg>--add-modules</arg>
            <arg>java.xml.bind</arg>
        </compilerArgs>
    </configuration>
</plugin>
```

## 9。结论

在这本内容丰富的指南中，我们重点介绍了新的 Java 9 模块系统的基础知识。

我们从讨论什么是模块开始。

接下来，我们讨论了如何发现 JDK 中包含哪些模块。

我们还详细介绍了模块声明文件。

我们讨论了构建模块所需的各种命令行参数，从而完善了这一理论。

最后，我们将前面的知识付诸实践，并在模块系统的基础上创建了一个简单的应用程序。

要查看这段代码和更多内容，请务必在 Github 上查看。