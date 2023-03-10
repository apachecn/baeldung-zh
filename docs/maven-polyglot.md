# 腹部多形性肿大

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/maven-polyglot>

## 1。概述

[Maven Polyglot](https://web.archive.org/web/20220523232153/https://github.com/takari/polyglot-maven/) 是一组 Maven 核心扩展，允许用任何语言编写 POM 模型。这包括除 XML 之外的许多脚本和标记语言。

Maven polyglot 的主要目标是摆脱 XML，因为它不再是当今的主流语言。

在本教程中，我们将首先从理解 Maven 核心扩展概念和 Maven Polyglot 项目开始。

然后，我们将展示如何编写一个 Maven 核心扩展，它允许从 JSON 文件而不是著名的`pom.xml.`构建 POM 模型

## 2。Maven 内核扩展加载机制

**Maven 核心扩展是在 Maven 初始化**时和 Maven 项目构建开始之前加载的插件。这些插件允许在不改变内核的情况下改变 Maven 的行为。

例如，启动时加载的插件可以覆盖 Maven 的默认行为，并且可以从另一个文件而不是`pom.xml`中读取 POM 模型。

从技术上来说，**Maven 核心扩展是在`extensions.xml`文件中声明的 Maven 工件:**

```java
${projectDirectory}/.mvn/extensions.xml
```

下面是一个扩展的例子:

```java
<?xml version="1.0" encoding="UTF-8"?>
<extensions>
    <extension>
        <groupId>com.baeldung.maven.polyglot</groupId>
        <artifactId>maven-polyglot-json-extension</artifactId>
        <version>1.0-SNAPSHOT</version>
    </extension>
</extensions>
```

最后需要注意的是**这个机制需要 Maven 3.3.1 或者更高的**。

## 3。腹部息肉〔t1〕

Maven Polyglot 是一个核心扩展的集合。它们中的每一个都负责从脚本或标记语言中读取 POM 模型。

Maven Polyglot 为以下语言提供了扩展:

```java
+-----------+-------------------+--------------------------------------+
| Language  | Artifact Id       | Accepted POM files                   |
+-----------+-------------------+--------------------------------------+
| Atom      | polyglot-atom     | pom.atom                             |
| Clojure   | polyglot-clojure  | pom.clj                              |
| Groovy    | polyglot-groovy   | pom.groovy, pom.gy                   |
| Java      | polyglot-java     | pom.java                             |
| Kotlin    | polyglot-kotlin   | pom.kts                              |
| Ruby      | polyglot-ruby     | pom.rb, Mavenfile, Jarfile, Gemfile  |
| Scala     | polyglot-scala    | pom.scala                            |
| XML       | polyglot-xml      | pom.xml                            |
| YAML      | polyglot-yaml     | pom.yaml, pom.yml                    |
+-----------+-------------------+--------------------------------------+ 
```

在接下来的几节中，我们将首先看看如何使用上面支持的语言之一来构建一个 Maven 项目。

然后，我们将编写扩展来支持基于 JSON 的 POM。

## 4。使用 Maven 多语言扩展

基于不同于 XML 的语言构建 Maven 项目的一个选择是使用 Polyglot 项目提供的工件之一。

在我们的例子中，**我们将创建一个带有`pom.yaml`配置文件**的 Maven 项目。

第一步是创建 Maven 核心扩展文件:

```java
${projectDirectory}/.mvn/extensions.xml
```

然后我们将添加以下内容:

```java
<?xml version="1.0" encoding="UTF-8"?>
<extensions>
    <extension>
        <groupId>io.takari.polyglot</groupId>
        <artifactId>polyglot-yaml</artifactId>
        <version>0.3.1</version>
    </extension>
</extensions>
```

请根据上述语言随意调整`artifactId`到您选择的语言，并检查是否有任何[新版本](https://web.archive.org/web/20220523232153/https://search.maven.org/search?q=io.takari.polyglot)可用。

最后一步是在`YAML`文件中提供项目元数据:

```java
modelVersion: 4.0.0
groupId: com.baeldung.maven.polyglot
artifactId: maven-polyglot-yml-app
version: 1.0-SNAPSHOT
name: 'YAML Demo'

properties: {maven.compiler.source: 1.8, maven.compiler.target: 1.8}
```

现在我们可以像往常一样运行我们的构建了。例如，我们可以调用命令:

```java
mvn clean install
```

## 5。使用多语言翻译插件

另一个选择是使用`[polyglot-translate-plugin](https://web.archive.org/web/20220523232153/https://search.maven.org/search?q=a:polyglot-translate-plugin).`来获得一个基于支持语言的项目

这意味着我们可以用传统的`pom.xml.`从现有的 Maven 项目开始

然后，**我们可以通过使用翻译插件**将现有的`pom.xml`项目转换成想要的多语言版本:

```java
mvn io.takari.polyglot:polyglot-translate-plugin:translate -Dinput=pom.xml -Doutput=pom.yml
```

## 6。编写自定义扩展

由于 JSON 不是 Maven Polyglot 项目提供的语言之一，**我们将实现一个简单的扩展，允许从 JSON 文件**中读取项目元数据。

我们的扩展将提供 Maven `ModelProcessor` API 的定制实现，它将覆盖 Maven 的默认实现。

为了实现这一点，我们将改变如何定位 POM 文件以及如何读取元数据并将其转换为 Maven `Model` API 的行为。

### 6.1.Maven 依赖性

我们将从创建一个具有以下依赖项的 Maven 项目开始:

```java
<dependency>
    <groupId>org.apache.maven</groupId>
    <artifactId>maven-core</artifactId>
    <version>3.5.4</version>
    <scope>provided</scope>
</dependency>
<dependency>
    <groupId>com.fasterxml.jackson.core</groupId>
    <artifactId>jackson-databind</artifactId>
    <version>2.13.0</version>
</dependency>
```

这里**我们使用 [maven-core](https://web.archive.org/web/20220523232153/https://search.maven.org/search?q=g:org.apache.maven%20AND%20a:maven-core) 依赖，因为我们将实现一个核心扩展**。 [Jackson](https://web.archive.org/web/20220523232153/https://search.maven.org/search?q=a:jackson-databind%20AND%20g:com.fasterxml.jackson.core) 依赖项用于反序列化 JSON 文件。

由于 Maven 使用丛依赖注入容器，**我们需要我们的实现成为丛组件**。所以我们需要这个插件来生成丛元数据:

```java
<plugin>
    <groupId>org.codehaus.plexus</groupId>
    <artifactId>plexus-component-metadata</artifactId>
    <version>1.7.1</version>
    <executions>
        <execution>
            <goals>
                <goal>generate-metadata</goal>
            </goals>
        </execution>
    </executions>
</plugin>
```

### 6.2.自定义`ModelProcessor`实现

Maven 通过调用`ModelBuilder.build()`方法构建 POM 模型，该方法又委托给`ModelProcessor.read()`方法。

Maven 提供了一个`DefaultModelProcessor`实现，默认情况下，它从位于根目录或指定为参数命令的`pom.xml`文件中读取 POM 模型。

**因此，我们将提供一个定制的`ModelProcessor`实现，它将覆盖默认行为。这就是 POM 模型文件的位置以及如何读取它。**

所以让我们从创建一个`CustomModelProcessor`实现开始，并将其标记为一个丛组件:

```java
@Component(role = ModelProcessor.class)
public class CustomModelProcessor implements ModelProcessor {

    @Override
    public File locatePom(File projectDirectory) {
        return null;
    }

    @Override
    public Model read(
      InputStream input, 
      Map<String, ?> options) throws IOException, ModelParseException {
        return null;
    }
    //...
}
```

**`@Component`注释将使实现可以被 DI 容器(丛)注入。**因此，当 Maven 需要在`ModelBuilder,`中进行`ModelProcessor`注入时，丛容器将提供这个实现，而不是`DefaultModelProcessor.`

接下来，**我们将提供`locatePom()`方法**的实现。该方法返回文件，Maven 将在该文件中读取项目元数据。

因此，我们将返回一个`pom.json`文件(如果它存在的话),否则，我们通常会返回`pom.xml`:

```java
@Override
public File locatePom(File projectDirectory) {
    File pomFile = new File(projectDirectory, "pom.json");
    if (!pomFile.exists()) {
        pomFile = new File(projectDirectory, "pom.xml");
    }
    return pomFile;
}
```

`The next step is to read this file and transform it to the Maven Model. This is achieved by the read() method:`

```java
@Requirement
private ModelReader modelReader;

@Override
public Model read(InputStream input, Map<String, ?> options) 
  throws IOException, ModelParseException {

    FileModelSource source = getFileFromOptions(options);
    try (InputStream is = input) {
        //JSON FILE ==> Jackson
        if (isJsonFile(source)) {
            ObjectMapper objectMapper = new ObjectMapper();
            return objectMapper.readValue(input, Model.class);
        } else {
            // XML FILE ==> DefaultModelReader
            return modelReader.read(input, options);
        }
    }
    return model;
}
```

在这个例子中，我们检查文件是否是 JSON 文件，并使用 Jackson 将其反序列化到 Maven `Model.`中，否则，它是一个普通的 XML 文件，将被 Maven `DefaultModelReader.`读取

我们需要构建扩展，它将可以使用:

```java
mvn clean install
```

### 6.3.使用扩展

为了演示扩展的使用，我们将使用一个 Spring Boot 的 Web 项目。

首先，我们将创建一个 Maven 项目，并删除`pom.xml.`

然后，**我们将在`${projectDirectory}/.mvn/extensions.xml:`** 中添加我们已经实现的扩展

```java
<?xml version="1.0" encoding="UTF-8"?>
<extensions>
    <extension>
        <groupId>com.baeldung.maven.polyglot</groupId>
        <artifactId>maven-polyglot-json-extension</artifactId>
        <version>1.0-SNAPSHOT</version>
    </extension>
</extensions>
```

最后我们用以下内容创建`pom.json`:

```java
{
  "modelVersion": "4.0.0",
  "groupId": "com.baeldung.maven.polyglot",
  "artifactId": "maven-polyglot-json-app",
  "version": "1.0.1",
  "name": "Json Maven Polyglot",
  "parent": {
    "groupId": "org.springframework.boot",
    "artifactId": "spring-boot-starter-parent",
    "version": "2.0.5.RELEASE",
    "relativePath": null
  },
  "properties": {
    "project.build.sourceEncoding": "UTF-8",
    "project.reporting.outputEncoding": "UTF-8",
    "maven.compiler.source": "1.8",
    "maven.compiler.target": "1.8",
    "java.version": "1.8"
  },
  "dependencies": [
    {
      "groupId": "org.springframework.boot",
      "artifactId": "spring-boot-starter-web"
    }
  ],
  "build": {
    "plugins": [
      {
        "groupId": "org.springframework.boot",
        "artifactId": "spring-boot-maven-plugin"
      }
    ]
  }
}
```

我们现在可以使用命令运行项目:

```java
mvn spring-boot:run
```

## 7。结论

在本文中，我们展示了如何通过 Maven Polyglot 项目来改变默认的 Maven 行为。为了实现这个目标，我们使用了新的 Maven 3.3.1 特性来简化核心组件的加载。

代码和所有的样本都可以在 Github 上找到。