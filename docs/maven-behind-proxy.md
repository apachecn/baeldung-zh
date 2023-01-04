# 在代理后面使用 Maven

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/maven-behind-proxy>

## 1.介绍

在本教程中，我们将**配置 [Maven](/web/20220529030000/https://www.baeldung.com/maven) 在代理**后面工作——这是我们不直接连接到互联网的环境中的常见情况。

在我们的示例中，我们的代理运行在机器“proxy.baeldung.com”上，它通过端口“80”上的 HTTP 侦听代理请求。我们还将使用 internal.baeldung.com 的一些内部网站，在那里我们不需要通过代理。

## 2.代理配置

首先，**让我们设置一个没有任何凭证的基本代理配置**。

让我们编辑一下我们的 Maven [`settings.xml`](https://web.archive.org/web/20220529030000/https://maven.apache.org/ref/3.6.3/maven-settings/settings.html) 通常在我们的'`<user_home>/.m2′`目录中找到。如果那里还没有，那么我们可以从‘`<maven_home>/conf'`目录下的全局设置中复制它。

现在让我们在`<[proxies](https://web.archive.org/web/20220529030000/https://maven.apache.org/settings.html#Proxies)>`部分中创建一个`<proxy>`条目:

```java
<proxies>
   <proxy>
        <host>proxy.baeldung.com</host>
        <port>80</port>
    </proxy>
</proxies>
```

因为我们也使用不需要通过代理的本地站点，所以让我们在`<nonProxyHosts>`中使用“|”分隔的列表和我们的本地主机来指定它:

```java
<nonProxyHosts>internal.baeldung.com|localhost|127.*|[::1]</nonProxyHosts> 
```

## 3.添加凭据

如果我们的代理不安全，那就是我们所需要的；然而，我们的是，所以**让我们将我们的凭证添加到代理定义**:

```java
<username>baeldung</username>
<password>changeme</password> 
```

**如果不需要，我们不会添加用户名/密码条目**-即使是空条目，因为当代理不需要它们时，让它们出现会导致我们的请求被拒绝。

我们的最低认证配置现在应该是这样的:

```java
<proxies>
   <proxy>
        <host>proxy.baeldung.com</host>
        <port>80</port>
        <username>baeldung</username>
        <password>changeme</password>
        <nonProxyHosts>internal.baeldung.com|localhost|127.*|[::1]</nonProxyHosts>
    </proxy>
</proxies>
```

现在，当我们运行`mvn `命令时，我们将通过代理连接到我们要寻找的站点。

### 3.1.可选条目

让我们给它一个可选的“BaeldungProxy_Authenticated”的`id`,以便在我们需要切换代理时更容易引用:

```java
<id>BaeldungProxy_Authenticated</id>
```

现在，如果我们有另一个代理，我们可以添加另一个代理定义，但只能有一个是活动的。**默认情况下，Maven 将使用它找到的第一个活动代理定义**。

默认情况下，代理定义是活动的，并获得隐式定义:

```java
<active>true</active>
```

如果我们想让另一个代理成为活动代理，那么我们可以通过将`<active>`设置为 false 来取消激活我们的原始条目:

```java
<active>false</active>
```

**Maven 对代理协议的默认值是 HTTP** ，这适合大多数情况。如果我们的代理使用不同的协议，我们应该在这里声明它，并用我们的代理需要的协议替换`http`:

```java
<protocol>http</protocol>
```

注意，这是代理使用的协议——我们请求的协议(ftp://，http://，https://)与此无关。

这里是我们扩展的代理定义的样子，包括可选元素:

```java
<proxies>
   <proxy>
        <id>BaeldungProxy_Authenticated</id>
        <active>true</active>
        <protocol>http</protocol>
        <host>proxy.baeldung.com</host>
        <port>80</port>
        <username>baeldung</username>
        <password>changeme</password>
        <nonProxyHosts>internal.baeldung.com|localhost|127.*|[::1]</nonProxyHosts>
    </proxy>
</proxies>
```

这就是我们基本的代理入口，但是它对我们来说足够安全吗？

## 4.保护我们的配置

现在，假设我们的一位同事希望我们将代理配置发送给他们。

我们不太喜欢以纯文本的形式发送我们的密码，所以**让我们看看 Maven 如何轻松地[加密我们的密码](https://web.archive.org/web/20220529030000/https://maven.apache.org/guides/mini/guide-encryption.html)** 。

### 4.1.创建主密码

首先，让我们选择一个主密码，说“te！st！马$特尔”。

现在**让我们加密我们的主密码**，在运行时的提示符下输入它:

```java
mvn --encrypt-master-password
Master Password: 
```

在我们按下 enter 后，我们会看到我们的加密密码被括在花括号中:

```java
{QFMlh/6WjF8H9po9UDo0Nv18e527jqWb6mUgIB798n4=}
```

### 4.2.密码生成故障排除

如果我们看到{}而不是`Master Password:`提示符(使用 bash 时会发生这种情况)，那么我们需要在命令行上指定密码。

**让我们用引号将密码括起来，以确保任何特殊字符，如'！'不要有不必要的影响。**

因此，如果我们使用 bash，就用单引号:

```java
mvn --encrypt-master-password 'te!st!ma$ter'
```

如果使用 Windows 命令提示符，请使用双引号:

```java
mvn --encrypt-master-password "te!st!ma$ter"
```

现在，**有时我们生成的主密码包含花括号**，就像这个例子，在“UD”后面有一个右花括号“}”:

```java
{QFMlh/6WjF8H9po9UD}0Nv18e527jqWb6mUgIB798n4=}
```

在这种情况下，我们可以:

*   再次运行`mvn –encrypt-master-password`命令生成另一个命令(希望没有花括号)
*   通过在' { '或`‘` } '前面添加一个反斜杠来转义我们密码中的花括号

### 4.3.创建一个`settings-security.xml`文件

现在让我们把加密的密码，加上转义的“\}”，放入**目录**中一个名为`settings-security.xml`的文件中:

```java
<settingsSecurity>
    <master>{QFMlh/6WjF8H9po9UD\}0Nv18e527jqWb6mUgIB798n4=}</master>
</settingsSecurity>
```

最后，Maven 让我们在 master 元素中添加注释。

让我们在密码分隔符' { '之前添加一些文本，注意不要在注释中使用{或}，因为 Maven 使用它们来查找我们的密码:

```java
<master>We escaped the curly brace with '\' {QFMlh/6WjF8H9po9UD\}0Nv18e527jqWb6mUgIB798n4=}</master>
```

### 4.4.使用可移动驱动器

假设我们需要额外的安全，并且**想要将我们的主密码存储在一个单独的设备**上。

首先，我们将把我们的`settings-security.xml`文件放在可移动驱动器上的 config 目录中，“R:

```java
R:\config\settings-security.xml
```

现在，我们将更新我们的`.m2`目录中的`settings-security.xml`文件，将 Maven 重定向到我们可移动驱动器上的真正的`settings-security.xml`:

```java
<settingsSecurity>
    <relocation>R:\config\settings-security.xml</relocation>
</settingsSecurity>
```

**Maven 现在将从我们在可移动驱动器上的`relocation`元素**中指定的文件中读取我们的加密主密码。

## 5.加密代理密码

现在我们有了加密的主密码，我们可以**加密我们的代理密码**。

让我们运行以下命令，并在提示符下输入我们的密码"`changeme”,`:

```java
mvn --encrypt-password
Password:
```

显示我们的加密密码:

```java
{U2iMf+7aJXQHRquuQq6MX+n7GOeh97zB9/4e7kkEQYs=}
```

我们的最后一步是**编辑 settings.xml 文件中的代理部分，并输入加密密码**:

```java
<proxies>
   <proxy>
        <id>BaeldungProxy_Encrypted</id>
        <host>proxy.baeldung.com</host>
        <port>80</port>
        <username>baeldung</username>
        <password>{U2iMf+7aJXQHRquuQq6MX+n7GOeh97zB9/4e7kkEQYs=}</password>
    </proxy>
</proxies>
```

保存这个，Maven 现在应该能够通过我们的代理，使用我们的加密密码连接到互联网。

## 6.使用系统属性

虽然通过设置文件配置 Maven 是推荐的方法，但是我们可以通过 Java 系统属性声明我们的代理配置。

如果我们的操作系统已经配置了代理，我们可以设置:

```java
-Djava.net.useSystemProxies=true
```

或者，如果我们有管理员权限，我们可以在我们的`<JRE>/lib/net.properties`文件中设置它，以便它总是被启用。

但是，让我们注意一下，虽然 Maven 本身可能尊重这个设置，但并不是所有的插件都这么做，所以我们仍然可能使用这个方法得到失败的连接。

即使启用了，我们也可以通过**在`http.proxyHost`系统属性**上设置 HTTP 代理的详细信息来覆盖它:

```java
-Dhttp.proxyHost=proxy.baeldung.com
```

我们的代理监听默认端口 80，但是如果它监听端口 8080，我们将**配置`http.proxyPort`属性**:

```java
-Dhttp.proxyPort=8080
```

并且**对于我们不需要代理**的站点:

```java
-Dhttp.nonLocalHosts="internal.baeldung.com|localhost|127.*|[::1]"
```

因此，如果我们的代理在 10.10.0.100，我们可以使用:

```java
mvn compile -Dhttp.proxyHost=10.10.0.100 -Dhttp.proxyPort=8080 -Dhttp.nonProxyHosts=localhost|127.0.0.1
```

当然，**如果我们的代理需要认证**，**我们也会添加**:

```java
-Dhttp.proxyUser=baeldung
-Dhttp.proxyPassword=changeme
```

如果我们希望这些设置应用于所有的 Maven 调用，我们可以在 MAVEN_OPTS 环境变量中定义它们:

```java
set MAVEN_OPTS= -Dhttp.proxyHost=10.10.0.100 -Dhttp.proxyPort=8080
```

现在，每当我们运行'`mvn`'时，这些设置将自动应用-直到我们退出。

## 7.结论

在本文中，我们配置了一个有凭证和没有凭证的 Maven 代理，并加密了我们的密码。我们看到了如何在外部驱动器上存储我们的主密码，还看到了如何使用系统属性配置代理。

现在，我们可以与同事共享我们的`settings.xml`文件，而不必以纯文本的形式给出我们的密码，并向他们展示如何加密他们的密码！

像往常一样，本文中的例子可以在 GitHub 上的[处找到。](https://web.archive.org/web/20220529030000/https://github.com/eugenp/tutorials/tree/master/maven-modules/maven-proxy)