# 查找未使用的梯度相关性

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/gradle-finding-unused-dependencies>

## 1.概观

有时在开发过程中，我们可能最终添加了比我们使用的更多的依赖项。

在这个快速教程中，我们将看到如何使用 Gradle [Nebula Lint](https://web.archive.org/web/20221126233016/https://github.com/nebula-plugins/gradle-lint-plugin) 插件来识别和修复这样的问题。

## 2.设置和配置

在我们的示例中，我们使用多模块 Gradle 5 设置。

这个插件只适用于基于 Groovy 的构建文件。

让我们在根项目构建文件中配置它:

```java
plugins {
    id "nebula.lint" version "16.9.0"
}

description = "Gradle 5 root project"

allprojects {
    apply plugin :"java"
    apply plugin :"nebula.lint"
    gradleLint {
        rules=['unused-dependency']
    }
    group = "com.baeldung"
    version = "0.0.1"
    sourceCompatibility = "1.8"
    targetCompatibility = "1.8"

    repositories {
        jcenter()
    }
}
```

**我们暂时只能为多项目构建这样配置**。这意味着我们不能在每个模块中单独应用它。

接下来，让我们配置模块依赖关系:

```java
description = "Gradle Unused Dependencies example"

dependencies {
    implementation('com.google.guava:guava:29.0-jre')
    testImplementation('junit:junit:4.12')
}
```

现在让我们在模块源代码中添加一个简单的主类:

```java
public class UnusedDependencies {

    public static void main(String[] args) {
        System.out.println("Hello world");
    }
}
```

稍后我们将在此基础上构建，看看插件是如何工作的。

## 3.检测场景和报告

该插件搜索输出 jar 来检测是否使用了依赖关系。

然而，**根据[几个条件](https://web.archive.org/web/20221126233016/https://github.com/nebula-plugins/gradle-lint-plugin/wiki/Unused-Dependency-Rule)，可以给我们不同的结果**。

我们将在接下来的部分探索更有趣的案例。

### 3.1.未使用的依赖项

现在我们有了自己的设置，让我们看看基本的用例。我们对未使用的依赖感兴趣。

让我们**运行`lintGradle`** 任务:

```java
$ ./gradlew lintGradle

> Task :lintGradle FAILED
# failure output omitted

warning   unused-dependency                  this dependency is unused and can be removed
unused-dependencies/build.gradle:6
implementation('com.google.guava:guava:29.0-jre')

✖ 1 problem (0 errors, 1 warning)

To apply fixes automatically, run fixGradleLint, review, and commit the changes.
# some more failure output 
```

让我们看看发生了什么。在我们的`compileClasspath`配置中有一个未使用的依赖项(`guava`)。

如果我们**像插件建议的那样运行`fixGradleLint`任务**,**的依赖性会自动从我们的`build.gradle`** 中移除。

然而，让我们对我们的依赖关系使用一些虚拟逻辑:

```java
public static void main(String[] args) {
    System.out.println("Hello world");
    useGuava();
}

private static void useGuava() {
    List<String> list = ImmutableList.of("Baledung", "is", "cool");
    System.out.println(list.stream().collect(Collectors.joining(" ")));
}
```

如果我们重新运行它，就不会再出现错误:

```java
$ ./gradlew lintGradle

BUILD SUCCESSFUL in 559ms
3 actionable tasks: 1 executed, 2 up-to-date
```

### 3.2.使用传递依赖关系

现在让我们加入另一个依赖项:

```java
dependencies {
    implementation('com.google.guava:guava:29.0-jre')
    implementation('org.apache.httpcomponents:httpclient:4.5.12')
    testImplementation('junit:junit:4.12')
}
```

这一次，让我们使用可传递依赖关系中的一些东西:

```java
public static void main(String[] args) {
    System.out.println("Hello world");
    useGuava();
    useHttpCore();
}

// other methods

private static void useHttpCore() {
    SSLContextBuilder.create();
}
```

让我们看看会发生什么:

```java
$ ./gradlew lintGradle

> Task :lintGradle FAILED
# failure output omitted 

warning   unused-dependency                  one or more classes in org.apache.httpcomponents:httpcore:4.4.13 
are required by your code directly (no auto-fix available)
warning   unused-dependency                  this dependency is unused and can be removed 
unused-dependencies/build.gradle:8
implementation('org.apache.httpcomponents:httpclient:4.5.12')

✖ 2 problems (0 errors, 2 warnings)
```

我们得到两个错误。第一个错误大致是说我们应该直接引用`httpcore`。

我们示例中的`SSLContextBuilder` 实际上是它的一部分。

第二个错误说我们没有使用来自`httpclient.`的任何东西

如果我们使用一个可传递的依赖关系，插件告诉我们把它变成一个直接的依赖关系。

让我们看一下我们的依赖关系树:

```java
$ ./gradlew unused-dependencies:dependencies --configuration compileClasspath

> Task :unused-dependencies:dependencies

------------------------------------------------------------
Project :unused-dependencies - Gradle Unused Dependencies example
------------------------------------------------------------

compileClasspath - Compile classpath for source set 'main'.
+--- com.google.guava:guava:29.0-jre
|    +--- com.google.guava:failureaccess:1.0.1
|    +--- com.google.guava:listenablefuture:9999.0-empty-to-avoid-conflict-with-guava
|    +--- com.google.code.findbugs:jsr305:3.0.2
|    +--- org.checkerframework:checker-qual:2.11.1
|    +--- com.google.errorprone:error_prone_annotations:2.3.4
|    \--- com.google.j2objc:j2objc-annotations:1.3
\--- org.apache.httpcomponents:httpclient:4.5.12
     +--- org.apache.httpcomponents:httpcore:4.4.13
     +--- commons-logging:commons-logging:1.2
     \--- commons-codec:commons-codec:1.11
```

在这种情况下，我们可以看到`httpcore` 是由`httpclient`带入的。

### 3.3.在反射中使用依赖关系

当我们使用反射时呢？

让我们稍微增强一下我们的例子:

```java
public static void main(String[] args) {
    System.out.println("Hello world");
    useGuava();
    useHttpCore();
    useHttpClientWithReflection();
}

// other methods

private static void useHttpClientWithReflection() {
    try {
        Class<?> httpBuilder = Class.forName("org.apache.http.impl.client.HttpClientBuilder");
        Method create = httpBuilder.getMethod("create", null);
        create.invoke(httpBuilder, null);
    } catch (Exception e) {
        e.printStackTrace();
    }
}
```

现在让我们重新运行 Gradle 任务:

```java
$ ./gradlew lintGradle

> Task :lintGradle FAILED
# failure output omitted

warning   unused-dependency                  one or more classes in org.apache.httpcomponents:httpcore:4.4.13 
are required by your code directly (no auto-fix available)

warning   unused-dependency                  this dependency is unused and can be removed
unused-dependencies/build.gradle:9
implementation('org.apache.httpcomponents:httpclient:4.5.12')

✖ 2 problems (0 errors, 2 warnings) 
```

发生了什么事？我们使用了依赖关系`(httpclient)`中的`HttpClientBuilder`,但仍然出错。

如果我们使用一个带反射的库，插件不会检测它的用法。

结果，我们可以看到同样的两个错误。

一般来说，我们应该配置像`runtimeOnly`这样的依赖关系。

### 3.4.生成报告

对于大型项目，终端返回的错误数量变得难以处理。

让我们将插件配置为给我们一个报告:

```java
allprojects {
    apply plugin :"java"
    apply plugin :"nebula.lint"
    gradleLint {
        rules=['unused-dependency']
        reportFormat = 'text'
    }
    // other  details omitted
}
```

让我们**运行`generateGradleLintReport` 任务**并检查我们的构建输出:

```java
$ ./gradlew generateGradleLintReport
# task output omitted

$ cat unused-dependencies/build/reports/gradleLint/unused-dependencies.txt

CodeNarc Report - Jun 20, 2020, 3:25:28 PM

Summary: TotalFiles=1 FilesWithViolations=1 P1=0 P2=3 P3=0

File: /home/user/tutorials/gradle-5/unused-dependencies/build.gradle
    Violation: Rule=unused-dependency P=2 Line=null Msg=[one or more classes in org.apache.httpcomponents:httpcore:4.4.13 
                                                         are required by your code directly]
    Violation: Rule=unused-dependency P=2 Line=9 Msg=[this dependency is unused and can be removed] 
                                                 Src=[implementation('org.apache.httpcomponents:httpclient:4.5.12')]
    Violation: Rule=unused-dependency P=2 Line=17 Msg=[this dependency is unused and can be removed] 
                                                  Src=[testImplementation('junit:junit:4.12')]

[CodeNarc (http://www.codenarc.org) v0.25.2] 
```

现在，它检测到了对`testCompileClasspath` 配置未使用的依赖项。

不幸的是，这是插件的不一致行为。结果，我们现在得到三个错误。

## 4.结论

在本教程中，我们看到了如何找到 Gradle 构建中未使用的依赖项。

首先，我们解释了一般的设置。之后，我们研究了不同依赖项报告的错误及其用法。

最后，我们看到了如何生成基于文本的报告。

像往常一样，我们可以在 GitHub 上找到完整的代码样本[。](https://web.archive.org/web/20221126233016/https://github.com/eugenp/tutorials/tree/master/gradle-modules/gradle-5)