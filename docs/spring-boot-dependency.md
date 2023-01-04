# 将 Spring Boot 应用程序用作依赖项

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-boot-dependency>

## 1.概观

在本教程中，我们将看到如何使用一个 Spring Boot 应用程序作为另一个项目的依赖。

## 2.Spring Boot 包装

Spring Boot Maven 和 Gradle 插件都将我们的应用程序打包成可执行 JARs 这样的文件**不能在另一个项目中使用，因为类文件被放在`BOOT-INF/classes`T2 中。这不是 bug，而是特性。**

为了与另一个项目共享类，最好的方法是**创建一个包含共享类**的单独的 jar，然后使它成为依赖它们的所有模块的依赖项。

但是如果这是不可能的，我们可以配置插件来生成一个单独的 jar，它可以作为一个依赖项使用。

### 2.1.Maven 配置

让我们用分类器配置插件:

```java
...
<build>
    ...
    <plugins>
        ...
        <plugin>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-maven-plugin</artifactId>
	    <configuration>
	        <classifier>exec</classifier>
            </configuration>
        </plugin>
    </plugins>
</build> 
```

不过，Spring Boot 1.x 的配置会有一点不同:

```java
...
<plugin>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-maven-plugin</artifactId>
    <executions>
        <execution>
            <goals>
                <goal>repackage</goal>
            </goals>
            <configuration>
                <classifier>exec</classifier>
            </configuration>
        </execution>
    </executions>
</plugin>
```

这将创建两个 jar，**一个带后缀`exec`作为可执行 jar，**和**另一个作为更典型的 jar，我们可以将它包含在其他项目**中。

## 3.用 Maven 汇编插件打包

我们也可以使用`[maven-assembly-plugin](https://web.archive.org/web/20221129213345/https://search.maven.org/search?q=a:maven-assembly-plugin)`来创建依赖 jar:

```java
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-assembly-plugin</artifactId>
    <configuration>
        <descriptorRefs>
            <descriptorRef>jar-with-dependencies</descriptorRef>
        </descriptorRefs>
    </configuration>
    <executions>
        <execution>
            <id>make-assembly</id>
            <phase>package</phase>
            <goals>
                <goal>single</goal>
            </goals>
        </execution>
    </executions>
</plugin>
```

如果我们将这个插件与`spring-boot-maven-plugin,` 中的`exec`分类器一起使用，它将生成三个 jar。前两个将与我们之前看到的相同。

第三个将具有我们在`<descriptorRef> `标签中指定的任何后缀，并将包含项目的所有可传递依赖项。如果我们将它包含在另一个项目中，我们就不需要单独包含 Spring 依赖项。

## 4.结论

在本文中，我们展示了几种打包 Spring Boot 应用程序的方法，以便在其他 Maven 项目中作为依赖项使用。

和往常一样，支持这篇文章的代码可以在 GitHub 上找到[。](https://web.archive.org/web/20221129213345/https://github.com/eugenp/tutorials/tree/master/spring-boot-modules/spring-boot-crud)