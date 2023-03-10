# Java 中的版本比较

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-comparing-versions>

## 1.概观

随着 DevOps 技术的进步，在一天之内多次构建和部署一个应用程序是很常见的。

因此，**每个构建都被分配了一个唯一的版本号，这样我们就可以区分不同的构建**。有时，需要以编程方式比较版本字符串。

在本文中，我们将探索几种通过各种库比较 Java 版本字符串的方法。最后，我们将编写一个自定义程序来处理通用的版本字符串比较。

## 2.使用`maven-artifact`

首先，我们来探索一下 [Maven](/web/20220628120607/https://www.baeldung.com/maven) 是如何处理版本比较的。

### 2.1.Maven 依赖性

首先，我们将最新的 [`maven-artifact`](https://web.archive.org/web/20220628120607/https://search.maven.org/search?q=g:org.apache.maven%20a:maven-artifact) Maven 依赖项添加到我们的`pom.xml`:

```java
<dependency>
    <groupId>org.apache.maven</groupId>
    <artifactId>maven-artifact</artifactId>
    <version>3.6.3</version>
</dependency>
```

### 2.2.`ComparableVersion`

让我们探索一下 [`ComparableVersion`](https://web.archive.org/web/20220628120607/https://maven.apache.org/ref/3.6.3/maven-artifact/apidocs/org/apache/maven/artifact/versioning/ComparableVersion.html) 类。它提供了一个**版本比较的通用实现，具有无限数量的版本组件**。

它包含一个`compareTo`方法，当一个版本大于或小于另一个版本时，比较结果将分别大于或小于 0:

```java
ComparableVersion version1_1 = new ComparableVersion("1.1");
ComparableVersion version1_2 = new ComparableVersion("1.2");
ComparableVersion version1_3 = new ComparableVersion("1.3");

assertTrue(version1_1.compareTo(version1_2) < 0);
assertTrue(version1_3.compareTo(version1_2) > 0);
```

这里可以确认 1.1 版本小于 1.2 版本，1.3 版本大于 1.2 版本。

但是，当比较相同的版本时，我们将得到 0 的结果:

```java
ComparableVersion version1_1_0 = new ComparableVersion("1.1.0");
assertEquals(0, version1_1.compareTo(version1_1_0));
```

### 2.3.版本分隔符和限定符

此外，`ComparableVersion`类尊重点(.)和连字符(-)作为分隔符，其中**点分隔主版本和次版本，连字符定义限定符**:

```java
ComparableVersion version1_1_alpha = new ComparableVersion("1.1-alpha");
assertTrue(version1_1.compareTo(version1_1_alpha) > 0);
```

这里可以确认 1.1 版本大于 1.1-alpha 版本。

`ComparableVersion`支持几个著名的限定词，如`alpha`、`beta`、`milestone`、`RC`和`snapshot`(按从低到高的顺序排列):

```java
ComparableVersion version1_1_beta = new ComparableVersion("1.1-beta");
ComparableVersion version1_1_milestone = new ComparableVersion("1.1-milestone");
ComparableVersion version1_1_rc = new ComparableVersion("1.1-rc");
ComparableVersion version1_1_snapshot = new ComparableVersion("1.1-snapshot");

assertTrue(version1_1_alpha.compareTo(version1_1_beta) < 0);
assertTrue(version1_1_beta.compareTo(version1_1_milestone) < 0);
assertTrue(version1_1_rc.compareTo(version1_1_snapshot) < 0);
assertTrue(version1_1_snapshot.compareTo(version1_1) < 0);
```

此外，它**允许我们定义未知的限定符，并遵循它们的顺序，在已经讨论过的已知限定符之后，使用不区分大小写的词汇顺序**:

```java
ComparableVersion version1_1_c = new ComparableVersion("1.1-c");
ComparableVersion version1_1_z = new ComparableVersion("1.1-z");
ComparableVersion version1_1_1 = new ComparableVersion("1.1.1");

assertTrue(version1_1_c.compareTo(version1_1_z) < 0);
assertTrue(version1_1_z.compareTo(version1_1_1) < 0);
```

## 3.使用`gradle-core`

和 Maven 一样， [Gradle](/web/20220628120607/https://www.baeldung.com/gradle) 也有处理版本比较的内置能力。

### 3.1.Maven 依赖性

首先，让我们添加来自 [Gradle Releases repo](https://web.archive.org/web/20220628120607/https://repo.gradle.org/gradle/libs-releases-local/) 的最新 [`gradle-core`](https://web.archive.org/web/20220628120607/https://mvnrepository.com/artifact/org.gradle/gradle-core/6.1.1) Maven 依赖:

```java
<dependency>
    <groupId>org.gradle</groupId>
    <artifactId>gradle-core</artifactId>
    <version>6.1.1</version>
</dependency>
```

### 3.2.`VersionNumber`

Gradle 提供的 [`VersionNumber`](https://web.archive.org/web/20220628120607/https://github.com/gradle/gradle/blob/master/subprojects/core/src/main/java/org/gradle/util/VersionNumber.java) 类比较两个版本，类似于 Maven 的`ComparableVersion`类:

```java
VersionNumber version1_1 = VersionNumber.parse("1.1");
VersionNumber version1_2 = VersionNumber.parse("1.2");
VersionNumber version1_3 = VersionNumber.parse("1.3");

assertTrue(version1_1.compareTo(version1_2) < 0);
assertTrue(version1_3.compareTo(version1_2) > 0);

VersionNumber version1_1_0 = VersionNumber.parse("1.1.0");
assertEquals(0, version1_1.compareTo(version1_1_0)); 
```

### 3.3.版本组件

与`ComparableVersion`类不同，`VersionNumber`类仅支持五个版本组件— `Major`、`Minor`、`Micro`、`Patch`和`Qualifier`:

```java
VersionNumber version1_1_1_1_alpha = VersionNumber.parse("1.1.1.1-alpha"); 
assertTrue(version1_1.compareTo(version1_1_1_1_alpha) < 0); 

VersionNumber version1_1_beta = VersionNumber.parse("1.1.0.0-beta"); 
assertTrue(version1_1_beta.compareTo(version1_1_1_1_alpha) < 0);
```

### 3.4.版本方案

另外，`VersionNumber`支持几个不同的版本方案，比如`Major.Minor.Micro-Qualifier`和`Major.Minor.Micro.Patch-Qualifier`:

```java
VersionNumber version1_1_1_snapshot = VersionNumber.parse("1.1.1-snapshot");
assertTrue(version1_1_1_1_alpha.compareTo(version1_1_1_snapshot) < 0);
```

## 4.使用`jackson-core`

### 4.1.Maven 依赖性

类似于其他依赖项，让我们将最新的 [`jackson-core`](https://web.archive.org/web/20220628120607/https://search.maven.org/search?q=g:com.fasterxml.jackson.core%20a:jackson-core) Maven 依赖项添加到我们的`pom.xml`:

```java
<dependency>
    <groupId>com.fasterxml.jackson.core</groupId>
    <artifactId>jackson-core</artifactId>
    <version>2.11.1</version>
</dependency>
```

### 4.2.`Version`

然后，我们可以检查 [Jackson](/web/20220628120607/https://www.baeldung.com/jackson) 的 **[`Version`](https://web.archive.org/web/20220628120607/https://javadoc.io/doc/com.fasterxml.jackson.core/jackson-core/latest/com/fasterxml/jackson/core/Version.html) 类，它可以保存组件的版本信息以及可选的`groupId`和`artifactId`值**。

因此，`Version`类的构造函数允许我们定义`groupId`和`artifactId,`，以及像`Major`、`Minor`和`Patch`这样的组件:

```java
public Version (int major, int minor, int patchLevel, String snapshotInfo, String groupId, String artifactId) {
    //...
}
```

因此，让我们使用`Version`类来比较几个版本:

```java
Version version1_1 = new Version(1, 1, 0, null, null, null);
Version version1_2 = new Version(1, 2, 0, null, null, null);
Version version1_3 = new Version(1, 3, 0, null, null, null);

assertTrue(version1_1.compareTo(version1_2) < 0);
assertTrue(version1_3.compareTo(version1_2) > 0);

Version version1_1_1 = new Version(1, 1, 1, null, null, null);
assertTrue(version1_1.compareTo(version1_1_1) < 0);
```

### 4.3.`snapshotInfo`组件

比较两个版本时不使用`snapshotInfo`组件:

```java
Version version1_1_snapshot = new Version(1, 1, 0, "snapshot", null, null); 
assertEquals(0, version1_1.compareTo(version1_1_snapshot));
```

此外，`Version`类提供了`isSnapshot`方法来检查版本是否包含快照组件:

```java
assertTrue(version1_1_snapshot.isSnapshot());
```

### 4.4.`groupId`和`artifactId`组件

同样，这个类比较了`groupId`和`artifactId` 版本组件`:`的词汇顺序

```java
Version version1_1_maven = new Version(1, 1, 0, null, "org.apache.maven", null);
Version version1_1_gradle = new Version(1, 1, 0, null, "org.gradle", null);
assertTrue(version1_1_maven.compareTo(version1_1_gradle) < 0);
```

## 5.使用`Semver4J`

`Semver4j`库允许我们遵循 Java 中的[语义版本](/web/20220628120607/https://www.baeldung.com/cs/semantic-versioning)规范的规则。

### 5.1.Maven 依赖性

首先，我们来添加最新的 [`semver4j`](https://web.archive.org/web/20220628120607/https://search.maven.org/search?q=g:com.vdurmont%20a:semver4j) 美文依赖:

```java
<dependency>
    <groupId>com.vdurmont</groupId>
    <artifactId>semver4j</artifactId>
    <version>3.1.0</version>
</dependency>
```

### 5.2.`Semver`

然后，我们可以用 [`Semver`](https://web.archive.org/web/20220628120607/https://github.com/vdurmont/semver4j/blob/master/src/main/java/com/vdurmont/semver4j/Semver.java) 类来定义一个版本:

```java
Semver version1_1 = new Semver("1.1.0");
Semver version1_2 = new Semver("1.2.0");
Semver version1_3 = new Semver("1.3.0");

assertTrue(version1_1.compareTo(version1_2) < 0);
assertTrue(version1_3.compareTo(version1_2) > 0); 
```

在内部，它将一个版本解析成类似于`Major`、`Minor`和`Patch`的组件。

### 5.3.版本比较

另外，`Semver`类带有各种内置方法，如用于版本比较的`isGreaterThan`、`isLowerThan`和`isEqualTo`:

```java
Semver version1_1_alpha = new Semver("1.1.0-alpha"); 
assertTrue(version1_1.isGreaterThan(version1_1_alpha)); 

Semver version1_1_beta = new Semver("1.1.0-beta"); 
assertTrue(version1_1_alpha.isLowerThan(version1_1_beta)); 

assertTrue(version1_1.isEqualTo("1.1.0"));
```

同样，它提供了返回两个版本之间主要差异的`diff`方法:

```java
assertEquals(VersionDiff.MAJOR, version1_1.diff("2.1.0"));
assertEquals(VersionDiff.MINOR, version1_1.diff("1.2.3"));
assertEquals(VersionDiff.PATCH, version1_1.diff("1.1.1"));
```

### 5.4.版本稳定性

此外，`Semver`类附带了`isStable`方法来检查版本的稳定性，这由后缀的存在与否来确定:

```java
assertTrue(version1_1.isStable());
assertFalse(version1_1_alpha.isStable());
```

## 6.自定义解决方案

我们已经看到了一些比较版本字符串的解决方案。如果它们不适用于特定的用例，我们可能不得不编写一个定制的解决方案。

这里有一个简单的例子，适用于一些基本的情况——如果我们需要更多的东西，它总是可以扩展的。

这里的想法是使用点分隔符标记版本字符串，然后从左边开始比较每个`String`标记的整数转换。如果令牌的整数值相同，则检查下一个令牌，继续此步骤，直到找到差异为止(或者直到到达任一字符串中的最后一个令牌为止):

```java
public static int compareVersions(String version1, String version2) {
    int comparisonResult = 0;

    String[] version1Splits = version1.split("\\.");
    String[] version2Splits = version2.split("\\.");
    int maxLengthOfVersionSplits = Math.max(version1Splits.length, version2Splits.length);

    for (int i = 0; i < maxLengthOfVersionSplits; i++){
        Integer v1 = i < version1Splits.length ? Integer.parseInt(version1Splits[i]) : 0;
        Integer v2 = i < version2Splits.length ? Integer.parseInt(version2Splits[i]) : 0;
        int compare = v1.compareTo(v2);
        if (compare != 0) {
            comparisonResult = compare;
            break;
        }
    }
    return comparisonResult;
}
```

让我们通过比较几个版本来验证我们的解决方案:

```java
assertTrue(VersionCompare.compareVersions("1.0.1", "1.1.2") < 0);
assertTrue(VersionCompare.compareVersions("1.0.1", "1.10") < 0);
assertTrue(VersionCompare.compareVersions("1.1.2", "1.0.1") > 0);
assertTrue(VersionCompare.compareVersions("1.1.2", "1.2.0") < 0);
assertEquals(0, VersionCompare.compareVersions("1.3.0", "1.3"));
```

这段代码有一个限制，它只能比较由点分隔的整数组成的版本号。

因此，为了比较字母数字版本字符串，我们可以使用正则表达式来分离字母并比较词汇顺序。

## 7.结论

在本文中，我们研究了在 Java 中比较版本字符串的各种方法。

首先，我们分别使用`maven-artifact`和`gradle-core`依赖项，检查了由 Maven 和 Gradle 等构建框架提供的内置解决方案。然后，我们探索了`jackson-core`和`semver4j`库的版本比较特性。

最后，我们编写了一个通用版本字符串比较的定制解决方案。

像往常一样，所有的代码实现都可以在 GitHub 上获得[。](https://web.archive.org/web/20220628120607/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-string-operations-3)