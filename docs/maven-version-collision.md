# 如何解决 Maven 中工件的版本冲突

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/maven-version-collision>

## 1.概观

多模块 Maven 项目可以有复杂的依赖图。模块之间相互导入的内容越多，就会产生不寻常的结果。

在本教程中，我们将看到如何**解决 Maven** 中工件的版本冲突。

我们将从一个[多模块](/web/20220706153505/https://www.baeldung.com/maven-multi-module)项目开始，在这个项目中，我们特意使用了同一工件的不同版本。然后，我们将会看到如何通过排除或者依赖管理来防止得到一个工件的错误版本。

最后，我们将尝试使用 m `aven-enforcer-plugin`通过禁止使用传递依赖来使事情更容易控制。

## 2.工件的版本冲突

我们在项目中包含的每个依赖项都可能链接到其他工件。Maven 可以自动引入这些工件，也称为传递依赖。当多个依赖项链接到同一个工件，但是使用不同的版本时，就会发生版本冲突。

因此，我们的应用程序**在编译阶段和运行时**都可能有错误。

### 2.1.项目结构

让我们定义一个多模块项目结构来进行实验。我们的项目由一个`version-collision`父模块和三个子模块组成:

```java
version-collision
    project-a
    project-b
    project-collision 
```

`project-a`和`project-b` 的`pom.xml` 几乎相同。唯一的区别是他们所依赖的` com.google.guava`工件的版本。特别是`project-a`使用版本`22.0`:

```java
<dependencies>
    <dependency>
        <groupId>com.google.guava</groupId>
        <artifactId>guava</artifactId>
        <version>22.0</version>
    </dependency>
</dependencies>
```

但是，`project-b`用的是更新的版本，`29.0-jre`:

```java
<dependencies>
    <dependency>
        <groupId>com.google.guava</groupId>
        <artifactId>guava</artifactId>
        <version>29.0-jre</version>
    </dependency>
</dependencies>
```

第三个模块`project-collision`，依赖于另外两个模块:

```java
<dependencies>
    <dependency>
        <groupId>com.baeldung</groupId>
        <artifactId>project-a</artifactId>
        <version>0.0.1-SNAPSHOT</version>
    </dependency>
    <dependency>
        <groupId>com.baeldung</groupId>
        <artifactId>project-b</artifactId>
        <version>0.0.1-SNAPSHOT</version>
    </dependency>
</dependencies>
```

那么，`guava`的哪个版本会提供给`project-collision`？

### 2.2.使用特定依赖版本的功能

我们可以通过在使用来自`guava`的`Futures.immediateVoidFuture`方法的`project-collision`模块中创建一个简单的测试来找出使用了哪个依赖项:

```java
@Test
public void whenVersionCollisionDoesNotExist_thenShouldCompile() {
    assertThat(Futures.immediateVoidFuture(), notNullValue());
}
```

该方法仅在`29.0-jre`版本中可用。我们从其他模块中继承了这一点，但是只有当我们从`project-b.`中获得传递依赖时，我们才能编译我们的代码

### 2.3.版本冲突导致的编译错误

根据`project-collision`模块中依赖关系的顺序，在某些组合中，Maven 会返回一个编译错误:

```java
[ERROR] Failed to execute goal org.apache.maven.plugins:maven-compiler-plugin:3.8.1:testCompile (default-testCompile) on project project-collision: Compilation failure
[ERROR] /tutorials/maven-all/version-collision/project-collision/src/test/java/com/baeldung/version/collision/VersionCollisionUnitTest.java:[12,27] cannot find symbol
[ERROR]   symbol:   method immediateVoidFuture()
[ERROR]   location: class com.google.common.util.concurrent.Futures
```

那是`com.google.guava`神器版本冲突的结果。默认情况下，对于依赖关系树中同一级别的依赖关系，Maven 选择它找到的第一个库。在我们的例子中，两个`com.google.guava `依赖项处于相同的高度，并且选择了旧版本。

### 2.4.使用`maven-dependency-plugin`

`maven-dependency-plugin` 是一个非常有用的工具，可以显示所有依赖项及其版本:

```java
% mvn dependency:tree -Dverbose

[INFO] --- maven-dependency-plugin:2.8:tree (default-cli) @ project-collision ---
[INFO] com.baeldung:project-collision:jar:0.0.1-SNAPSHOT
[INFO] +- com.baeldung:project-a:jar:0.0.1-SNAPSHOT:compile
[INFO] |  \- com.google.guava:guava:jar:22.0:compile
[INFO] \- com.baeldung:project-b:jar:0.0.1-SNAPSHOT:compile
[INFO]    \- (com.google.guava:guava:jar:29.0-jre:compile - omitted for conflict with 22.0)
```

`-Dverbose`标志显示冲突的工件。事实上，我们在两个版本中都有一个`com.google.guava`依赖:22.0 和 29.0-jre。后者是我们想在`project-collision`模块中使用的。

## 3.从工件中排除可传递的依赖关系

解决版本冲突的一种方法是通过**从特定工件**中移除冲突的可传递依赖。在我们的例子中，我们不希望从`project-a `工件中过渡性地添加 *com.google.guava* 库。

因此，我们可以将其排除在`project-collision` pom 中:

```java
<dependencies>
    <dependency>
        <groupId>com.baeldung</groupId>
        <artifactId>project-a</artifactId>
        <version>0.0.1-SNAPSHOT</version>
        <exclusions>
            <exclusion>
                <groupId>com.google.guava</groupId>
                <artifactId>guava</artifactId>
            </exclusion>
        </exclusions>
    </dependency>
    <dependency>
        <groupId>com.baeldung</groupId>
        <artifactId>project-b</artifactId>
        <version>0.0.1-SNAPSHOT</version>
    </dependency>
</dependencies>
```

现在，当我们运行`dependency:tree`命令时，我们可以看到它不再存在:

```java
% mvn dependency:tree -Dverbose

[INFO] --- maven-dependency-plugin:2.8:tree (default-cli) @ project-collision ---
[INFO] com.baeldung:project-collision:jar:0.0.1-SNAPSHOT
[INFO] \- com.baeldung:project-b:jar:0.0.1-SNAPSHOT:compile
[INFO]    \- com.google.guava:guava:jar:29.0-jre:compile
```

结果，编译阶段没有错误地结束，我们可以使用版本`29.0-jre`中的类和方法。

## 4.使用`dependencyManagement`部分

Maven 的`dependencyManagement`部分是一个用于集中依赖信息的**机制。它最有用的特性之一是控制被用作可传递依赖项的工件版本。**

记住这一点，让我们在父`pom`中创建一个`dependencyManagement`配置:

```java
<dependencyManagement>
   <dependencies>
      <dependency>
         <groupId>com.google.guava</groupId>
         <artifactId>guava</artifactId>
         <version>29.0-jre</version>
      </dependency>
   </dependencies>
</dependencyManagement>
```

因此，Maven 将确保在所有子模块中使用`com.google.guava`工件的版本`29.0-jre`:

```java
% mvn dependency:tree -Dverbose

[INFO] --- maven-dependency-plugin:2.8:tree (default-cli) @ project-collision ---
[INFO] com.baeldung:project-collision:jar:0.0.1-SNAPSHOT
[INFO] +- com.baeldung:project-a:jar:0.0.1-SNAPSHOT:compile
[INFO] |  \- com.google.guava:guava:jar:29.0-jre:compile (version managed from 22.0)
[INFO] \- com.baeldung:project-b:jar:0.0.1-SNAPSHOT:compile
[INFO]    \- (com.google.guava:guava:jar:29.0-jre:compile - version managed from 22.0; omitted for duplicate)
```

## 5.防止意外的传递依赖

`maven-enforcer-plugin`提供了许多内置规则，使得**简化了多模块项目**的管理。其中之一**禁止使用来自传递依赖**的类和方法。

显式依赖声明消除了工件版本冲突的可能性。让我们将带有该规则的`maven-enforcer-plugin`添加到父 pom 中:

```java
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-enforcer-plugin</artifactId>
    <version>3.0.0-M3</version>
    <executions>
        <execution>
            <id>enforce-banned-dependencies</id>
            <goals>
                <goal>enforce</goal>
            </goals>
            <configuration>
                <rules>
                    <banTransitiveDependencies/>
                </rules>
            </configuration>
        </execution>
    </executions>
</plugin>
```

因此，如果我们想自己使用它，我们现在必须在我们的`project-collision`模块中显式声明`com.google.guava `工件。我们必须指定要使用的版本，或者在父`pom.xml`中设置`dependencyManagement`。这使得我们的项目更加防错，但要求我们在`pom.xml`文件中更加明确。

## 6.结论

在本文中，我们看到了如何解决 Maven 中工件的版本冲突。

首先，我们探索了一个多模块项目中版本冲突的例子。

然后，我们展示了如何在`pom.xml`中排除传递依赖。我们看了如何用父`pom.xml`中的`dependencyManagement`部分控制依赖版本。

最后，我们尝试用`maven-enforcer-plugin`来禁止使用传递性依赖，以便强制每个模块控制自己的模块。

和往常一样，本文中显示的代码可以在 GitHub 上的[处获得。](https://web.archive.org/web/20220706153505/https://github.com/eugenp/tutorials/tree/master/maven-modules/version-collision)