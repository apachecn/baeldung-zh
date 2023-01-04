# 超级、最简单和有效 POM 之间的区别

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/maven-super-simplest-effective-pom>

## 1.概观

在这个简短的教程中，我们将使用 [Maven](/web/20221128044028/https://www.baeldung.com/maven) 来概述超级、最简单和有效的 POM 之间的区别。

## 2.什么是 POM？

`POM`代表`Project Object Model,`，是 Maven 中项目配置的核心。它是一个名为`pom.xml`的配置 XML 文件，包含了构建项目所需的大部分信息。

**`POM`文件的作用是描述项目，管理依赖关系，并声明帮助 Maven 构建项目的配置细节。**

## 3.超级 POM

为了更容易理解 super POM，我们可以用 Java 中的`Object`类进行类比:默认情况下，Java 中的每个类都扩展了`Object`类。类似地，对于 POM，每个 POM 都扩展了超级 POM。

****超级 POM 文件定义了所有的默认配置。**因此，即使是最简单的 POM 文件也会继承超级 POM 文件中定义的所有配置。**

根据我们使用的 Maven 版本，超级 POM 看起来可能会略有不同。例如，如果我们在机器上安装了 Maven，我们可以在`${M2_HOME}/lib`、`maven-model-builder-<version>.jar`文件中可视化它。如果我们打开这个 JAR 文件，我们会发现它的名字是`org/apache/maven/model/pom-4.0.0.xml`。

在接下来的章节中，我们将介绍版本`3.6.3.`的超级 POM 配置元素

### 3.1.仓库

在 Maven 构建期间，Maven 使用在`repositories`部分下定义的存储库来下载所有依赖的工件。

让我们来看一个例子:

```java
<repositories>
    <repository>
        <id>central</id>
        <name>Central Repository</name>
        <url>https://repo.maven.apache.org/maven2</url>
        <layout>default</layout>
        <snapshots>
            <enabled>false</enabled>
        </snapshots>
    </repository>
</repositories> 
```

### 3.2.插件库

默认的插件存储库是中央 Maven 存储库。让我们看看它在 *pluginRepository* 部分是如何定义的:

```java
<pluginRepositories>
    <pluginRepository>
        <id>central</id>
        <name>Central Repository</name>
        <url>https://repo.maven.apache.org/maven2</url>
        <layout>default</layout>
        <snapshots>
            <enabled>false</enabled>
        </snapshots>
        <releases>
            <updatePolicy>never</updatePolicy>
        </releases>
    </pluginRepository>
</pluginRepositories>
```

正如我们在上面看到的，快照被禁用，`updatePolicy`被设置为“`never`”。因此，在这种配置下，如果有新版本发布，Maven 永远不会自动更新插件。

### 3.3.建设

`build`配置部分包括构建项目所需的所有信息。

让我们看一个默认`build`部分的例子:

```java
<build>
    <directory>${project.basedir}/target</directory>
    <outputDirectory>${project.build.directory}/classes</outputDirectory>
    <finalName>${project.artifactId}-${project.version}</finalName>
    <testOutputDirectory>${project.build.directory}/test-classes</testOutputDirectory>
    <sourceDirectory>${project.basedir}/src/main/java</sourceDirectory>
    <scriptSourceDirectory>${project.basedir}/src/main/scripts</scriptSourceDirectory>
    <testSourceDirectory>${project.basedir}/src/test/java</testSourceDirectory>
    <resources>
        <resource>
	    <directory>${project.basedir}/src/main/resources</directory>
	</resource>
    </resources>
    <testResources>
        <testResource>
	    <directory>${project.basedir}/src/test/resources</directory>
	</testResource>
    </testResources>
    <pluginManagement>
        <!-- NOTE: These plugins will be removed from future versions of the super POM -->
	<!-- They are kept for the moment as they are very unlikely to conflict 
		with lifecycle mappings (MNG-4453) -->
	<plugins>
	    <plugin>
		<artifactId>maven-antrun-plugin</artifactId>
		<version>1.3</version>
	    </plugin>
	    <plugin>
		<artifactId>maven-assembly-plugin</artifactId>
		<version>2.2-beta-5</version>
	    </plugin>
	    <plugin>
		<artifactId>maven-dependency-plugin</artifactId>
		<version>2.8</version>
	    </plugin>
	    <plugin>
		<artifactId>maven-release-plugin</artifactId>
	        <version>2.5.3</version>
	    </plugin>
	</plugins>
    </pluginManagement>
</build>
```

### 3.4.报告

对于`reporting`，超级 POM 只为输出目录提供一个默认值:

```java
<reporting>
    <outputDirectory>${project.build.directory}/site</outputDirectory>
</reporting>
```

### 3.5.轮廓

如果我们没有在应用程序级别定义`profiles`，将会执行默认的构建概要文件。

默认的*概要文件*部分如下所示:

```java
<profiles>
    <!-- NOTE: The release profile will be removed from future versions of the super POM -->
    <profile>
        <id>release-profile</id>
	<activation>
	    <property>
		<name>performRelease</name>
		<value>true</value>
	    </property>
        </activation>
	<build>
	    <plugins>
		<plugin>
		    <inherited>true</inherited>
		    <artifactId>maven-source-plugin</artifactId>
		    <executions>
			    <execution>
			    <id>attach-sources</id>
			    <goals>
			        <goal>jar-no-fork</goal>
			    </goals>
			</execution>
		    </executions>
		</plugin>
		<plugin>
		    <inherited>true</inherited>
		    <artifactId>maven-javadoc-plugin</artifactId>
		    <executions>
			<execution>
			    <id>attach-javadocs</id>
			    <goals>
			        <goal>jar</goal>
			     </goals>
			</execution>
		    </executions>
		</plugin>
		<plugin>
		    <inherited>true</inherited>
		    <artifactId>maven-deploy-plugin</artifactId>
		    <configuration>
			<updateReleaseInfo>true</updateReleaseInfo>
		    </configuration>
		</plugin>
	    </plugins>
        </build>
    </profile>
</profiles> 
```

## 4.最简单的 POM

**最简单的 POM 是您在 Maven 项目中声明的 POM。**为了声明一个 POM，您至少需要指定这四个元素:`modelVersion`、`groupId`、`artifactId`和`version`。**最简单的 POM 将继承超级 POM** 的所有配置。

让我们来看看 Maven 项目的最低要求:

```java
<project>
    <modelVersion>4.0.0</modelVersion>
    <groupId>com.baeldung</groupId>
    <artifactId>maven-pom-types</artifactId>
    <version>1.0-SNAPSHOT</version>
</project>
```

Maven 中 POM 层次结构的一个主要优点是我们可以扩展和覆盖从顶层继承的配置。因此，要覆盖 POM 层次结构中给定元素或工件的配置，Maven 应该能够唯一地标识相应的工件。

## 5.有效 POM

**Effective POM 结合了超级 POM 文件中的所有默认设置和我们的应用程序 POM 中定义的配置。**当配置元素在应用程序`pom.xml`中没有被覆盖时，Maven 使用默认值。因此，如果我们从最简单的 POM 部分获取相同的样本 POM 文件，我们将看到有效的 POM 文件将是最简单和超级 POM 的合并。我们可以从命令行直观地看到它:

```java
mvn help:effective-pom
```

这也是查看 Maven 使用的默认值的最佳方式。

## 6.结论

在这个简短的教程中，我们讨论了 Maven 中`Project Object Models`的不同之处。

和往常一样，本教程中的例子可以在 GitHub 上找到。