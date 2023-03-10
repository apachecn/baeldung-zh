# 从父插件覆盖 Maven 插件配置

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/maven-plugin-override-parent>

## 1.概观

在一个 Maven [多模块](/web/20220701171807/https://www.baeldung.com/maven-multi-module)项目中，**有效的 POM 是一个模块及其父模块中定义的所有配置合并的结果。**

为了避免模块之间的冗余和重复，我们通常在共享的父模块中保存公共配置。然而，如果我们需要为一个子模块定制配置，而又不影响它的所有兄弟模块，这可能是一个挑战。

在本教程中，我们将学习如何**覆盖父插件配置**。

## 2.默认配置继承

插件配置允许我们跨项目重用一个公共的构建逻辑。如果父母有一个插件，孩子将自动拥有它，不需要额外的配置。这就像一份遗产。

为了实现这一点，Maven **在元素级别**合并 XML 文件。如果子元素定义了不同值的元素，它将替换父元素的值。让我们来看看实际情况。

### 2.1.项目结构

首先，让我们定义一个多模块 Maven 项目来进行实验。我们的项目将由一对父母和两个孩子组成:

```java
+ parent
     + child-a
     + child-b 
```

假设我们想要配置`maven-compiler-plugin `在模块之间使用不同的 Java 版本。让我们配置我们的项目，一般使用 Java 11，但是让`child-a` 使用 Java 8。

我们将从`parent`配置开始:

```java
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-compiler-plugin</artifactId>
    <configuration>
        <source>11</source>
        <target>11</target>
        <maxmem>512m</maxmem>
    </configuration>
</plugin>
```

这里我们指定了一个额外的属性`maxmem`，我们也想使用它。然而，我们希望`child-a`有自己的编译器设置。

所以，让我们配置`child-a`使用 Java 8:

```java
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-compiler-plugin</artifactId>
    <configuration>
        <source>1.8</source>
        <target>1.8</target>
    </configuration>
</plugin>
```

现在我们有了例子，让我们看看这对有效的 POM 有什么影响。

### 2.2.了解有效的 POM

有效的 POM 受各种因素的影响，比如继承、配置文件、外部设置等等。为了查看实际的 POM，让我们从`child-a`目录运行`mvn help:effective-pom` :

```java
mvn help:effective-pom

...
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-compiler-plugin</artifactId>
    <configuration>
        <source>1.8</source>
        <target>1.8</target>
        <maxmem>512m</maxmem>
    </configuration>
</plugin>
```

正如所料，`child-a`有自己的`source`和`target` 值的变化。然而，一个额外的惊喜是它还拥有来自其父节点的`maxmem`属性。

这意味着**如果定义了任何子属性，它将获胜，否则，将使用父属性。**

## 3.高级配置继承

当我们想要微调合并策略时，我们可以使用属性。这些属性放在我们想要控制的 XML 元素上。此外，它们将被继承，并且将仅影响第一级子代。

### 3.1.使用列表

在前面的例子中，我们看到了如果子元素有不同的值会发生什么。现在，我们将看到子元素列表不同的情况。作为一个例子，让我们看看如何用 [maven-resources-plugin](/web/20220701171807/https://www.baeldung.com/maven-resources-plugin) 包含多个资源目录。

作为基线，让我们配置`parent`以包含来自`parent-resources`目录的资源:

```java
<plugin>
    <artifactId>maven-resources-plugin</artifactId>
    <configuration>
        <resources>
            <resource>
                <directory>parent-resources</directory>
            </resource>
        </resources>
    </configuration>
</plugin>
```

此时，`child-a`将从其父插件继承这个插件配置。然而，假设我们想要为`child-a`定义一个替代资源目录:

```java
<plugin>
    <artifactId>maven-resources-plugin</artifactId>
    <configuration>
        <resources>
            <resource>
                <directory>child-a-resources</directory>
            </resource>
        </resources>
    </configuration>
</plugin>
```

现在，让我们回顾一下有效的 POM:

```java
mvn help:effective-pom
...
<configuration>
    <resources>
        <resource>
            <directory>child-a-resources</directory>
        </resource>
    </resources>
</configuration>
```

在这种情况下，**整个列表被子配置覆盖**。

### 3.2.附加父配置

也许我们想让一些孩子使用一个公共的资源目录，并定义额外的资源目录。为此，**我们可以通过在父配置中的 resources 元素上使用`combine.children=”append”`** 来附加父配置:

```java
<resources combine.children="append">
    <resource>
        <directory>parent-resources</directory>
    </resource>
</resources>
```

因此，有效的 POM 将包含这两者:

```java
mvn help:effective-pom
....
<resources combine.children="append">
    <resource>
        <directory>parent-resources</directory>
    </resource>
    <resource>
        <directory>child-a-resources</directory>
    </resource>
</resources>
```

`combine` 属性不会传播到任何嵌套元素。因此，如果`resources`部分是一个复杂的结构，嵌套的元素将使用默认策略合并。

### 3.3.覆盖子配置

在前面的例子中，由于父母的联合策略，孩子不能完全控制最终的 POM。**子元素可以通过在其`resources`元素上添加`combine.self=”override”`** 来否决父元素:

```java
<resources combine.self="override">
    <resource>
        <directory>child-a-resources</directory>
    </resource>
</resources>
```

在这种情况下，孩子重新获得控制权:

```java
mvn help:effective-pom
...
<resources combine.self="override">
    <resource>
        <directory>child-a-resources</directory>
    </resource>
</resources>
```

## 4.不可继承插件

前面的属性适合于微调，但是当子对象完全不同意其父对象时，它们就不合适了。

为了避免插件被继承，我们可以**在父级**添加属性`<inherited>false</inherited>` :

```java
<plugin>
    <inherited>false</inherited>
    <groupId>org.apache.maven.plugins</groupId>
    ...
</plugin>
```

这意味着该插件将只应用于父插件，而不会传播到其子插件。

## 5.结论

在本文中，我们看到了如何覆盖父插件配置。

首先，我们研究了默认行为。然后我们看到父母如何定义合并策略，孩子如何拒绝它。最后，我们看到了如何将一个插件标记为不可继承，以避免所有的孩子都必须覆盖它。

和往常一样，代码可以在 GitHub 的[上获得。](https://web.archive.org/web/20220701171807/https://github.com/eugenp/tutorials/tree/master/maven-modules/version-overriding-plugins)