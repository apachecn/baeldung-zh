# 具有 Java 模块的多模块 Maven 应用程序

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/maven-multi-module-project-java-jpms>

## 1。概述

Java 平台模块系统 (JPMS)为 Java 应用程序增加了更多的可靠性、更好的关注点分离和更强的封装。然而，它不是一个构建工具，因此**它缺乏自动管理项目依赖的能力。**

当然，我们可能想知道是否可以在模块化的应用程序中使用成熟的构建工具，比如 Maven T2 或者 grad le T4 T5。

其实我们可以！在本教程中，**我们将学习如何使用 Java 模块**创建多模块 Maven 应用程序。

## 2。在 Java 模块中封装 Maven 模块

因为**模块化和依赖性管理在 Java 中并不是互斥的概念，**我们可以无缝地集成 [JPMS](/web/20220626115805/https://www.baeldung.com/new-java-9) ，例如，与 Maven，从而利用两个世界的优点。

在标准的多模块 Maven 项目中，我们添加一个或多个子 Maven 模块，方法是将它们放在项目的根文件夹下，并在父 POM 中的`<modules>`部分中声明它们。

接下来，我们编辑每个子模块的 POM，并通过标准的<`groupId>`、<、`artifactId>`和<、`version>`坐标指定其依赖关系。

Maven 中的`reactor`机制——负责处理多模块项目——负责按照正确的顺序构建整个项目。

在这种情况下，我们将基本上使用相同的设计方法，但有一个微妙但基本的变化:**我们将通过添加模块描述符文件**、`module-info.java`将每个 Maven 模块包装到一个 Java 模块中。

## 3。父 Maven 模块

为了演示模块化和依赖性管理如何很好地协同工作，我们将构建一个基本的演示多模块 Maven 项目，**它的功能将被缩小到仅仅从持久层**获取一些域对象。

为了保持代码简单，我们将使用普通的`Map`作为存储域对象的底层数据结构。当然，我们可以很容易地进一步转向成熟的关系数据库。

让我们从定义父 Maven 模块开始。为了实现这一点，让我们创建一个名为`multimodulemavenproject`的根项目目录(但也可以是其他目录)，并向其中添加父文件`pom.xml`:

```java
<groupId>com.baeldung.multimodulemavenproject</groupId>
<artifactId>multimodulemavenproject</artifactId>
<version>1.0</version>
<packaging>pom</packaging>
<name>multimodulemavenproject</name>

<build>
    <pluginManagement>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <version>3.8.0</version>
                <configuration>
                    <source>11</source>
                    <target>11</target>
                </configuration>
            </plugin>
        </plugins>
    </pluginManagement>
</build>

<properties>
    <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
</properties>
```

在父 POM 的定义中有一些细节值得注意。

首先，由于我们使用的是 Java 11，**我们的系统至少需要 Maven 3.5.0 版本`,` ，因为 Maven 支持 Java 9 及更高版本**。

而且，我们还需要至少 3.8.0 版本的 [Maven 编译器插件](/web/20220626115805/https://www.baeldung.com/maven-compiler-plugin)。因此，让我们确保在 Maven Central 上查看[最新版本的插件。](https://web.archive.org/web/20220626115805/https://search.maven.org/classic/#search%7Cga%7C1%7Cg%3A%22org.apache.maven.plugins%22%20AND%20a%3A%22maven-compiler-plugin%22)

## 4。子 Maven 模块

注意到目前为止，**父 POM 没有声明任何子模块**。

由于我们的演示项目将从持久层获取一些域对象，我们将创建四个子 Maven 模块:

1.  `entitymodule`:将包含一个简单的域类
2.  `daomodule`:将持有访问持久层所需的接口(一个基本的[道](/web/20220626115805/https://www.baeldung.com/java-dao-pattern)契约)
3.  `userdaomodule`:将包括`daomodule`接口的实现
4.  `mainappmodule`:项目的入口点

### 4.1。`entitymodule` Maven 模块

现在，让我们添加第一个子 Maven 模块，它只包含一个基本的域类。

在项目的根目录下，让我们创建`entitymodule/src/main/java/com/baeldung/entity`目录结构并添加一个`User` 类:

```java
public class User {

    private final String name;

    // standard constructor / getter / toString

}
```

接下来，让我们包含模块的`pom.xml`文件:

```java
<parent>
    <groupId>com.baeldung.multimodulemavenproject</groupId>
    <artifactId>multimodulemavenproject</artifactId>
    <version>1.0</version>
</parent>

<groupId>com.baeldung.entitymodule</groupId>
<artifactId>entitymodule</artifactId>
<version>1.0</version>
<packaging>jar</packaging>
<name>entitymodule</name>
```

正如我们所见，`Entity` 模块不依赖于其他模块，也不需要额外的 Maven 工件，因为它只包含了`User`类。

现在，我们需要**将 Maven 模块封装成 Java 模块**。为此，让我们简单地将下面的模块描述符文件(`module-info.java`)放在`entitymodule/src/main/java`目录下:

```java
module com.baeldung.entitymodule {
    exports com.baeldung.entitymodule;
}
```

最后，让我们将子 Maven 模块添加到父 POM 中:

```java
<modules>
    <module>entitymodule</module>
</modules>
```

### 4.2。`daomodule` Maven 模块

让我们创建一个包含简单接口的新 Maven 模块。这对于定义从持久层获取泛型类型的抽象契约很方便。

事实上，有一个非常有说服力的理由将这个接口放在一个单独的 Java 模块中。通过这样做，我们有了一个抽象的、高度解耦的契约，它很容易在不同的上下文中重用。核心上，这是[依赖倒置原则](/web/20220626115805/https://www.baeldung.com/java-dependency-inversion-principle)的替代实现，它产生了一个更灵活的设计。

因此，让我们在项目的根目录下创建`daomodule/src/main/java/com/baeldung/dao`目录结构，并在其中添加`Dao<T>`接口:

```java
public interface Dao<T> {

    Optional<T> findById(int id);

    List<T> findAll();

}
```

现在，让我们定义模块的`pom.xml`文件:

```java
<parent>
    // parent coordinates
</parent>

<groupId>com.baeldung.daomodule</groupId>
<artifactId>daomodule</artifactId>
<version>1.0</version>
<packaging>jar</packaging>
<name>daomodule</name>
```

这个新模块也不需要其他模块或工件，所以我们将把它包装成一个 Java 模块。让我们在`daomodule/src/main/java`目录下创建模块描述符:

```java
module com.baeldung.daomodule {
    exports com.baeldung.daomodule;
}
```

最后，让我们将模块添加到父 POM:

```java
<modules>
    <module>entitymodule</module>
    <module>daomodule</module>
</modules> 
```

### 4.3。`userdaomodule` Maven 模块

接下来，让我们定义保存了`Dao`接口实现的 Maven 模块。

在项目的根目录下，让我们创建`userdaomodule/src/main/java/com/baeldung/userdao`目录结构，并向其中添加下面的`UserDao`类:

```java
public class UserDao implements Dao<User> {

    private final Map<Integer, User> users;

    // standard constructor

    @Override
    public Optional<User> findById(int id) {
        return Optional.ofNullable(users.get(id));
    }

    @Override
    public List<User> findAll() {
        return new ArrayList<>(users.values());
    }
}
```

**简单地说，`UserDao`类提供了一个基本的 API，允许我们从持久层获取`User`对象。**

为了简单起见，我们使用了一个`Map`作为持久化域对象的支持数据结构。当然，有可能提供一个更彻底的实现，比如使用 [Hibernate 的实体管理器](/web/20220626115805/https://www.baeldung.com/hibernate-entitymanager)。

现在，让我们定义 Maven 模块的 POM:

```java
<parent>
    // parent coordinates
</parent>

<groupId>com.baeldung.userdaomodule</groupId>
<artifactId>userdaomodule</artifactId>
<version>1.0</version>
<packaging>jar</packaging>
<name>userdaomodule</name>

<dependencies>
    <dependency>
        <groupId>com.baeldung.entitymodule</groupId>
        <artifactId>entitymodule</artifactId>
        <version>1.0</version>
    </dependency>
    <dependency>
        <groupId>com.baeldung.daomodule</groupId>
        <artifactId>daomodule</artifactId>
        <version>1.0</version>
    </dependency>
</dependencies>
```

在这种情况下，事情略有不同，因为`userdaomodule`模块需要 e `ntitymodule`和`daomodule`模块。这就是为什么我们将它们作为依赖项添加到`pom.xml`文件中。

我们仍然需要将这个 Maven 模块封装到一个 Java 模块中。因此，让我们在`userdaomodule/src/main/java`目录下添加以下模块描述符:

```java
module com.baeldung.userdaomodule {
    requires com.baeldung.entitymodule;
    requires com.baeldung.daomodule;
    provides com.baeldung.daomodule.Dao with com.baeldung.userdaomodule.UserDao;
    exports com.baeldung.userdaomodule;
} 
```

最后，我们需要将这个新模块添加到父 POM 中:

```java
<modules>
    <module>entitymodule</module>
    <module>daomodule</module>
    <module>userdaomodule</module>
</modules>
```

从高层次来看，很容易看出**、`pom.xml`文件和模块描述符扮演不同的角色**。即便如此，它们也能很好地互补。

假设我们需要更新 e `ntitymodule`和`daomodule` Maven 工件的版本。我们可以很容易地做到这一点，而不必改变模块描述符中的依赖关系。Maven 将负责为我们包含正确的工件。

类似地，我们可以通过修改模块描述符中的`“provides..with”`指令来改变模块提供的服务实现。

当我们一起使用 Maven 和 Java 模块时，我们会受益匪浅。前者带来了自动化、集中化的依赖管理功能，而后者提供了模块化的内在优势。

### 4.4。`mainappmodule` Maven 模块

此外，我们需要定义包含项目主类的 Maven 模块。

正如我们之前所做的，让我们在根目录下创建`mainappmodule/src/main/java/mainapp`目录结构，并向其中添加下面的`Application`类:

```java
public class Application {

    public static void main(String[] args) {
        Map<Integer, User> users = new HashMap<>();
        users.put(1, new User("Julie"));
        users.put(2, new User("David"));
        Dao userDao = new UserDao(users);
        userDao.findAll().forEach(System.out::println);
    }   
}
```

`Application`类的`main()`方法相当简单。首先，它用几个`User`对象填充一个`HashMap`。接下来，它使用一个`UserDao`实例从`Map,` 获取它们，然后将它们显示到控制台。

此外，我们还需要定义模块的`pom.xml`文件:

```java
<parent>
    // parent coordinates
</parent>

<groupId>com.baeldung.mainappmodule</groupId>
<artifactId>mainappmodule</artifactId>
<version>1.0</version>
<packaging>jar</packaging>
<name>mainappmodule</name>

<dependencies>
    <dependency>
        <groupId>com.baeldung.entitymodule</groupId>
         <artifactId>entitymodule</artifactId>
         <version>1.0</version>
    </dependency>
    <dependency>
        <groupId>com.baeldung.daomodule</groupId>
        <artifactId>daomodule</artifactId>
        <version>1.0</version>
    </dependency>
    <dependency>
        <groupId>com.baeldung.userdaomodule</groupId>
        <artifactId>userdaomodule</artifactId>
        <version>1.0</version>
    </dependency>
</dependencies> 
```

模块的依赖关系是不言自明的。因此，我们只需要将模块放在 Java 模块中。因此，在`mainappmodule/src/main/java`目录结构下，让我们包含模块描述符:

```java
module com.baeldung.mainappmodule {
    requires com.baeldung.entitypmodule;
    requires com.baeldung.userdaopmodule;
    requires com.baeldung.daopmodule;
    uses com.baeldung.daopmodule.Dao;
} 
```

最后，让我们将这个模块添加到父 POM 中:

```java
<modules>
    <module>entitymodule</module>
    <module>daomodule</module>
    <module>userdaomodule</module>
    <module>mainappmodule</module>
</modules> 
```

所有的子 Maven 模块都已经就位，并且整齐地封装在 Java 模块中，下面是项目的结构:

```java
multimodulemavenproject (the root directory)
pom.xml
|-- entitymodule
    |-- src
        |-- main
            | -- java
            module-info.java
            |-- com
                |-- baeldung
                    |-- entity
                    User.class
    pom.xml
|-- daomodule
    |-- src
        |-- main
            | -- java
            module-info.java
            |-- com
                |-- baeldung
                    |-- dao
                    Dao.class
    pom.xml
|-- userdaomodule
    |-- src
        |-- main
            | -- java
            module-info.java
            |-- com
                |-- baeldung
                    |-- userdao
                    UserDao.class
    pom.xml
|-- mainappmodule
    |-- src
        |-- main
            | -- java
            module-info.java
            |-- com
                |-- baeldung
                    |-- mainapp
                    Application.class
    pom.xml 
```

## 5。运行应用程序

最后，让我们运行应用程序，无论是从我们的 IDE 还是从控制台。

正如我们所料，当应用程序启动时，我们应该看到几个`User`对象被打印到控制台:

```java
User{name=Julie}
User{name=David} 
```

## 6。结论

在本教程中，我们以一种实用的方式**学习了如何在使用 Java 模块**的基本多模块 Maven 项目的开发中让 Maven 和 JPMS 并肩工作。

像往常一样，本教程中显示的所有代码示例都可以从 GitHub 上的[处获得。](https://web.archive.org/web/20220626115805/https://github.com/eugenp/tutorials/tree/master/maven-modules/multimodulemavenproject)