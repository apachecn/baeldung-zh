# 使用命令行编译多个 Java 源文件

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-compile-multiple-files>

## 1.概观

在本教程中，我们将学习如何通过命令行界面与 Java 编译器交互。

作为先决条件，我们需要下载 [Java](/web/20221208143832/https://www.baeldung.com/java-check-is-installed) 并在我们的机器中配置 [JAVA_HOME](https://web.archive.org/web/20221208143832/https://baeldung.com/maven-java-home-jdk-jre) 环境变量。

## 2.**编译单个 Java 源代码文件**

java 提供了一个简单的工具——[`javac`](/web/20221208143832/https://www.baeldung.com/javac)来编译 Java 源代码文件。先来编一个小类，`Car.java`:

```java
public class Car {
    private String make;
    private String model;

   // standard setters and getters
} 
```

我们可以在这个文件所在的目录中用一个命令来编译它:

```java
javac Car.java
```

如果一切正常，就不会有产出。编译器将在当前工作目录中创建包含字节码的`Car.class,` 。

## 3.编译多个源代码文件

通常，我们的程序使用不止一个类文件。现在让我们看看如何用多个类编译一个简单的程序。

首先，让我们添加两个新类型，`Owner.java`和`History.java`:

```java
public class Car {
    private String make;
    private String model;
    private Owner owner;
    private History history;
} 
```

```java
public class Owner {
    private String name;
} 
```

```java
public class History {
    private String details;
} 
```

现在，我们需要运行下面的命令来编译:

```java
javac Owner.java Car.java History.java
```

**我们应该注意，由于`Car`类使用的类在同一个目录中，我们是否指定它们实际上是可选的。**我们仍然可以只编译`Car.java`。

## 4.基本 Java 编译器选项

到目前为止，我们只是使用了 javac 命令，没有任何额外的选项，只是将类名作为参数传递。但是，我们也可以自定义它。我们可以告诉 java 编译器在哪里找到我们库的类，我们代码驻留的基本路径，以及在哪里生成最终结果。

让我们仔细看看其中的一些选项。

*   `-cp`或`-classpath`
*   `-sourcepath`
*   `-d`(目录)

### 4.1.**什么是`-cp`或`-classpath`选项？**

使用类路径，我们可以定义一组目录或文件，比如`*.jar`、`*.zip`，我们的源代码在编译期间依赖于这些目录或文件。或者，我们可以设置`CLASSPATH`环境变量。

我们应该注意到**class path 选项比环境变量**具有更高的优先级。

如果都没有指定，则假定类路径是当前目录。当我们希望指定多个目录时，对于大多数操作系统，路径分隔符是“`:`”，但 Windows 除外，在 Windows 中是“`;`”。

### 4.2.什么是`-sourcepath`选项？

这个选项可以指定所有需要编译的源代码所在的顶层目录。

如果未指定，将扫描类路径中的源。

### 4.3.什么是`-d`选项？

当我们想把所有编译结果放在一个地方，与源代码分开时，我们使用这个选项。**我们需要记住，我们想要指定的路径必须预先存在**。

在编译期间，该路径被用作根目录，并且根据类的包结构自动创建子文件夹。如果没有指定这个选项，每个单独的`*.class`文件都写在它们对应的源代码`*.java`文件旁边。

## 5.使用外部库编译

除了我们创建的类，我们还需要在程序中使用外部库。现在让我们看一个更复杂的例子:

```java
libs/
├─ guava-31.1-jre.jar
model/
├─ Car.java
├─ History.java
├─ Owner.java
service/
├─ CarService.java
target/
```

在这里，我们已经将我们的类组织成包。此外，我们还引入了`target`和`libs`目录来分别存放编译后的结果和库。

假设我们想要使用由[番石榴](https://web.archive.org/web/20221208143832/https://mvnrepository.com/artifact/com.google.guava/guava)库提供的`ImmutableSet `类。我们下载并把它放在`libs`文件夹下。然后，在`service `包下，我们引入了一个新类，它使用了`CarService.java`中的外部库:

```java
package service;

import model.Car;
import java.util.Set;

import com.google.common.collect.ImmutableSet;

public class CarService {

    public Set<Car> getCars() {

        Car car1 = new Car();
        Car car2 = new Car();

        ImmutableSet<Car> cars = ImmutableSet.<Car>builder()
          .add(car1)
          .add(car2)
          .build();
        return cars;
    }
}
```

现在，是时候编译我们的项目了:

```java
javac -classpath libs/*:. -d target -sourcepath . service/CarService.java model/*.java
```

我们已经将`libs`文件夹包含在带有`-cp`的类路径中。

```java
libs/
├─ guava-31.1-jre.jar
model/
├─ Car.java
├─ History.java
├─ Owner.java
service/
├─ CarService.java
target/
├─ model/
│ ├─ Car.class
│ ├─ History.class
│ ├─ Owner.class
├─ service/
│ ├─ CarService.class
```

正如我们所见，`javac`成功地解析了外部的`ImmutbleSet`类，并将编译后的类放在了`target`文件夹中。

## 6.结论

在本文中，我们学习了如何编译多个源代码文件，即使我们依赖于外部库。

此外，我们快速查看了一些在复杂的编译任务中可以利用的基本选项。