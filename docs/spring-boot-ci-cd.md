# 将 CI/CD 应用于 Spring Boot

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-boot-ci-cd>

## 1.概观

在本教程中，我们将了解持续集成/持续部署(CI/CD)过程，并实现其基本部分。

我们将创建一个简单的 Spring Boot 应用程序，然后将其推送到共享的 Git 存储库。之后，我们将使用构建集成服务来构建它，创建 Docker 映像，并将其推送到 Docker 存储库。

最后，我们将自动将我们的应用程序部署到 PaaS 服务(Heroku)。

## 2.版本控制

CI/CD 的关键部分是管理我们代码的版本控制系统。此外，我们需要一个存储库托管服务，我们的构建和部署步骤将与该服务相结合。

让我们选择 [Git](https://web.archive.org/web/20220703154105/https://git-scm.com/) 作为 VCS，选择 [GitHub](https://web.archive.org/web/20220703154105/https://github.com/) 作为我们的存储库提供商，因为它们是目前最流行的，并且可以免费使用。

首先，我们必须[在 GitHub 上创建一个账户](https://web.archive.org/web/20220703154105/https://github.com/join)。

此外，我们应该[创建一个 Git 库](https://web.archive.org/web/20220703154105/https://github.com/new)。姑且称之为`baeldung-ci-cd-process`。此外，让我们选择一个公共存储库，因为它将允许我们免费访问其他服务。最后，让我们用一个`README.md`来初始化我们的存储库。

既然我们的存储库已经创建，我们应该在本地克隆我们的项目。为此，让我们在本地计算机上执行以下命令:

```java
git clone https://github.com/$USERNAME/baeldung-ci-cd-process.git
```

这将在我们执行命令的目录中初始化我们的项目。目前，它应该只包含`README.md`文件。

## 3.创建应用程序

在这一节中，我们将创建一个简单的 Spring Boot 应用程序来参与这个过程。我们还将使用 Maven 作为我们的构建工具。

首先，让我们在克隆版本控制库的目录中初始化我们的项目。

例如，我们可以用 [Spring Initializer](https://web.archive.org/web/20220703154105/https://start.spring.io/) 来做这件事，增加`web `和`actuator `模块。

### 3.1.手动创建应用程序

或者，我们可以手动添加 [`spring-boot-starter-web`](https://web.archive.org/web/20220703154105/https://search.maven.org/search?q=g:org.springframework.boot%20AND%20a:spring-boot-starter-web) 和 `[spring-boot-starter-actuator](https://web.archive.org/web/20220703154105/https://search.maven.org/search?q=g:org.springframework.boot%20AND%20a:spring-boot-starter-actuator)`的依赖关系:

```java
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>

<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```

第一个是引入 REST 端点，第二个是健康检查端点。

此外，让我们添加允许我们运行应用程序的插件:

```java
<plugin>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-maven-plugin</artifactId>
</plugin>
```

最后，让我们添加一个 Spring Boot 主类:

```java
@SpringBootApplication
public class CiCdApplication {

    public static void main(String[] args) {
        SpringApplication.run(CiCdApplication.class, args);
    }
}
```

### 3.2.推

无论是使用 Spring Initializr 还是手动创建项目，我们现在都准备好提交我们的更改并将其推送到我们的存储库中。

让我们用下面的命令来实现这一点:

```java
git add .
git commit -m 'Initialize application'
git push
```

我们现在可以检查我们的变更是否存在于存储库中。

## 4.构建自动化

CI/CD 过程的另一部分是一个服务，它将构建和测试我们的推送代码。

我们将在这里使用 [Travis CI](https://web.archive.org/web/20220703154105/https://travis-ci.com/) ,但是任何构建服务也可以。

### 4.1.Maven 包装

让我们从给我们的应用程序添加一个 [Maven 包装器](https://web.archive.org/web/20220703154105/https://github.com/takari/maven-wrapper)开始。如果我们已经使用了 Spring Initializr，我们可以跳过这一部分，因为它是默认包含的。

在应用程序目录中，让我们执行以下操作:

```java
mvn -N io.takari:maven:0.7.7:wrapper
```

这将添加 Maven 包装文件，包括可以代替 Maven 使用的`mvnw`和`mvnw.cmd` 文件。

虽然 Travis CI 有自己的 Maven，但其他建筑服务可能没有。这个 Maven 包装器将帮助我们为这两种情况做好准备。此外，开发人员不必在他们的机器上安装 Maven。

### 4.2.建筑服务

之后，让我们使用 GitHub 帐户通过[登录](https://web.archive.org/web/20220703154105/https://travis-ci.com/signin)在 Travis CI 上创建一个帐户。展望未来，我们应该允许在 GitHub 中访问我们的项目。

接下来，我们应该创建一个`.travis.yml`文件，它将描述 Travis CI 中的构建过程。大多数构建服务都允许我们创建这样一个文件，它位于我们的存储库的根目录下。

在我们的例子中，让我们告诉 Travis 使用 Java 11 和 Maven 包装器来构建我们的应用程序:

```java
language: java
jdk:
  - openjdk11
script:
  - ./mvnw clean install
```

属性表明我们想要使用 Java。

`jdk` 属性表示从 DockerHub 下载哪个 Docker 图像，在本例中为`openjdk11` 。

属性说明了运行什么命令——我们希望使用我们的 Maven 包装器。

最后，我们应该将我们的更改推送到存储库。Travis 配置项应该会自动触发构建。

## 5.归档

在本节中，我们将使用我们的应用程序构建一个 Docker 映像，并作为 CD 过程的一部分将其托管在 [DockerHub](https://web.archive.org/web/20220703154105/https://hub.docker.com/) 上。它将允许我们轻松地在任何机器上运行它。

### 5.1.Docker 图像存储库

首先，我们应该为我们的图像创建一个 Docker 存储库。

让我们在 [DockerHub](https://web.archive.org/web/20220703154105/https://hub.docker.com/signup) 上创建一个帐户。同样，让我们[通过填写适当的字段来为我们的项目创建存储库](https://web.archive.org/web/20220703154105/https://docs.docker.com/docker-hub/repos/):

*   名称:`baeldung-ci-cd-process`
*   可见性:公共
*   构建设置:GitHub

### 5.2.Docker 图像

现在，我们准备创建一个 Docker 映像并将其推送到 DockerHub。

首先，让我们添加 [`jib-maven-plugin`](https://web.archive.org/web/20220703154105/https://search.maven.org/search?q=g:com.google.cloud.tools%20AND%20a:jib-maven-plugin)，它将创建我们的应用程序映像并将其推送到 Docker 存储库中(用正确的用户名替换`DockerHubUsername`):

```java
<profile>
    <id>deploy-docker</id>
    <properties>
        <maven.deploy.skip>true</maven.deploy.skip>
    </properties>
    <build>
        <plugins>
            <plugin>
                <groupId>com.google.cloud.tools</groupId>
                <artifactId>jib-maven-plugin</artifactId>
                <version>2.2.0</version>
                <configuration>
                    <to>
                        <image>${DockerHubUsername}/baeldung-ci-cd-process</image>
                        <tags>
                            <tag>${project.version}</tag>
                            <tag>latest</tag>
                        </tags>
                    </to>
                </configuration>
            </plugin>
        </plugins>
    </build>
</profile>
```

我们已经将它添加为 Maven 概要文件的一部分，以免用默认构建触发它。

此外，我们为图像指定了两个标签。要了解更多关于这个插件的信息，请访问我们关于 Jib 的[文章。](/web/20220703154105/https://www.baeldung.com/jib-dockerizing)

接下来，让我们调整我们的构建文件(`.travis.yml`):

```java
before_install:
  - echo "$DOCKER_PASSWORD" | docker login -u "$DOCKER_USERNAME" --password-stdin
  - docker pull openjdk:11-jre-slim-sid

script:
  - ./mvnw clean install
  - ./mvnw deploy jib:build -P deploy-docker
```

通过这些更改，构建服务将在构建应用程序之前登录 DockerHub。此外，它将使用我们的概要文件执行`deploy`阶段。在这个阶段，我们的应用程序将作为映像被推送到 Docker 存储库中。

最后，我们应该在构建服务中定义`DOCKER_PASSWORD` 和`DOCKER_USERNAME `变量。在 Travis CI 中，这些变量可以被定义为[构建设置](https://web.archive.org/web/20220703154105/https://docs.travis-ci.com/user/environment-variables/)的一部分。

现在，让我们把我们的变化推向 VCS。构建服务应该自动触发包含我们的更改的构建。

我们可以通过在本地运行以下命令来检查 Docker 映像是否已经被推送到存储库:

```java
docker run -p 8080:8080 -t $DOCKER_USERNAME/baeldung-ci-cd-process
```

现在，我们应该能够通过访问`[http://localhost:8080/actuator/health](https://web.archive.org/web/20220703154105/http://localhost:8080/actuator/health).`来访问我们的健康检查

## 6.代码分析

我们将在 CI/CD 过程中包括的下一件事是静态代码分析。这种过程的主要目标是确保最高的代码质量。例如，它可以检测到我们没有足够的测试用例，或者我们有一些安全问题。

让我们与 [CodeCov](https://web.archive.org/web/20220703154105/https://codecov.io/) 集成，它将通知我们关于我们的测试覆盖率。

首先，我们应该用 GitHub 概要文件登录 CodeCov 来建立集成。

其次，我们应该修改我们的代码。让我们从添加 [`jacoco`插件](https://web.archive.org/web/20220703154105/https://search.maven.org/search?q=g:org.jacoco%20AND%20a:jacoco-maven-plugin)开始:

```java
<plugin>
    <groupId>org.jacoco</groupId>
    <artifactId>jacoco-maven-plugin</artifactId>
    <version>0.8.5</version>
    <executions>
        <execution>
            <id>default-prepare-agent</id>
            <goals>
                <goal>prepare-agent</goal>
            </goals>
        </execution>
        <execution>
            <id>report</id>
            <phase>test</phase>
            <goals>
                <goal>report</goal>
            </goals>
        </execution>
    </executions>
</plugin>
```

该插件负责生成 CodeCov 将使用的测试报告。

接下来，我们应该调整构建服务文件(`.travis.yml`)中的脚本部分:

```java
script:
  - ./mvnw clean org.jacoco:jacoco-maven-plugin:prepare-agent install
  - ./mvnw deploy jib:build -P deploy-docker

after_success:
  - bash <(curl -s https://codecov.io/bash)
```

我们指示 jacoco 插件在`clean` `install`阶段触发。此外，我们还包含了`after_success `部分，它将在构建成功后向 CodeCov 发送报告。

接下来，我们应该在应用程序中添加一个测试类。例如，它可以是对主类的测试:

```java
@SpringBootTest
class CiCdApplicationIntegrationTest {

    @Test
    public void contextLoads() {

    }
}
```

最后，我们应该推动我们的变革。构建应该被触发，并且报告应该在与存储库相关的 CodeCov 概要文件中生成。

## 7.部署应用程序

作为流程的最后一部分，我们将部署我们的应用程序。有了 Docker 映像可供使用，我们可以将它部署在任何服务上。例如，我们可以将它部署在基于云的 PaaS 或 IaaS 上。

让我们将我们的应用程序部署到 Heroku，这是一个 PaaS，只需要很少的设置。

首先，我们应该[创建一个账户](https://web.archive.org/web/20220703154105/https://signup.heroku.com/)，然后[登录](https://web.archive.org/web/20220703154105/https://id.heroku.com/login)。

接下来，我们在 Heroku 中创建[应用空间](https://web.archive.org/web/20220703154105/https://dashboard.heroku.com/new-app)，命名为`baeldung-ci-cd-process` **。**应用程序的名称必须是唯一的，所以我们可能需要使用另一个名称。

我们将通过集成 Heroku 和 GitHub 来部署它，因为这是最简单的解决方案。然而，我们也可以编写使用 Docker 映像的管道。

接下来，我们应该在 pom 中包含 [`heroku`插件](https://web.archive.org/web/20220703154105/https://search.maven.org/search?q=g:com.heroku.sdk%20AND%20a:heroku-maven-plugin):

```java
<profile>
    <id>deploy-heroku</id>
    <properties>
        <maven.deploy.skip>true</maven.deploy.skip>
    </properties>
    <build>
        <plugins>
            <plugin>
                <groupId>com.heroku.sdk</groupId>
                <artifactId>heroku-maven-plugin</artifactId>
                <version>3.0.2</version>
                <configuration>
                    <appName>spring-boot-ci-cd</appName>
                    <processTypes>
                        <web>java $JAVA_OPTS -jar -Dserver.port=$PORT target/${project.build.finalName}.jar</web>
                    </processTypes>
                </configuration>
            </plugin>
        </plugins>
    </build>
</profile>
```

像 Docker 一样，我们已经将它添加为 Maven 概要文件的一部分。此外，我们在`web`部分包含了一个启动命令。

接下来，我们应该调整我们的构建服务文件(`.travis.yml`)，以便将应用程序也部署到 Heroku:

```java
script:
  - ./mvnw clean install
  - ./mvnw heroku:deploy jib:build -P deploy-heroku,deploy-docker
```

此外，让我们在构建服务的一个`HEROKU_API_KEY`变量中添加 Heroku API-KEY。

最后，让我们提交我们的更改。构建完成后，应该将应用程序部署到 Heroku。

我们可以通过访问 [`https://baeldung-ci-cd-process.herokuapp.com/actuator/health`](https://web.archive.org/web/20220703154105/https://baeldung-ci-cd-process.herokuapp.com/actuator/health) 来查看

## 8.结论

在本文中，我们了解了 CI/CD 流程的基本部分以及如何准备它们。

首先，我们在 GitHub 中准备了一个 Git 存储库，并将我们的应用程序推到那里。然后，我们使用 Travis CI 作为构建工具，从该存储库构建我们的应用程序。

之后，我们创建了一个 Docker 映像，并将其推送到 DockerHub。

接下来，我们添加了一个负责静态代码分析的服务。

最后，我们将应用程序部署到 PaaS 并访问它。

和往常一样，这些例子的代码可以在 GitHub 的[上获得。](https://web.archive.org/web/20220703154105/https://github.com/eugenp/tutorials/tree/master/spring-boot-modules/spring-boot-ci-cd)