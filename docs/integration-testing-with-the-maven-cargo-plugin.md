# 与 Maven Cargo 插件的集成测试

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/integration-testing-with-the-maven-cargo-plugin>

## 1.概观

项目生命周期中一个非常常见的需求是建立集成测试。在本教程中，我们将看到如何使用 Maven Cargo 插件来设置这个场景。

## 2.Maven 集成测试构建阶段

幸运的是，Maven 内置了对这种场景的支持，包括默认构建生命周期的以下阶段(来自 Maven [文档](https://web.archive.org/web/20220812060855/https://maven.apache.org/guides/introduction/introduction-to-the-lifecycle.html#Lifecycle_Reference "Lifecycle Reference in the official Maven documentation") ):

*   `**pre-integration-test**` : `Perform actions required before integration tests are executed. This may involve things such as setting up the required environment.`
*   `**integration-test**` : `Process and deploy the package if necessary into an environment where integration tests can be run.`
*   `**post-integration-test**` : `Perform actions required after integration tests have been executed. This may including cleaning up the environment.`

## 3.设置货物插件

让我们一步一步地检查所需的设置。

### 3.1.从 Surefire 插件中排除集成测试

首先， [maven-surefire-plugin](https://web.archive.org/web/20220812060855/https://maven.apache.org/plugins/maven-surefire-plugin/ "Maven surefire plugin") 被配置为从标准构建生命周期中排除**集成测试**:

```java
<plugin>
   <groupId>org.apache.maven.plugins</groupId>
   <artifactId>maven-surefire-plugin</artifactId>
   <version>2.22.2</version>
   <configuration>
      <excludes>
         <exclude>**/*IntegrationTest.java</exclude>
      </excludes>
   </configuration>
</plugin>
```

[排除](https://web.archive.org/web/20220812060855/https://maven.apache.org/plugins/maven-surefire-plugin/examples/inclusion-exclusion.html "Exclusions and Inclusions of tests")是通过 ant 风格的路径表达式完成的，所以所有的集成测试都必须遵循这种模式，并以`“IntegrationTest.java`结束。

### 3.2.配置货物插件

接下来，使用 [`cargo-maven3-plugin`](https://web.archive.org/web/20220812060855/https://codehaus-cargo.github.io/cargo/Maven+3+Plugin.html "The cargo-maven2-plugin") ，因为 [Cargo](https://web.archive.org/web/20220812060855/https://codehaus-cargo.github.io/ "Cargo homepage") 自带对嵌入式 web 服务器的顶级开箱即用支持。当然，如果服务器环境需要特定的配置，cargo 也知道如何从归档的包中构建服务器，并部署到外部服务器。

```java
<plugin>
   <groupId>org.codehaus.cargo</groupId>
   <artifactId>cargo-maven3-plugin</artifactId>
   <version>1.9.9</version>
   <configuration>
      <configuration>
         <properties>
            <cargo.servlet.port>8080</cargo.servlet.port>
         </properties>
      </configuration>
   </configuration>
</plugin>
```

定义了一个默认的嵌入式 Jetty 9 web 服务器，监听端口 8080。

在较新版本的 cargo (1.1.0 以上)中，**的默认值`wait`标志**已更改为`false,` 代表`cargo:start`。这个目标应该只用于运行集成测试，并且绑定到 Maven 生命周期；对于开发来说，应该执行`cargo:run`目标——它有`wait=true`。

为了让`package` maven 阶段生成一个**可部署的`war`** 文件，项目的打包必须是`<packaging>war</packaging>`。

### 3.3.添加新的 Maven 概要文件

接下来，一个新的`integration` **Maven 概要文件**被创建，以便当这个概要文件是活动的时候，只运行集成测试**，而不是作为标准构建生命周期的一部分。**

```java
<profiles>
   <profile>
      <id>integration</id>
      <build>

         <plugins>
            ...
         </plugins>

      </build>
   </profile>
</profiles>
```

这个概要文件将包含所有剩余的配置细节。

现在，Jetty 服务器被配置为在`pre-integration-test`阶段**启动**，在`post-integration-test`阶段**停止**。

```java
<plugin>
   <groupId>org.codehaus.cargo</groupId>
   <artifactId>cargo-maven3-plugin</artifactId>
   <executions>
      <execution>
         <id>start-server</id>
         <phase>pre-integration-test</phase>
         <goals>
            <goal>start</goal>
         </goals>
      </execution>
      <execution>
         <id>stop-server</id>
         <phase>post-integration-test</phase>
         <goals>
            <goal>stop</goal>
         </goals>
      </execution>
   </executions>
</plugin>
```

这确保了`cargo:start`目标和`cargo:stop`目标将在`integration-test`阶段之前和之后执行。注意，因为有两个单独的**执行**定义，所以 **`id`** 元素必须在两者中都存在(并且不同)，这样 Maven 才能接受配置。

### 3.4.在新的概要文件中配置集成测试

接下来，需要在`integration`概要文件中覆盖`maven-surefire-plugin`配置，这样，在默认生命周期中被排除的集成测试现在将被**包含在**中并运行:

```java
<plugins>
   <plugin>
      <groupId>org.apache.maven.plugins</groupId>
      <artifactId>maven-surefire-plugin</artifactId>
      <executions>
         <execution>
            <phase>integration-test</phase>
            <goals>
               <goal>test</goal>
            </goals>
            <configuration>
               <excludes>
                  <exclude>none</exclude>
               </excludes>
               <includes>
                  <include>**/*IntegrationTest.java</include>
               </includes>
            </configuration>
         </execution>
      </executions>
   </plugin>
</plugins>
```

有几样东西一文不值:

1.`maven-surefire-plugin`的 **`test`** 目标在`integration-test`阶段执行；此时，Jetty 已经启动，项目已经部署，所以集成测试运行起来应该没有问题。

2.集成测试现在**包括**在执行中。为了实现这一点，排除项也被覆盖——这是因为 Maven 处理覆盖概要文件中插件配置的方式。

基本配置并没有被完全覆盖，而是在概要文件中增加了新的配置元素。

因此，最初排除集成测试的原始`<excludes>`配置仍然存在于概要文件中，需要被覆盖，否则它将与`<includes>`配置冲突，测试仍然无法运行。

3.注意，因为只有一个`<execution>`元素，所以**不需要定义`id`** 。

**现在，整个流程可以运行:**

`mvn clean install -Pintegration`

## 4。结论

Maven 的逐步配置涵盖了作为项目生命周期的一部分设置集成过程的整个过程。

通常，这被设置为在持续集成环境中运行，最好是在每次提交之后。如果 CI 服务器已经有一个运行和使用端口的服务器，那么 cargo 配置将必须处理这种情况，我将在以后的文章中介绍这一点。

要获得该机制的完整运行配置，请查看 REST GitHub 项目。

另外，[查看这篇文章](https://web.archive.org/web/20220812060855/http://www.petrikainulainen.net/programming/maven/integration-testing-with-maven/ "How to run unit and integration tests separately"),了解构建项目和组织单元和集成测试的最佳实践。**