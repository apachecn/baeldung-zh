# 在 Tomcat 根目录下部署应用程序

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/tomcat-root-application>

## 1.概观

在这篇简短的文章中，我们将讨论在 Tomcat 的根目录下部署 web 应用程序。

## 2.Tomcat 部署基础和术语

首先，在本指南中可以找到将应用程序部署到 Tomcat 的基础知识:[如何将 WAR 文件部署到 Tomcat](/web/20220627074813/https://www.baeldung.com/tomcat-deploy-war) 。

简单来说，web 应用放在`$CATALINA_HOME\webapps`下，其中`$CATALINA_HOME`是 Tomcat 的安装目录。

**上下文路径是指相对于服务器地址的位置，服务器地址代表 web 应用程序的名称。**

默认情况下，Tomcat 从部署的 war 文件的名称中派生出它。因此，如果我们部署一个文件`ExampleApp.war`，它将在`http://localhost:8080/ExampleApp`可用。即上下文路径是`/ExampleApp`。

如果我们现在需要在`http://localhost:8080/`获得该应用，我们有几个选项，我们将在下面的部分中讨论。

关于 Tomcat 上下文概念的更详细的解释，请看一下官方的 [Tomcat 文档](https://web.archive.org/web/20220627074813/https://tomcat.apache.org/tomcat-8.5-doc/config/context.html)。

## 3.将应用程序部署为`ROOT.war`

第一个选项非常简单:**我们只需删除`$CATALINA_HOME\webapps`中默认的`/ROOT/`文件夹，将我们的`ExampleApp.war`重命名为`ROOT.war`，并部署它。**

我们的应用程序将于`http://localhost:8080/`推出。

## 4.在`server.xml`中指定上下文路径

第二个选项是在`server.xml`(位于`$CATALINA_HOME\conf`)中设置应用的上下文路径。

为此，我们必须在`<Host>`标签中插入以下内容:

```java
<Context path="" docBase="ExampleApp"></Context>
```

**注意:手动定义上下文路径的副作用是应用程序默认部署两次**:在`http://localhost:8080/ExampleApp/`和`http://localhost:8080/`。

为了防止这种情况，我们必须在`<Host>`标签中设置`autoDeploy=”false”`和`deployOnStartup=”false”`:

```java
<Host name="localhost" appBase="webapps" unpackWARs="true"
  autoDeploy="false" deployOnStartup="false">
    <Context path="" docBase="ExampleApp"></Context>

    <!-- Further settings for localhost -->
</Host>
```

**重要提示:不再推荐这个选项，因为 Tomcat 5:它使上下文配置更具侵入性，因为不重新启动 Tomcat 就不能重新加载`server.xml`文件。**

## 5.在特定于应用程序的 XML 文件中指定上下文路径

为了避免`server.xml`的这个问题，我们有了第三个选择:我们将在特定于应用程序的 XML 文件中设置上下文路径。

因此，我们必须在`$CATALINA_HOME\conf\Catalina\localhost`处创建一个`ROOT.xml`，其内容如下:

```java
<Context docBase="../deploy/ExampleApp.war"/>
```

两点在这里一文不值。

首先，我们不需要像前一个选项那样显式地指定路径——Tomcat 从我们的`ROOT.xml`的名称中派生出来。

第二——由于我们在不同于`server.xml`的文件中定义我们的上下文，我们的`docBase`必须在`$CATALINA_HOME\webApps`之外。

## 6.结论

在本教程中，我们讨论了如何在 Tomcat 的根目录下部署 web 应用程序的不同选项。