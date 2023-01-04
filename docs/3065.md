# Maven Surefire 和 Failsafe 插件的区别

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/maven-surefire-vs-failsafe>

## 1.概观

在典型的测试驱动开发中，我们的目标是编写大量的低级单元测试，这些测试可以快速运行并独立设置。此外，还有一些依赖于外部系统的高级集成测试，例如，设置服务器或数据库。不出所料，这些通常既耗费资源又耗费时间。

因此，这些**测试大多需要一些集成前的设置和集成后的清理，以便优雅地终止。** **因此，区分这两种类型的测试并能够在构建过程中分别运行它们是可取的。**

在本教程中，我们将比较在典型的 [Apache Maven](/web/20220628051932/https://www.baeldung.com/maven-guide) 版本中运行各种类型的测试最常用的 Surefire 和 Failsafe 插件。

## 2.Surefire 插件

[Surefire 插件](/web/20220628051932/https://www.baeldung.com/maven-surefire-plugin)属于一组 Maven [核心](/web/20220628051932/https://www.baeldung.com/core-maven-plugins)插件，运行应用程序的单元测试。

项目 POM 默认包含这个插件，但是我们也可以显式地配置它:

```
<build>
    <pluginManagement>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-surefire-plugin</artifactId>
                <version>3.0.0-M5</version>
                ....
            </plugin>
         </plugins>
    </pluginManagement>
</build>
```

该插件绑定到默认[生命周期](/web/20220628051932/https://www.baeldung.com/maven#introduction-8)的`test`阶段。因此，让我们用以下命令执行它:

```
mvn clean test
```

这将运行我们项目中的所有单元测试。由于 **Surefire 插件与`test`阶段绑定，在任何测试失败的情况下，构建失败，并且在构建过程中没有进一步的阶段执行**。

或者，我们可以修改插件配置来运行集成测试和单元测试。然而，对于集成测试来说，这可能不是理想的行为，因为集成测试可能需要一些环境设置，以及一些测试执行后的清理工作。

Maven 为此提供了另一个插件。

## 3.故障保护插件

[故障保护插件](/web/20220628051932/https://www.baeldung.com/maven-failsafe-plugin)被设计用来运行项目中的集成测试。

### 3.1.配置

首先，让我们在项目 POM 中对此进行配置:

```
<plugin>
    <artifactId>maven-failsafe-plugin</artifactId>
    <version>3.0.0-M5</version>
    <executions>
        <execution>
            <goals>
                <goal>integration-test</goal>
                <goal>verify</goal>
            </goals>
            ....
        </execution>
    </executions>
</plugin>
```

这里，插件的目标绑定到构建周期的`integration-test`和`verify`阶段，以便执行集成测试。

现在，让我们从命令行执行`verify`阶段:

```
mvn clean verify
```

这将运行**所有的集成测试，但是如果在`integration-test`阶段有任何测试失败，插件不会立即导致构建失败**。

相反，Maven 仍然执行`post-integration-test`阶段。因此，作为`post-integration-test`阶段的一部分，我们仍然可以执行任何清理和环境拆除工作。构建过程的后续`verify`阶段报告任何测试失败。

### 3.2.例子

在我们的示例中，我们将配置一个 Jetty 服务器，在运行集成测试之前启动，在测试执行之后停止。

首先，让我们将 Jetty 插件添加到 POM 中:

```
<plugin>
    <groupId>org.eclipse.jetty</groupId>
    <artifactId>jetty-maven-plugin</artifactId>
    <version>9.4.11.v20180605</version>
    ....
    <executions>
        <execution>
            <id>start-jetty</id>
            <phase>pre-integration-test</phase>
            <goals>
                <goal>start</goal>
            </goals>
        </execution>
        <execution>
            <id>stop-jetty</id>
            <phase>post-integration-test</phase>
            <goals>
                <goal>stop</goal>
            </goals>
        </execution>
    </executions>
</plugin>
```

在这里，我们已经添加了分别在`pre-integration-test`和`post-integration-test`阶段启动和停止 Jetty 服务器的配置。

现在，让我们再次执行集成测试，并查看控制台输出:

```
....
[INFO] <<< jetty-maven-plugin:9.4.11.v20180605:start (start-jetty) 
  < validate @ maven-integration-test <<<
[INFO] --- jetty-maven-plugin:9.4.11.v20180605:start (start-jetty)
  @ maven-integration-test ---
[INFO] Started [[email protected]](/web/20220628051932/https://www.baeldung.com/cdn-cgi/l/email-protection){HTTP/1.1,[http/1.1]}{0.0.0.0:8999}
[INFO] Started @6794ms
[INFO] Started Jetty Server
[INFO]
[INFO] --- maven-failsafe-plugin:3.0.0-M5:integration-test (default)
  @ maven-integration-test ---
[INFO]
[INFO] -------------------------------------------------------
[INFO]  T E S T S
[INFO] -------------------------------------------------------
[INFO] Running com.baeldung.maven.it.FailsafeBuildPhaseIntegrationTest
[ERROR] Tests run: 1, Failures: 1, Errors: 0, Skipped: 0, Time elapsed: 0.024 s
  <<< FAILURE! - in com.baeldung.maven.it.FailsafeBuildPhaseIntegrationTest
[ERROR] com.baeldung.maven.it.FailsafeBuildPhaseIntegrationTest.whenTestExecutes_thenPreAndPostIntegrationBuildPhasesAreExecuted
  Time elapsed: 0.012 s  <<< FAILURE!
org.opentest4j.AssertionFailedError: expected: <true> but was: <false>
	at com.baeldung.maven.it.FailsafeBuildPhaseIntegrationTest
          .whenTestExecutes_thenPreAndPostIntegrationBuildPhasesAreExecuted(FailsafeBuildPhaseIntegrationTest.java:11)
[INFO]
[INFO] Results:
[INFO]
[ERROR] Failures:
[ERROR]   FailsafeBuildPhaseIntegrationTest.whenTestExecutes_thenPreAndPostIntegrationBuildPhasesAreExecuted:11
  expected: <true> but was: <false>
[INFO]
[ERROR] Tests run: 1, Failures: 1, Errors: 0, Skipped: 0
[INFO]
[INFO] --- jetty-maven-plugin:9.4.11.v20180605:stop (stop-jetty)
  @ maven-integration-test ---
[INFO]
[INFO] --- maven-failsafe-plugin:3.0.0-M5:verify (default)
  @ maven-integration-test ---
[INFO] Stopped [[email protected]](/web/20220628051932/https://www.baeldung.com/cdn-cgi/l/email-protection){HTTP/1.1,[http/1.1]}{0.0.0.0:8999}
[INFO] node0 Stopped scavenging
[INFO] ------------------------------------------------------------------------
[INFO] BUILD FAILURE
[INFO] ------------------------------------------------------------------------
....
```

在这里，按照我们的配置，Jetty 服务器在集成测试执行之前启动。为了进行演示，我们有一个失败的集成测试，但是这不会立即导致构建失败。`post-integration-test`阶段在测试执行之后执行，服务器在构建失败之前停止。

**相反，如果我们使用 Surefire 插件来运行这些集成测试，构建会在`integration-test`阶段停止，而不执行任何必要的清理**。

为不同类型的测试使用不同插件的另一个好处是不同配置之间的分离。这提高了项目构建的可维护性。

## 4.结论

在本文中，我们比较了 Surefire 和 Failsafe 插件用于分离和运行不同类型的测试。我们还看了一个例子，看到了故障安全插件如何为运行需要进一步设置和清理的测试提供额外的功能。

和往常一样，代码可以在 GitHub 的[上获得。](https://web.archive.org/web/20220628051932/https://github.com/eugenp/tutorials/tree/master/maven-modules/maven-integration-test)