# 从 Jacoco 报告中排除

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/jacoco-report-exclude>

## 1.介绍

在本教程中，我们将学习如何从 [JaCoCo](/web/20220926195336/https://www.baeldung.com/jacoco) 测试覆盖报告中排除某些类和包。

通常，排除的候选对象可以是配置类、POJOs、dto 以及生成的字节代码。这些没有特定的业务逻辑，从报告中排除它们可能是有用的，以便提供测试覆盖的更好视图。

我们将在 Maven 和 Gradle 项目中探索各种排除方法。

## 2.例子

让我们从一个示例项目开始，在这个项目中，我们已经拥有了测试所覆盖的所有必需的代码。

接下来，我们将通过运行`mvn clean package`或`mvn jacoco:report`来生成覆盖率报告:

[![](img/3c0e527a820b252de3f8fb36e93964e4.png)](/web/20220926195336/https://www.baeldung.com/wp-content/uploads/2021/06/Screenshot-2021-05-30-at-21.07.46.png)

这个报告表明我们已经拥有了所需的覆盖率，遗漏的指令应该从 JaCoCo 报告度量中排除。

## 3.排除使用插件配置

类和包可以使用标准*和？插件配置中的通配符语法:

*   *匹配零个或多个字符
*   **匹配零个或多个目录
*   ？匹配单个字符

### 3.1.Maven 配置

让我们更新 Maven 插件，添加几个被排除的模式:

```java
<plugin> 
    <groupId>org.jacoco</groupId>
    <artifactId>jacoco-maven-plugin</artifactId>
    <configuration>
        <excludes>
            <exclude>com/baeldung/**/ExcludedPOJO.class</exclude>
            <exclude>com/baeldung/**/*DTO.*</exclude>
            <exclude>**/config/*</exclude>
        </excludes>
     </configuration>
     ...
</plugin>
```

这里，我们指定了以下排除:

*   `com.baeldung`包下任何子包中的`ExcludedPOJO `类
*   在`com.baeldung`包下的任何子包中，名称以`DTO`结尾的所有类
*   在根或子包中的任何地方声明的`config`包

### 3.2.梯度构型

我们也可以在毕业设计中应用同样的排除法。

首先，我们将更新`build.gradle`中的 JaCoCo 配置，并指定一个排除列表，使用与前面相同的模式:

```java
jacocoTestReport {
    dependsOn test // tests are required to run before generating the report

    afterEvaluate {
        classDirectories.setFrom(files(classDirectories.files.collect {
            fileTree(dir: it, exclude: [
                "com/baeldung/**/ExcludedPOJO.class",
                "com/baeldung/**/*DTO.*",
                "**/config/*"
            ])
        }))
    }
}
```

我们使用闭包来遍历类目录，并删除与指定模式列表匹配的文件。因此，使用`./gradlew jacocoTestReport`或 `./gradlew clean test`生成的报告将会排除所有指定的类和包，这是意料之中的。

值得注意的是，JaCoCo 插件被绑定到这里的`test`阶段，它在生成报告之前运行所有的测试。

## 4.使用自定义注释排除

从 JaCoCo 0.8.2 开始，我们可以通过使用具有以下属性的[自定义注释](/web/20220926195336/https://www.baeldung.com/java-custom-annotation) 来注释类和方法，从而**排除它们:**

*   注释的名称应该包括`Generated`。
*   注释的保留策略应该是`runtime `或`class.`

首先，我们将创建我们的注释:

```java
@Documented
@Retention(RUNTIME)
@Target({TYPE, METHOD})
public @interface Generated {
}
```

现在我们可以注释应该从覆盖率报告中排除的类或方法。

让我们首先在类级别使用这个注释:

```java
@Generated
public class Customer {
    // everything in this class will be excluded from jacoco report because of @Generated
}
```

类似地，我们可以将这个自定义注释应用于类中的特定方法:

```java
public class CustomerService {

    @Generated
    public String getCustomerId() {
        // method excluded form coverage report
    }

    public String getCustomerName() {
        // method included in test coverage report
    }
}
```

## 5.排除 Lombok 生成的代码

Project Lombok 是一个流行的库，用于大大减少 Java 项目中的样板文件和重复代码。

让我们看看如何通过向项目根目录中的文件添加一个属性来**排除所有由 Lombok 生成的字节码:**

```java
lombok.addLombokGeneratedAnnotation = true
```

基本上，这个属性将`[[email protected]](/web/20220926195336/https://www.baeldung.com/cdn-cgi/l/email-protection)`注释添加到所有用 Lombok 注释注释的类的相关方法、类和字段中，例如`Product`类。因此，JaCoCo 会忽略所有用这个注释标注的结构，它们不会显示在报告中。

最后，我们可以看到应用了上面显示的所有排除技术后的报告:

[![](img/c6dedb6da7e9c9f2d6ca7aab188d365e.png)](/web/20220926195336/https://www.baeldung.com/wp-content/uploads/2021/06/Screenshot-2021-05-30-at-21.25.03.png)

## 6.结论

在本文中，我们展示了从 JaCoCo 测试报告中指定排除项的各种方法。

最初，我们在插件配置中使用命名模式排除了几个文件和包。然后我们看到了如何使用`@Generated`来排除某些类，以及方法。最后，我们学习了如何使用配置文件从测试覆盖报告中排除所有 Lombok 生成的代码。

和往常一样， [Maven 源代码](https://web.archive.org/web/20220926195336/https://github.com/eugenp/tutorials/tree/master/testing-modules/testing-libraries-2)和 [Gradle 源代码](https://web.archive.org/web/20220926195336/https://github.com/eugenp/tutorials/tree/master/gradle-modules/gradle/gradle-jacoco)可以在 Github 上获得。