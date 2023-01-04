# 使用 Maven 删除重复的依赖项

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/maven-duplicate-dependencies>

## 1.概观

在本教程中，我们将学习如何使用 Maven 命令检测`pom.xml`中的重复依赖项。我们还将看到如果使用 Maven Enforcer 插件存在重复的依赖项，如何使构建失败。

## 2.为什么要检测重复的依赖关系？

在`pom.xml`中拥有重复依赖项的风险是，目标库的最新版本可能不会应用到我们项目的构建路径中。例如，让我们考虑下面的`pom.xml`:

```java
<project>
  [...]
  <dependencies>
    <dependency>
      <groupId>org.apache.commons</groupId>
      <artifactId>commons-lang3</artifactId>
      <version>3.12.0</version>
    </dependency>
    <dependency>
      <groupId>org.apache.commons</groupId>
      <artifactId>commons-lang3</artifactId>
      <version>3.11</version>
    </dependency>
  </dependencies>
   [...]
</project>
```

正如我们所见，同一个库`commons-lang3`有两个依赖项，尽管这两个依赖项中的版本不同。

接下来，让我们看看如何使用 Maven 命令来检测这些重复的依赖项。

## 3.依赖关系树命令

让我们从终端运行命令`[mvn dependency:tree](https://web.archive.org/web/20220630140426/https://maven.apache.org/plugins/maven-dependency-plugin/tree-mojo.html) `并查看输出。

```java
$ mvn dependency:tree
[INFO] Scanning for projects...
[WARNING]
[WARNING] Some problems were encountered while building the effective model for com.baeldung:maven-duplicate-dependencies:jar:0
.0.1-SNAPSHOT
[WARNING] 'dependencies.dependency.(groupId:artifactId:type:classifier)' must be unique: org.apache.commons:commons-lang3:jar -
> version 3.12.0 vs 3.11 @ line 14, column 15
[WARNING]
[WARNING] It is highly recommended to fix these problems because they threaten the stability of your build.
[WARNING]
[WARNING] For this reason, future Maven versions might no longer support building such malformed projects.
[WARNING]
[INFO]
[INFO] -------------< com.baeldung:maven-duplicate-dependencies >--------------
[INFO] Building maven-duplicate-dependencies 0.0.1-SNAPSHOT
[INFO] --------------------------------[ jar ]---------------------------------
[INFO]
[INFO] --- maven-dependency-plugin:2.8:tree (default-cli) @ maven-duplicate-dependencies ---
[WARNING] The artifact xml-apis:xml-apis:jar:2.0.2 has been relocated to xml-apis:xml-apis:jar:1.0.b2
[INFO] com.baeldung:maven-duplicate-dependencies:jar:0.0.1-SNAPSHOT
[INFO] \- org.apache.commons:commons-lang3:jar:3.11:compile
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time:  1.136 s
[INFO] Finished at: 2021-12-20T09:45:20+05:30
[INFO] ------------------------------------------------------------------------
```

这里，我们得到一个关于在`pom.xml. `中存在重复依赖项的警告。我们还注意到,`commons-lang3.jar is` 的 3.11 版本被添加到项目中，尽管有一个更高的版本,`3.12.0,`存在。**这是因为 Maven 选择了出现在`pom.xml`中的依赖项。**

## 4.依赖项`analyze-duplicate`命令

