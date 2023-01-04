# 用 Maven 生成 WSDL 存根

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/maven-wsdl-stubs>

## 1.介绍

在本教程中，我们将展示如何配置 [JAX-WS maven 插件](https://web.archive.org/web/20221208143832/https://www.mojohaus.org/jaxws-maven-plugin/)来从 WSDL (web 服务描述语言)文件生成 Java 类。因此，我们将能够使用生成的类轻松地调用 web 服务。

## 2.配置我们的 Maven 插件

首先，让我们在我们的`pom.xml`文件的构建插件部分中包含我们的目标为`wsimport`的 JAX-WS Maven 插件:

```
<build>
    <plugins>
        <plugin>
            <groupId>org.codehaus.mojo</groupId>
            <artifactId>jaxws-maven-plugin</artifactId>
            <version>2.6</version>
            <executions>
                <execution>
                    <goals>
                        <goal>wsimport</goal>
                    </goals>
                </execution>
            </executions>
        </plugin>
    </plugins>
</build>
```

简而言之，`wsimport`目标**生成 JAX-WS 可移植工件，用于 JAX-WS 客户端和服务**。该工具读取一个 WSDL 文件，并生成 web 服务开发、部署和调用所需的所有构件。

### 2.1.WSDL 目录配置

在我们的 Maven 插件部分，**`wsdlDirectory`配置属性通知插件我们的 WSDL 文件位于哪里**。在这个例子中，我们将告诉插件获取我们项目的`src/main/resources`目录中的所有 WSDL 文件:

```
<plugin>
    ...
    <configuration>
        <wsdlDirectory>${project.basedir}/src/main/resources/</wsdlDirectory>
    </configuration>
</plugin>
```

### 2.2.WSDL 目录特定的文件配置

此外，我们可以使用`wsdlFiles` 配置属性来定义生成类时要考虑的 WSDL 文件列表:

```
<plugin>
    ...
    <configuration>
        <wsdlDirectory>${project.basedir}/src/main/resources/</wsdlDirectory>
        <wsdlFiles>
            <wsdlFile>file1.wsdl</wsdlFile>
            <wsdlFile>file2.wsdl</wsdlFile>
            ...
        </wsdlFiles>
    </configuration>
</plugin>
```

但是，**当没有设置`wsdlFiles`属性时，由`wsdlDirectory`属性指定的目录中的所有文件都将被视为**。

### 2.3.WSDL URL 配置

或者，我们可以配置插件的`wsdlUrl`配置属性:

```
<plugin>
    ...
    <configuration>
        <wsdlUrls>
            <wsdlUrl>http://localhost:8888/ws/country?wsdl</wsdlUrl>
        ...
        </wsdlUrls>
    </configuration>
</plugin>
```

要使用这个选项，**托管 WSDL 文件 URL 的服务器必须启动并运行，这样我们的插件才能读取它**。

### 2.4.配置生成的类目录

接下来，在`packageName` 属性中，我们可以设置生成的类的包名，并在`sourceDestDir`中，输出目录:

```
<plugin>
    ...
    <configuration>
        <packageName>com.baeldung.soap.ws.client</packageName>
        <sourceDestDir>
            ${project.build.directory}/generated-sources/
        </sourceDestDir>
    </configuration>   
</plugin>
```

因此，我们使用`wsdlDirectory` 选项的插件配置的最终版本是:

```
<plugin>
    <groupId>org.codehaus.mojo</groupId>
    <artifactId>jaxws-maven-plugin</artifactId>
    <version>2.6</version>
    <executions>
        <execution>
            <goals>
                <goal>wsimport</goal>
            </goals>
        </execution>
    </executions>
    <configuration>
        <wsdlDirectory>${project.basedir}/src/main/resources/</wsdlDirectory>
        <packageName>com.baeldung.soap.ws.client</packageName>
        <sourceDestDir>
            ${project.build.directory}/generated-sources/
        </sourceDestDir>
    </configuration>
</plugin>
```

## 3.运行 JAX-WS 插件

最后，配置好插件后，我们可以用 Maven 生成我们的类，并检查输出日志:

```
mvn clean install
```

```
[INFO] --- jaxws-maven-plugin:2.6:wsimport (default) @ jaxws ---
[INFO] Processing: file:/D:/projetos/baeldung/tutorials/maven-modules/maven-plugins/jaxws/src/main/resources/country.wsdl
[INFO] jaxws:wsimport args: [-keep, -s, 'D:\projetos\baeldung\tutorials\maven-modules\maven-plugins\jaxws\target\generated-sources', -d, 'D:\projetos\baeldung\tutorials\maven-modules\maven-plugins\jaxws\target\classes', -encoding, UTF-8, -Xnocompile, -p, com.baeldung.soap.ws.client, "file:/D:/projetos/baeldung/tutorials/maven-modules/maven-plugins/jaxws/src/main/resources/country.wsdl"]
parsing WSDL...
Generating code...
```

## 4.检查生成的类

运行我们的插件后，我们可以检查在`sourceDestDir` 属性中配置的文件夹`target/generated-sources`中的输出。

按照在`packageName`属性中的配置，可以在`com.baeldung.soap.ws.client`中找到生成的类:

```
com.baeldung.soap.ws.client.Country.java
com.baeldung.soap.ws.client.CountryService.java  
com.baeldung.soap.ws.client.CountryServiceImplService.java
com.baeldung.soap.ws.client.Currency.java
com.baeldung.soap.ws.client.ObjectFactory.java
```

## 5.结论

在本文中，我们看到了如何使用 JAX-WS 插件从 WSDL 文件生成 Java 类。因此，我们现在能够创建一个 web 服务客户端，并使用生成的类来调用我们的服务。

GitHub 上的[提供了我们应用程序的源代码。](https://web.archive.org/web/20221208143832/https://github.com/eugenp/tutorials/tree/master/maven-modules/maven-plugins/jaxws)