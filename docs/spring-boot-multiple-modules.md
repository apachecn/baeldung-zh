# 与 Spring Boot 的多模块项目

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-boot-multiple-modules>

## 1.概观

在这个快速教程中，我们将向**展示如何用`Spring Boot`** 创建一个多模块项目。

首先，我们将构建一个不是应用程序本身的库 jar，然后我们将构建一个使用我们的库的应用程序。

关于`Spring Boot`的介绍，请参考[这篇文章](/web/20220627091457/https://www.baeldung.com/spring-boot-start)。

## 2.设置

为了设置我们的多模块项目，让我们使用`pom packaging `创建一个简单的模块，在我们的 Maven 配置中聚合我们的库和应用程序模块:

```java
<groupId>com.baeldung</groupId>
<artifactId>parent-multi-module</artifactId>
<packaging>pom</packaging>
```

我们将在我们的项目中创建两个目录，这两个目录将应用程序模块与库`jar`模块分开。

让我们在`pom.xml`中声明我们的模块:

```java
<modules>
    <module>library</module>
    <module>application</module>
</modules>
```

## 3.图书馆罐子

对于我们的`library`模块，我们将使用`jar`封装:

```java
<groupId>com.baledung.example</groupId>
<artifactId>library</artifactId>
<packaging>jar</packaging>
```

由于我们希望**利用`Spring Boot`依赖管理**，我们将使用`spring-boot-starter-parent `作为父项目，注意**将`<relativePath/> `设置为空值**，这样 Maven 将从存储库中解析父项目`pom.xml`:

```java
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.2.6.RELEASE</version>
    <relativePath/>
</parent>
```

注意**如果我们有自己的父项目，我们可以在`pom.xml`的`<dependencyManagement/>`部分导入依赖管理作为物料清单(BOM)** :

```java
<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-dependencies</artifactId>
            <type>pom</type>
            <version>2.4.0</version>
            <scope>import</scope>
        </dependency>
    </dependencies>
<dependencyManagement>
```

最后，最初的依赖关系将非常简单:

```java
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
	    <artifactId>spring-boot-starter</artifactId>
    </dependency>
</dependencies>
```

在这个模块中， **`Spring Boot`插件是不必要的，因为它的主要功能是创建一个可执行的** `**über-jar**,`，这是我们不想要也不需要的库。

之后，我们准备好**开发一个将由库**提供的服务组件:

```java
@Service
public class EvenOddService {

    public String isEvenOrOdd(Integer number) {
        return number % 2 == 0 ? "Even" : "Odd";
    }
}
```

## 4.应用项目

像我们的`library`模块一样，我们的应用程序模块将使用`jar`打包:

```java
<groupId>com.baeldung.example</groupId>
<artifactId>application</artifactId>
<packaging>jar</packaging>
```

我们将像以前一样利用`Spring Boot`依赖性管理:

```java
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.2.6.RELEASE</version>
    <relativePath/>
</parent>
```

除了 Spring Boot 启动器依赖项之外，我们还将**包括我们在上一节**中创建的库 `jar` :

```java
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    <dependency>
        <groupId>com.baeldung.example</groupId>
        <artifactId>library</artifactId>
        <version>${project.version}</version>
    </dependency>
</dependencies> 
```

最后，我们将使用`Spring Boot`插件

```java
<build>
    <plugins>
        <plugin>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-maven-plugin</artifactId>
        </plugin>
    </plugins>
</build>
```

在这个地方使用上面提到的插件有几个方便的原因。

首先，它提供了一个内置的依赖关系解析器，设置版本号来匹配`Spring Boot` 依赖关系。

其次，它搜索 main 方法以标记为 runnable 类。

最后，也可能是最重要的，它收集了`classpath`上的所有`jars`，并构建了一个单独的、可运行的`über-jar`。

既然已经准备好编写我们的应用程序类并直奔主题，让我们**在主应用程序类**中实现一个控制器:

```java
@SpringBootApplication(scanBasePackages = "com.baeldung")
@RestController
public class EvenOddApplication {

    private EvenOddService evenOddService;

    // constructor

    @GetMapping("/validate/")
    public String isEvenOrOdd(
      @RequestParam("number") Integer number) {
        return evenOddService.isEvenOrOdd(number);
    }

    public static void main(String[] args) {
        SpringApplication.run(EvenOddApplication.class, args);
    }
}
```

## 5.结论

在本文中，我们探索了如何实现和配置一个多模块项目，并使用`Spring Boot`构建一个独立的库`jar`。

和往常一样，代码样本可以在 GitHub 上找到[。](https://web.archive.org/web/20220627091457/https://github.com/eugenp/tutorials/tree/master/spring-boot-modules/spring-boot-custom-starter)