现在让我们运行命令 [`mvn dependency:analyze-duplicate`](https://web.archive.org/web/20220630140426/https://maven.apache.org/plugins/maven-dependency-plugin/analyze-duplicate-mojo.html) 并检查输出。

```java
$ mvn dependency:analyze-duplicate
[INFO] Scanning for projects...
[WARNING]
[WARNING] Some problems were encountered while building the effective model for com.baeldung:maven-duplicate-dependencies:jar:0
.0.1-SNAPSHOT
[WARNING] 'dependencies.dependency.(groupId:artifactId:type:classifier)' must be unique: org.apache.commons:commons-lang3:jar -
> version 3.12.0 vs 3.11 @ line 14, column 15
[WARNING]
[WARNING] It is highly recommended to fix these problems because they threaten the stability of your build.
[WARNING]
[WARNING] For this reason, future Maven versions might no longer support building such malformed projects.
[WARNING]
[INFO]
[INFO] -------------< com.baeldung:maven-duplicate-dependencies >--------------
[INFO] Building maven-duplicate-dependencies 0.0.1-SNAPSHOT
[INFO] --------------------------------[ jar ]---------------------------------
[INFO]
[INFO] --- maven-dependency-plugin:2.8:analyze-duplicate (default-cli) @ maven-duplicate-dependencies ---
[WARNING] The artifact xml-apis:xml-apis:jar:2.0.2 has been relocated to xml-apis:xml-apis:jar:1.0.b2
[INFO] List of duplicate dependencies defined in <dependencies/> in your pom.xml:
        o org.apache.commons:commons-lang3:jar
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time:  0.835 s
[INFO] Finished at: 2021-12-20T09:54:02+05:30
[INFO] ------------------------------------------------------------------------
```

这里，我们注意到`WARNING` 和`INFO`日志语句都提到了重复依赖的存在。

## 5.如果存在重复的依赖项，则构建失败

在上面的例子中，我们看到了如何检测重复的依赖关系，但是`BUILD` 仍然成功。这可能导致使用的`jar`版本不正确。

使用 [Maven Enforcer 插件](https://web.archive.org/web/20220630140426/https://maven.apache.org/enforcer/maven-enforcer-plugin/index.html)，我们可以确保如果存在重复的依赖项，构建会失败。

为此，我们需要将这个 Maven 插件添加到我们的`pom.xml `中，并添加规则 `banDuplicatePomDependencyVersions` :

```java
<project>
  [...]
  <build>
    <plugins>
      <plugin>
        <groupId>org.apache.maven.plugins</groupId>
        <artifactId>maven-enforcer-plugin</artifactId>
        <version>3.0.0</version>
        <executions>
          <execution>
            <id>no-duplicate-declared-dependencies</id>
            <goals>
              <goal>enforce</goal>
            </goals>
            <configuration>
              <rules>
                <banDuplicatePomDependencyVersions/>
              </rules>
            </configuration>
          </execution>
        </executions>
      </plugin>
    </plugins>
  </build>
  [...]
</project>
```

现在，规则绑定了我们的 Maven 构建:

```java
$ mvn verify
[INFO] Scanning for projects...
[WARNING]
[WARNING] Some problems were encountered while building the effective model for com.baeldung:maven-duplicate-dependencies:jar:0
.0.1-SNAPSHOT
[WARNING] 'dependencies.dependency.(groupId:artifactId:type:classifier)' must be unique: org.apache.commons:commons-lang3:jar -
> version 3.12.0 vs 3.11 @ line 14, column 14
[WARNING]
[INFO] -------------< com.baeldung:maven-duplicate-dependencies >--------------
[INFO] Building maven-duplicate-dependencies 0.0.1-SNAPSHOT
[INFO] --------------------------------[ jar ]---------------------------------
[INFO]
[INFO] --- maven-enforcer-plugin:3.0.0:enforce (no-duplicate-declared-dependencies) @ maven-duplicate-dependencies ---
[WARNING] Rule 0: org.apache.maven.plugins.enforcer.BanDuplicatePomDependencyVersions failed with message:
Found 1 duplicate dependency declaration in this project:
 - dependencies.dependency[org.apache.commons:commons-lang3:jar] ( 2 times )

[INFO] ------------------------------------------------------------------------
[INFO] BUILD FAILURE
[INFO] ------------------------------------------------------------------------
[INFO] Total time:  0.537 s
[INFO] Finished at: 2021-12-20T09:55:46+05:30
[INFO] ------------------------------------------------------------------------
[ERROR] Failed to execute goal org.apache.maven.plugins:maven-enforcer-plugin:3.0.0:enforce (no-duplicate-declared-dependencies) on project maven-duplicate-dependencie
s: Some Enforcer rules have failed. Look above for specific messages explaining why the rule failed.
```

## 6.删除重复的依赖关系

一旦我们确定了重复的依赖项，移除它们最简单的方法就是从`pom.xml` 中删除它们，只保留我们的项目使用的那些唯一的依赖项。

## 7.结论

在本文中，我们学习了如何使用`mvn dependency:tree`和`mvn dependency:analyze-duplicate`命令在 Maven 中检测重复的依赖关系。我们还看到了 Maven Enforcer 插件如何通过应用内置规则来使包含重复依赖项的构建失败。