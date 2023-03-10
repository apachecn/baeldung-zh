# Maven 中的 settings.xml 文件

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/maven-settings-xml>

## 1.概观

在使用 Maven 时，我们将大多数特定于项目的配置保存在`pom.xml`中。

Maven 提供了一个设置文件，`settings.xml,`,允许我们指定它将使用哪些本地和远程存储库。我们还可以用它来存储我们不希望在源代码中出现的设置，比如凭证。

在本教程中，我们将学习如何使用`settings.xml`。我们将看看代理、镜像和概要文件。我们还将讨论如何确定适用于我们项目的当前设置。

## 延伸阅读:

## 【Maven 属性的默认值

In this short article, we'll go through how to configure Maven properties default values, and how to use them.[Read more](/web/20220726092738/https://www.baeldung.com/maven-properties-defaults) →

## [Maven 中的附加源目录](/web/20220726092738/https://www.baeldung.com/maven-add-src-directories)

Learn how to configure additional source directories in Maven.[Read more](/web/20220726092738/https://www.baeldung.com/maven-add-src-directories) →

## [Maven 包装类型](/web/20220726092738/https://www.baeldung.com/maven-packaging-types)

In this article, we explore the different packaging types available in Maven.[Read more](/web/20220726092738/https://www.baeldung.com/maven-packaging-types) →

## 2.配置

`settings.xml`文件配置了一个 [Maven](/web/20220726092738/https://www.baeldung.com/maven) 安装。它类似于一个`pom.xml`文件，但是是全局定义的或者是针对每个用户的。

让我们探索一下可以在`settings.xml`文件中配置的元素。`settings.xml`文件的主`settings `元素可以包含九个可能的预定义子元素:

```java
<settings  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
      xsi:schemaLocation="http://maven.apache.org/SETTINGS/1.0.0 https://maven.apache.org/xsd/settings-1.0.0.xsd">
    <localRepository/>
    <interactiveMode/>
    <offline/>
    <pluginGroups/>
    <servers/>
    <mirrors/>
    <proxies/>
    <profiles/>
    <activeProfiles/>
</settings>
```

### 2.1.简单值

一些顶级配置元素包含简单值:

```java
<settings  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/SETTINGS/1.0.0 https://maven.apache.org/xsd/settings-1.0.0.xsd">
    <localRepository>${user.home}/.m2/repository</localRepository>
    <interactiveMode>true</interactiveMode>
    <offline>false</offline>
</settings>
```

`localRepository`元素指向系统本地存储库的路径。 **[本地存储库](/web/20220726092738/https://www.baeldung.com/maven-local-repository)是我们项目的所有依赖项被缓存的地方**。默认情况下，使用用户的主目录。但是，我们可以对其进行更改，以允许所有登录的用户从一个公共的本地存储库进行构建。

`interactiveMode`标志定义了我们是否允许 Maven 与请求输入的用户进行交互。该标志默认为`true`。

`offline`标志确定构建系统是否可以在离线模式下运行。这默认为`false;` ,但是，在构建服务器无法连接到远程存储库的情况下，我们可以将它切换到`true`。

### 2.2.插件组

`pluginGroups`元素包含一个指定了`groupId`的子元素列表。一个`groupId` 是创建特定 Maven 工件的组织的唯一标识符:

```java
<settings  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/SETTINGS/1.0.0 https://maven.apache.org/xsd/settings-1.0.0.xsd">
    <pluginGroups>
        <pluginGroup>org.apache.tomcat.maven</pluginGroup>
    </pluginGroups>
</settings>
```

**当一个插件在命令行**没有提供 **的情况下使用时，Maven 搜索插件组列表。默认情况下，该列表包含组`org.apache.maven.plugins`和`org.codehaus.mojo`。**

上面定义的`settings.xml`文件允许我们执行截断的 Tomcat 插件命令:

```java
mvn tomcat7:help
mvn tomcat7:deploy
mvn tomcat7:run
```

### 2.3.委托书

我们可以为 Maven 的部分或全部 HTTP 请求配置一个代理。*代理*元素允许一系列子代理元素，但是**一次只能有一个代理处于活动状态**:

```java
<settings  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/SETTINGS/1.0.0 https://maven.apache.org/xsd/settings-1.0.0.xsd">
    <proxies>
        <proxy>
            <id>baeldung-proxy</id>
            <active>true</active>
            <protocol>http</protocol>
            <host>baeldung.proxy.com</host>
            <port>8080</port>
            <username>demo-user</username>
            <password>dummy-password</password>
            <nonProxyHosts>*.baeldung.com|*.apache.org</nonProxyHosts>
        </proxy>
    </proxies>
</settings>
```

我们通过`active`标志来定义当前活动的代理。然后使用`nonProxyHosts`元素，我们指定哪些主机没有被代理。使用的分隔符取决于特定的代理服务器。最常见的分隔符是竖线和逗号。

### 2.4.镜子

存储库可以在项目中声明 `pom.xml.`这意味着共享项目代码的开发人员可以获得开箱即用的正确存储库设置。

在我们想要**定义** **作为特定存储库**的替代镜像的情况下，我们可以使用镜像。这将覆盖`pom.xml`中的内容。

例如，我们可以通过镜像所有存储库请求来强制 Maven 使用单个存储库:

```java
<settings  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" 
  xsi:schemaLocation="http://maven.apache.org/SETTINGS/1.0.0 https://maven.apache.org/xsd/settings-1.0.0.xsd">
    <mirrors>
        <mirror>
            <id>internal-baeldung-repository</id>
            <name>Baeldung Internal Repo</name>
            <url>https://baeldung.com/repo/maven2/</url>
            <mirrorOf>*</mirrorOf>
        </mirror>
    </mirrors>
</settings>
```

我们可以为给定的存储库只定义一个镜像，Maven 将选择第一个匹配。正常情况下，**我们应该使用通过 CDN 分布在世界各地的官方知识库**。

### 2.5.服务器

在项目 `pom.xml`中定义存储库是一个很好的实践。然而，我们不应该用`pom.xml`将安全设置(比如凭证)放入我们的源代码库中。相反，我们**在`settings.xml`文件**中定义这个 **安全信息:**

```java
<settings  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/SETTINGS/1.0.0 https://maven.apache.org/xsd/settings-1.0.0.xsd">
    <servers>
        <server>
            <id>internal-baeldung-repository</id>
            <username>demo-user</username>
            <password>dummy-password</password>
            <privateKey>${user.home}/.ssh/bael_key</privateKey>
            <passphrase>dummy-passphrase</passphrase>
            <filePermissions>664</filePermissions>
            <directoryPermissions>775</directoryPermissions>
            <configuration></configuration>
        </server>
    </servers>
</settings>
```

我们应该注意到，`settings.xml` 中服务器的`ID`需要匹配`pom.xml`中提到的存储库的`ID`元素。XML 还允许我们使用占位符从环境变量中获取凭证。

## 3.轮廓

*概要文件*元素使我们能够创建多个 [*概要文件*](/web/20220726092738/https://www.baeldung.com/maven-profiles) 子元素，通过它们的`ID`子元素来区分。`settings.xml`中的`profile`元素是`pom.xml`中相同元素的删节版本。

它只能包含四个子元素:`activation`、`repositories`、`pluginRepositories,` 和`properties.` 这些元素将构建系统配置为一个整体，而不是任何特定的项目。

需要注意的是，`settings.xml`中的活动配置文件的值将会被**覆盖`pom.xml`** 或`profiles.xml `文件中的任何等效配置文件值。通过`ID`匹配轮廓。

### 3.1.激活

我们只能在给定的情况下使用配置文件来修改某些值。我们可以使用`activation `元素来指定这些情况。因此，当满足所有指定标准时，配置文件**激活**:

```java
<settings  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/SETTINGS/1.0.0 https://maven.apache.org/xsd/settings-1.0.0.xsd">
    <profiles>
        <profile>
            <id>baeldung-test</id>
            <activation>
                <activeByDefault>false</activeByDefault>
                <jdk>1.8</jdk>
                <os>
                    <name>Windows 10</name>
                    <family>Windows</family>
                    <arch>amd64</arch>
                    <version>10.0</version>
                </os>
                <property>
                    <name>mavenVersion</name>
                    <value>3.0.7</value>
                </property>
                <file>
                    <exists>${basedir}/activation-file.properties</exists>
                    <missing>${basedir}/deactivation-file.properties</missing>
                </file>
            </activation>
        </profile>
    </profiles>
</settings>
```

有四种可能的激活器，不需要全部指定:

*   `jdk:`根据指定的 JDK 版本激活(支持范围)
*   *os:* 根据操作系统属性激活
*   *属性:*如果 Maven 检测到特定的属性值，则激活配置文件
*   *文件:*如果给定的文件名存在或缺失，则激活配置文件

为了检查哪个概要文件将激活某个构建，我们可以使用 Maven help 插件:

```java
mvn help:active-profiles
```

输出将显示给定项目的当前活动配置文件:

```java
[INFO] --- maven-help-plugin:3.2.0:active-profiles (default-cli) @ core-java-streams-3 ---
[INFO]
Active Profiles for Project 'com.baeldung.core-java-modules:core-java-streams-3:jar:0.1.0-SNAPSHOT':
The following profiles are active:
 - baeldung-test (source: com.baeldung.core-java-modules:core-java-streams-3:0.1.0-SNAPSHOT) 
```

### 3.2.性能

Maven 属性可以被认为是某个值的命名占位符。使用`${property_name}`符号可以在`pom.xml`文件中访问这些值的**:**

```java
<settings  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/SETTINGS/1.0.0 https://maven.apache.org/xsd/settings-1.0.0.xsd">
    <profiles>
        <profile>
            <id>baeldung-test</id>
            <properties>
                <user.project.folder>${user.home}/baeldung-tutorials</user.project.folder>
            </properties>
        </profile>
    </profiles>
</settings>
```

在`pom.xml`文件中有四种不同类型的属性:

*   使用`env` 前缀的属性返回一个环境变量值，比如`${env.` PATH}。
*   使用*项目*前缀的属性返回在`pom.xml`的`project`元素中设置的属性值，例如`${project.version}.`
*   使用*设置*前缀的属性从`settings.xml`返回相应元素的值，例如`${settings.localRepository}.`
*   我们可以通过 Java 中的`System.getProperties` 方法直接引用所有可用的属性，比如`${java.home}.`
*   我们可以在没有前缀的`properties`元素中使用属性集，比如`${junit.version}.`

### 3.3.仓库

远程存储库包含 Maven 用来填充本地存储库的工件集合。特定的工件可能需要不同的远程存储库。Maven **搜索在活动概要文件**下启用的存储库:

```java
<settings  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/SETTINGS/1.0.0 https://maven.apache.org/xsd/settings-1.0.0.xsd">
    <profiles>
        <profile>
            <id>adobe-public</id>
            <repositories>
	        <repository>
	            <id>adobe-public-releases</id>
	            <name>Adobe Public Repository</name>
	            <url>https://repo.adobe.com/nexus/content/groups/public</url>
	            <releases>
	                <enabled>true</enabled>
	                <updatePolicy>never</updatePolicy>
	            </releases>
	            <snapshots>
	                <enabled>false</enabled>
	            </snapshots>
	        </repository>
	    </repositories>
        </profile>
    </profiles>
</settings>
```

我们可以使用`repository`元素来启用特定存储库中工件的发布或快照版本。

### 3.4.插件库

Maven 工件有两种标准类型，依赖性和插件。由于 Maven 插件是一种特殊类型的工件，我们可以**将插件库与其他插件库**分开:

```java
<settings  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/SETTINGS/1.0.0 https://maven.apache.org/xsd/settings-1.0.0.xsd">
    <profiles>
        <profile>
            <id>adobe-public</id>
            <pluginRepositories>
               <pluginRepository>
                  <id>adobe-public-releases</id>
                  <name>Adobe Public Repository</name>
                  <url>https://repo.adobe.com/nexus/content/groups/public</url>
                  <releases>
                      <enabled>true</enabled>
                      <updatePolicy>never</updatePolicy>
                  </releases>
                  <snapshots>
                      <enabled>false</enabled>
                  </snapshots>
	        </pluginRepository>
	    </pluginRepositories>
        </profile>
    </profiles>
</settings>
```

值得注意的是，`pluginRepositories `元素的结构与`repositories `元素非常相似。

### 3.5.活动配置文件

`activeProfiles`元素包含引用特定概要文件`ID`的子元素。 **Maven 自动激活此处引用的任何配置文件**:

```java
<settings  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/SETTINGS/1.0.0 https://maven.apache.org/xsd/settings-1.0.0.xsd">
    <activeProfiles>
        <activeProfile>baeldung-test</activeProfile>
        <activeProfile>adobe-public</activeProfile>
    </activeProfiles>
</settings>
```

在这个例子中，每次调用`mvn`都像我们在命令行中添加了`-P baeldung-test,adobe-public`一样运行。

## 4.设置级别

一个`settings.xml`文件通常可以在几个地方找到:

*   Mavens 主目录中的全局设置:`${maven.home}/conf/settings.xml`
*   用户家中的用户设置:`${user.home}/.m2/settings.xml`

如果两个文件都存在，它们的内容将被合并。**用户设置中的配置优先**。

### 4.1.确定文件位置

为了确定全局和用户设置的位置，我们可以使用 debug 标志运行 Maven，并在输出中搜索`“settings”`:

```java
$ mvn -X clean | grep "settings"  [DEBUG] Reading global settings from C:\Program Files (x86)\Apache\apache-maven-3.6.3\bin\..\conf\settings.xml
[DEBUG] Reading user settings from C:\Users\Baeldung\.m2\settings.xml
```

### 4.2.确定有效设置

我们可以使用 Maven help 插件来**找出组合的全局和用户设置**的内容:

```java
mvn help:effective-settings
```

这以 XML 格式描述了设置:

```java
<settings  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/SETTINGS/1.0.0 https://maven.apache.org/xsd/settings-1.0.0.xsd">
    <localRepository>C:\Users\Baeldung\.m2\repository</localRepository>
    <pluginGroups>
        <pluginGroup>org.apache.tomcat.maven</pluginGroup>
        <pluginGroup>org.apache.maven.plugins</pluginGroup>
        <pluginGroup>org.codehaus.mojo</pluginGroup>
    </pluginGroups>
</settings> 
```

### 4.3.覆盖默认位置

Maven 还允许我们通过命令行覆盖全局和用户设置的位置:

```java
$ mvn clean --settings c:\user\user-settings.xml --global-settings c:\user\global-settings.xml
```

我们还可以使用相同命令的更短的`–s`版本:

```java
$ mvn clean --s c:\user\user-settings.xml --gs c:\user\global-settings.xml
```

## 5.结论

在本文中，我们**探索了 Maven 的`settings.xml`文件**中可用的配置。

我们学习了如何配置代理、存储库和概要文件。接下来，我们看了全局设置文件和用户设置文件之间的区别，以及如何确定哪个文件在使用中。

最后，我们研究了如何确定所使用的有效设置，以及如何覆盖默认文件位置。