# Eclipse 错误:缺少 web.xml，并且 failOnMissingWebXml 设置为 true

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/eclipse-error-web-xml-missing>

## 1.介绍

在本教程中，我们将讨论常见的 Eclipse 错误，“缺少 **`web.xml`并且`<failOnMissingWebXml>`被设置为 true** ”，这是我们在创建 web 应用程序时遇到的。

## 2.Eclipse 错误

在 Java web 应用程序中， [`web.xml`](https://web.archive.org/web/20220525134822/https://docs.oracle.com/cd/E13222_01/wls/docs70/webapp/webappdeployment.html) 是部署描述符的标准名称。

我们可以使用 Maven 创建一个 web 应用程序，或者使用 Eclipse 创建一个动态 web 项目。Eclipse 不会在`WEB-INF/`目录下创建默认的部署描述符`web.xml`。

Java EE 6+规范试图弱化部署描述符，因为它们可以被注释所取代。但是，较低版本仍然需要它。

`failOnMissingWebXml`属性是 [Apache Maven](/web/20220525134822/https://www.baeldung.com/maven) war 插件的属性之一，`org.apache.maven.plugins:maven-war-plugin.` 对于版本< 3.1.0，该插件的默认值为`true`，对于以后的版本，默认值为`false`。

这意味着，如果我们使用的是 3.1.0 版本之前的`maven-war-plugin`，而`web.xml`文件不存在，那么将它打包成 war 文件的目标将会失败。

## 3.使用`web.xml`

对于所有我们仍然需要`web.xml`部署描述符的情况，我们可以很容易地**在 Eclipse** 中生成`web.xml`:

*   右键单击 web 项目
*   悬停到菜单上的 **Java EE 工具**
*   从子菜单中选择**生成部署描述符**存根

[![](img/1a0725807723ece45859f8ef886e49f7.png)](/web/20220525134822/https://www.baeldung.com/wp-content/uploads/2019/03/generatestub.jpg)

瞧啊。`web.xml`文件在`WEB-INF/`目录下生成。

## 4.不带`web.xml`

在大多数情况下，我们可能根本不需要`web.xml`文件。我们可以简单地跳过创建它，而不是在我们的项目中保留一个空白的`web.xml`文件。幸运的是，根据我们使用的`maven-war-plugin`的版本，有两种简单的方法。

### 4.1.使用 3.1.0 之前的 maven-war-plugin

我们可以在我们的`pom.xml`的`<plugins>`部分配置一个 Maven 项目的所有插件。正如我们之前说过的，在插件版本 3.1.0 之前，`failOnMissingWebXml`的默认值是`true`。

让我们声明我们的`pom.xml`和**中的`maven-war-plugin`显式设置属性`failOnMissingWebXml`为`false`** :

```java
<plugin>
    <artifactId>maven-war-plugin</artifactId>
    <version>2.4</version>
    <configuration>
        <failOnMissingWebXml>false</failOnMissingWebXml>    
    </configuration>
</plugin>
```

### 4.2.使用 maven-war-plugin 3.1.0 和更高版本

我们也可以通过升级`maven-war-plugin`的版本来避免显式设置属性。对于 3.1.0 及更高版本的`maven-war-plugin`，属性`failOnMissingWebXml`的默认值为`false`:

```java
<plugin>
    <artifactId>maven-war-plugin</artifactId>
    <version>3.1.0</version>
</plugin>
```

## 5.结论

在本文中，我们看到了遗漏`web.xml`错误背后的原因以及修复它的多种方法。

像往常一样，我们的例子可以在 GitHub 上找到[。](https://web.archive.org/web/20220525134822/https://github.com/eugenp/tutorials/tree/master/maven-modules/maven-war-plugin)