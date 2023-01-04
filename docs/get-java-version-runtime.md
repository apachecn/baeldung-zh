# 在运行时获取 Java 版本

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/get-java-version-runtime>

## 1.概观

有时当用 Java 编程时，以编程方式找到我们正在使用的 Java 版本可能会有所帮助。在本教程中，我们将研究几种获得 Java 版本的方法。

## 2.Java 版本命名约定

直到 Java 9，Java 版本都没有遵循[语义版本](/web/20221128042423/https://www.baeldung.com/cs/semantic-versioning)。格式为`1.X.Y_Z`。`X`和`Y`分别表示主要版本和次要版本。`Z`用于表示更新版本，用下划线“_”分隔。例如，`1.8.0_181`。对于 Java 9 和更高版本，Java 版本遵循语义版本化。语义版本使用`X.Y.Z`格式。它指的是主要、次要和修补程序。比如`11.0.7`。

## 3.获取 Java 版本

### 3.1.使用`System.getProperty`

系统属性是 Java 运行时提供的键值对。`java.version`是提供 Java 版本的系统属性。让我们定义一个获取版本的方法:

```java
public void givenJava_whenUsingSystemProp_thenGetVersion() {
    int expectedVersion = 8;
    String[] versionElements = System.getProperty("java.version").split("\\.");
    int discard = Integer.parseInt(versionElements[0]);
    int version;
    if (discard == 1) {
        version = Integer.parseInt(versionElements[1]);
    } else {
        version = discard;
    }
    Assertions.assertThat(version).isEqualTo(expectedVersion);
}
```

为了支持两种 Java [版本格式](/web/20221128042423/https://www.baeldung.com/java-comparing-versions)，我们应该检查第一个数字直到点号。

### 3.2.使用 Apache Commons Lang 3

获得 Java 版本的第二种方法是通过 [Apache Commons Lang 3](/web/20221128042423/https://www.baeldung.com/java-commons-lang-3) 库。我们首先需要将 [`commons-lang3`](https://web.archive.org/web/20221128042423/https://search.maven.org/search?q=g:org.apache.commons%20AND%20a:commons-lang3&core=gav) Maven 依赖项添加到我们的`pom.xml`文件中:

```java
<dependency>
    <groupId>org.apache.commons</groupId>
    <artifactId>commons-lang3</artifactId>
    <version>3.12.0</version>
</dependency>
```

我们将使用 [`SystemUtils`](/web/20221128042423/https://www.baeldung.com/java-commons-lang-3#the-systemutils-class) 类来获取关于 Java 平台的信息。让我们为此定义一个方法:

```java
public void givenJava_whenUsingCommonsLang_thenGetVersion() {
    int expectedVersion = 8;
    String[] versionElements = SystemUtils.JAVA_SPECIFICATION_VERSION.split("\\.");
    int discard = Integer.parseInt(versionElements[0]);
    int version;
    if (discard == 1) {
        version = Integer.parseInt(versionElements[1]);
    } else {
        version = discard;
    }
    Assertions.assertThat(version).isEqualTo(expectedVersion);
}
```

我们使用的`JAVA_SPECIFICATION_VERSION`是为了反映来自`java.specification.version`系统属性的可用值。

### 3.3.使用`Runtime.version()`

在 Java 9 及以上版本中，我们可以使用`Runtime.version()`来获取版本信息。让我们为此定义一个方法:

```java
public void givenJava_whenUsingRuntime_thenGetVersion(){
    String expectedVersion = "15";
    Runtime.Version runtimeVersion = Runtime.version();
    String version = String.valueOf(runtimeVersion.version().get(0));
    Assertions.assertThat(version).isEqualTo(expectedVersion);
}
```

## 4.结论

在本文中，我们描述了几种获得 Java 版本的方法。像往常一样，本教程中使用的所有代码示例都可以在 GitHub 上获得。