# 将 Tomcat HTTP 端口更改为 80

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/tomcat-change-port>

## 1。概述

默认情况下，Apache Tomcat 运行在端口`8080`上。在某些情况下，这个端口可能已经被另一个进程占用，或者要求我们必须使用一个不同的端口。

在这篇简短的文章中，我们将展示如何更改 Apache Tomcat 服务器的 HTTP 端口。在我们的例子中，我们将使用端口`80`,尽管这个过程对于任何端口都是一样的。

## 2。Apache Tomcat 配置

这个过程的第一步是修改 Apache Tomcat 配置。

首先，我们定位服务器的`<TOMCAT_HOME>/conf/server.xml`文件。然后我们找到配置 HTTP 连接器端口的行:

```java
<Connector connectionTimeout="20000" port="8080" protocol="HTTP/1.1" redirectPort="8443"/>
```

我们将端口更改为`80`:

```java
<Connector connectionTimeout="20000" port="80" protocol="HTTP/1.1" redirectPort="8443"/>
```

## 3。Linux 和 Unix 系统变化

在 Linux 和 Unix 系统上，`1024`下面的**端口号是[特权端口](https://web.archive.org/web/20220627171218/https://www.w3.org/Daemon/User/Installation/PrivilegedPorts.html)，保留给运行为`root`** 的程序。如果我们运行在端口`1024`或更高的端口上，那么我们可以跳过这一节的剩余部分，直接按照第 4 节的解释开始/重启我们的服务器。

**如果我们拥有`root`或`sudo`访问**的权限，我们可以使用以下命令作为 root 用户启动 Tomcat 进程:

```java
sudo startup.sh
```

**但是如果我们没有`root`或`sudo`访问**，我们将不得不安装和配置`authbind`，如下所述。

**注意:当使用非特权端口** ( `1024`或更高)时，我们可以跳过这一节的剩余部分，直接进入启动/重启我们的服务器。

### 3.1。安装`authbind`包

**对于基于 Linux 的系统:**下载并安装`authbind`包:

```java
sudo apt-get install authbind
```

**对于 MacOS 系统:**首先，从[这里](https://web.archive.org/web/20220627171218/https://github.com/Castaglia/MacOSX-authbind)下载 MacOS 的`authbind`，展开包。然后进入扩展目录进行构建和安装:

```java
$ cd MacOSX-authbind
$ make
$ sudo make install
```

### 3.2。在 Apache Tomcat 上启用`authbind`

打开`<TOMCAT_HOME>/conf/server.xml`文件取消对以下行的注释:

```java
AUTHBIND=yes
```

### 3.3。启用端口的读取和执行

现在，我们需要执行一些命令来启用端口的读取和执行权限。

下面是一个使用 Tomcat 版的示例:

```java
sudo touch <AUTHBIND_HOME>/byport/80
sudo chmod 500 <AUTHBIND_HOME>/byport/80
sudo chown tomcat8 <AUTHBIND_HOME>/byport/80
```

注意:如果使用 Tomcat 版本 6 或 7，那么我们将在最后一个命令中分别使用`tomcat6`或`tomcat7`，而不是`tomcat8`。

### 3.4。使用旧版本的`authbind`

如果使用不支持 IPv6 的旧版本`authbind` ( [版本低于 2.0.0](https://web.archive.org/web/20220627171218/https://bugs.debian.org/cgi-bin/bugreport.cgi?bug=596921) )，我们需要将 IPv4 设为默认版本。

**如果我们已经有了一个`<TOMCAT_HOME>/bin/setenv.sh`文件，那么替换:**

```java
exec "$PRGDIR"/"$EXECUTABLE" start "[[email protected]](/web/20220627171218/https://www.baeldung.com/cdn-cgi/l/email-protection)"
```

**用这一行:**

```java
exec authbind --deep "$PRGDIR"/"$EXECUTABLE" start "[[email protected]](/web/20220627171218/https://www.baeldung.com/cdn-cgi/l/email-protection)"
```

**然后添加下面一行:**

```java
export CATALINA_OPTS="$CATALINA_OPTS -Djava.net.preferIPv4Stack=true"
```

**如果我们还没有`<TOMCAT_HOME>/bin/setenv.sh`文件，那么使用**创建一个

```java
exec authbind --deep "$PRGDIR"/"$EXECUTABLE" start "[[email protected]](/web/20220627171218/https://www.baeldung.com/cdn-cgi/l/email-protection)"
export CATALINA_OPTS="$CATALINA_OPTS -Djava.net.preferIPv4Stack=true"
```

## 4。重启服务器

现在，我们已经对我们的配置做了所有必要的更改，我们可以启动或重启 Tomcat 服务器，并在端口`80`上访问它。

## 5。结论

在本文中，我们展示了如何将 Apache Tomcat 的端口从默认的`8080`更改为端口`80`。值得注意的是，对于 Tomcat 版本 [6.x](https://web.archive.org/web/20220627171218/https://tomcat.apache.org/tomcat-6.0-doc/config/http.html) 、 [7.x](https://web.archive.org/web/20220627171218/https://tomcat.apache.org/tomcat-7.0-doc/config/http.html) 、 [8.x](https://web.archive.org/web/20220627171218/https://tomcat.apache.org/tomcat-7.0-doc/config/http.html) 来说，流程是一样的。