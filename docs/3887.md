# Java Web 入门指南

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-web-start>

## 1。概述

本文解释了什么是 Java Web Start (JWS ),如何在服务器端配置它，以及如何创建一个简单的应用程序。

注意:**从 Java 11 开始，JWS 已经从 Oracle JDK 中删除。作为替代，考虑使用 [OpenWebStart](https://web.archive.org/web/20221208143839/https://openwebstart.com/) 。**

## 2。简介

JWS 是一个运行时环境，它随 Java SE 一起提供给客户机的 web 浏览器，从 Java 版本 5 开始就已经存在了。

通过从 web 服务器下载 JNLP 文件(也称为 Java 网络启动协议)，这个环境允许我们远程运行它所引用的 JAR 包。

简单地说，这种机制在安装了常规 JRE 的客户机上加载并运行 Java 类。它还允许来自 Jakarta EE 的一些额外指令。然而，客户端的 JRE 会严格应用安全限制，通常会警告用户不可信的域、缺乏 HTTPS 甚至未签名的 jar。

从一个普通的网站，人们可以下载一个 JNLP 文件来执行 JWS 应用程序。下载后，可以直接从桌面快捷方式或 Java 缓存查看器运行。之后，它下载并执行 JAR 文件。

这种机制对于交付非基于 web(HTML free)的图形界面非常有帮助，例如安全文件传输应用程序、科学计算器、安全键盘、本地图像浏览器等等。

## 3。一个简单的 JNLP 应用程序

一个好的方法是编写一个应用程序，并将其打包成一个 WAR 文件，供常规 web 服务器使用。我们所需要的就是编写我们想要的应用程序(通常用 Swing)并打包到一个 JAR 文件中。然后，这个 JAR 必须与一个 JNLP 一起打包成一个 WAR 文件，该文件将正常引用、下载和执行其应用程序的`Main`类。

与打包在 WAR 文件中的常规 web 应用程序没有什么不同，只是我们需要一个 JNLP 文件来启用 JWS，这将在下面演示。

### `**3.1\. Java Application**`

让我们从编写一个简单的 Java 应用程序开始:

```
public class Hello {
    public static void main(String[] args) {
        JFrame f = new JFrame("main");
        f.setSize(200, 100);
        f.setLocationRelativeTo(null);
        f.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
        JLabel label = new JLabel("Hello World");
        f.add(label);
        f.setVisible(true);
    }
}
```

我们可以看到这是一个非常简单的 Swing 类。事实上，没有添加任何东西使其符合 JWS 标准。

### `**3.2\. Web Application**`

我们只需要将这个示例 Swing 类和下面的 JNLP 文件一起打包到一个 WAR 文件中:

```
<?xml version="1.0" encoding="UTF-8"?>
<jnlp spec="1.0+" 
  codebase="http://localhost:8080/jnlp-example">
    <information>
        <title>Hello</title>
        <vendor>Example</vendor>
    </information>
    <resources>
        <j2se version="1.2+"/>
        <jar href="hello.jar" main="true" />
    </resources>
    <application-desc/>
</jnlp>
```

我们把它命名为`hello.jndl`，放在我们战争的任何一个 web 文件夹下。JAR 和 WAR 都是可下载的，所以我们不需要担心把 JAR 放在一个`lib`文件夹中。

我们最终 JAR 的 URL 地址是硬编码在 JNLP 文件中的，这会导致一些分发问题。如果我们改变部署服务器，应用程序将不再工作。

让我们在本文后面用一个合适的 servlet 来解决这个问题。现在，让我们将下载的 JAR 文件作为`index.html`放在根文件夹中，并将其链接到一个锚元素:

```
<a href="hello.jnlp">Launch</a>
```

**让我们也在 JAR 清单中设置主类**。这可以通过在`pom.xml`文件中配置 JAR 插件来实现。类似地，我们将 JAR 文件移到了`WEB-INF/lib`之外，因为它只用于下载，即不用于类加载器:

```
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-jar-plugin</artifactId>
    ...
    <executions>
        <execution>
            <phase>compile</phase>
            <goals>
                <goal>jar</goal>
            </goals>
            <configuration>
                <archive>
                    <manifest>
                        <mainClass>
                            com.example.Hello
                        </mainClass>
                    </manifest>
                </archive>
                <outputDirectory>
                    ${project.basedir}/target/jws
                </outputDirectory>
            </configuration>
        </execution>
    </executions>
</plugin>
```

## 4。特殊配置

### T2`4.1\. Security Issues`

要运行一个应用程序，**我们需要对 JAR** 进行签名。创建有效的证书和使用 JAR Sign Maven 插件超出了本文的范围，但是出于开发目的，或者如果我们对用户的计算机有管理权限，我们可以绕过这个安全策略。

为此，我们需要将本地 URL(例如:`http://localhost:8080`)添加到将要执行应用程序的计算机上的 JRE 安装的安全异常列表中。可以通过打开安全选项卡上的 Java 控制面板(在 Windows 上，我们可以通过控制面板找到它)找到它。

## 5。`JnlpDownloadServlet`

### `**5.1\. Compression Algorithms**`

有一个特殊的 servlet 可以包含在我们的 WAR 中。它通过寻找 JAR 文件的最压缩的编译版本(如果有的话)来优化下载，并修复 JLNP 文件上的硬编码的`codebase`值。

因为我们的 JAR 可以下载，所以建议用压缩算法(比如 Pack200)打包它，并将常规 JAR 和任何 JAR.PACK.GZ 或 JAR.GZ 压缩版本放在同一个文件夹中，以便这个 servlet 可以为每种情况选择最佳选项。

不幸的是，对于这种压缩算法还没有稳定版本的 Maven 插件，但是我们可以使用 JRE 附带的 Pack200 可执行文件(通常安装在路径`{JAVA_SDK_HOME}/jre/bin/`)。

在不改变我们的 JNLP 的情况下，通过将 JAR 的`jar.gz`和`jar.pack.gz`版本放在同一个文件夹中，servlet 一旦收到来自远程 JNLP 的调用，就会选择更好的版本。这增强了用户体验并优化了网络流量。

### `**5.2\. Codebase Dynamic Substitution**`

servlet 还可以对`<jnlp spec=”1.0+” codebase=”http://localhost:8080/jnlp-example”>`标签中的硬编码 URL 执行动态替换。通过将 JNLP 更改为通配符`<jnlp spec=”1.0+” codebase=”$$context”>`，它提供了相同的最终呈现标记。

servlet 还使用通配符`$$codebase`、`$$hostname`、`$$name`和`$$site`，它们将分别解析`http://localhost:8080/jnlp-example/`、`localhost:8080`、`hello.jnlp`和`http://localhost:8080`。

### `**5.3\. Adding the Servlet to the Classpath**`

为了添加 servlet，让我们为 JAR 和 JNLP 模式配置一个普通的 servlet 映射到我们的`web.xml`:

```
<servlet>
    <servlet-name>JnlpDownloadServlet</servlet-name>
    <servlet-class>
        jnlp.sample.servlet.JnlpDownloadServlet
    </servlet-class>
</servlet>
<servlet-mapping>
    <servlet-name>JnlpDownloadServlet</servlet-name>
    <url-pattern>*.jar</url-pattern>
</servlet-mapping>
<servlet-mapping>
    <servlet-name>JnlpDownloadServlet</servlet-name>
    <url-pattern>*.jnlp</url-pattern>
</servlet-mapping>
```

servlet 本身包含在一组 jar(`jardiff.jar`和`jnlp-servlet.jar`)中，这些 jar 现在位于 Java SDK 下载页面上的 Demos & Samples 部分。

在 GitHub 示例中，这些文件包含在`java-core-samples-lib`文件夹中，并作为 web 资源由 Maven WAR 插件包含:

```
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-war-plugin</artifactId>
    ...
    <configuration>
        <webResources>
            <resource>
                <directory>
                    ${project.basedir}/java-core-samples-lib/
                </directory>
                <includes>
                    <include>**/*.jar</include>
                </includes>
                <targetPath>WEB-INF/lib</targetPath>
            </resource>
        </webResources>
    </configuration>
</plugin>
```

## 6。最终想法

Java Web Start 是一个可以在没有应用服务器的(内部网)环境中使用的工具。此外，对于需要操作本地用户文件的应用程序。

应用程序通过简单的下载协议发送给最终用户，除了一些安全问题(HTTPS、签名 JAR 等)，没有任何附加的依赖性或配置。).

在 [Git 示例](https://web.archive.org/web/20221208143839/https://github.com/eugenp/tutorials/tree/master/jws)中，本文描述的完整源代码可以下载。我们可以直接从 GitHub 下载到一个有 Tomcat 和 Apache Maven 的 OS 上。下载后，我们需要从源目录运行`mvn install`命令，并将生成的`jws.war`文件从`target`复制到 Tomcat 安装的`webapps`文件夹中。

之后，我们可以像往常一样启动 Tomcat。

在默认的 Apache Tomcat 安装中，这个例子可以在 URL `http://localhost:8080/jws/index.html`中找到。