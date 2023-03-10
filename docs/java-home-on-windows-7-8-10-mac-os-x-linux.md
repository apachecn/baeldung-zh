# 在 Windows 7、8、10、Mac OS X、Linux 上设置 JAVA_HOME

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-home-on-windows-7-8-10-mac-os-x-linux>

## 1。概述

在这个快速教程中，我们将看看如何在 Windows、Mac OS X 和 Linux 上设置`JAVA_HOME`变量。

## 2。窗户

### 2.1。Windows 10 和 8

1.  打开**搜索**，输入**高级系统设置。**
2.  在显示的选项中，选择**查看高级系统设置**链接。
3.  在**高级**选项卡下，点击**环境变量。**
4.  在**系统变量**部分，点击**新建**(或**用户变量**进行单用户设置)。
5.  将`JAVA_HOME`设置为**变量名**，将**变量值**设置为 JDK 安装路径，点击**确定。**
6.  点击**确定**并点击**应用**以应用更改。

### 2.2。Windows 7

1.  在桌面上，右键单击**我的电脑**，选择**属性。**
2.  在**高级**选项卡下，点击**环境变量。**
3.  在**系统变量**部分，点击**新建**(或**用户变量**进行单用户设置)。
4.  将`JAVA_HOME`设置为**变量名**，将**变量值**设置为 JDK 安装路径，点击**确定。**
5.  点击**确定**并点击**应用**以应用更改。

打开命令提示符并检查`JAVA_HOME`变量的值:

```java
echo %JAVA_HOME%
```

结果应该是 JDK 安装的路径:

```java
C:\Program Files\Java\jdk1.8.0_111
```

## 3。麦克·OS X

### 3.1。单用户–Mac OS X 10.5 或更新版本

从 OS X 10.5 开始，苹果引入了一个[命令行工具](https://web.archive.org/web/20220811232139/https://developer.apple.com/library/content/qa/qa1170/_index.html) ( `/usr/libexec/java_home`)，可以动态地为当前用户找到 Java 首选项中指定的最高 Java 版本。

在任何文本编辑器中打开`~/.bash_profile`并添加以下内容:

```java
export JAVA_HOME=$(/usr/libexec/java_home)
```

保存并关闭文件。

打开终端并运行 source 命令以应用更改:

```java
source ~/.bash_profile
```

现在我们可以检查`JAVA_HOME`变量的值:

```java
echo $JAVA_HOME
```

结果应该是 JDK 安装的路径:

```java
/Library/Java/JavaVirtualMachines/jdk1.8.0_111.jdk/Contents/Home
```

### 3.2。单用户–Mac OS X 旧版本

对于旧版本的 OS X，我们必须设置 JDK 安装的确切路径。

在任何编辑器中打开`~/.bash_profile`,并添加以下内容:

```java
export JAVA_HOME=/path/to/java_installation
```

保存并关闭文件。

打开终端并运行 source 命令以应用更改:

```java
source ~/.bash_profile
```

现在我们可以检查`JAVA_HOME`变量的值:

```java
echo $JAVA_HOME
```

结果应该是 JDK 安装的路径:

```java
/Library/Java/JavaVirtualMachines/jdk1.8.0_111.jdk/Contents/Home
```

### 3.3。全局设置

要为所有用户全局设置`JAVA_HOME`，步骤与单个用户相同，但是我们使用文件`/etc/profile`。

## 4。Linux

当然，我们将在这里操纵路径，所以[这里是关于如何做的详细说明](/web/20220811232139/https://www.baeldung.com/linux/path-variable)。

### 4.1。单一用户

要在 Linux 中为单个用户设置`JAVA_HOME`，我们可以使用`/etc/profile`或`/etc/environment`(系统级设置首选)或~/。bashrc(用户特定设置)。

在任何文本编辑器中打开~ `/.bashrc`,并添加以下内容:

```java
export JAVA_HOME=/path/to/java_installation
```

保存并关闭文件。

运行 source 命令加载变量:

```java
source ~/.bashrc
```

现在我们可以检查`JAVA_HOME`变量的值:

```java
echo $JAVA_HOME
```

结果应该是 JDK 安装的路径:

```java
/usr/lib/jvm/java-8-oracle
```

### 4.2。全局设置

要在 Linux 中为所有用户设置`JAVA_HOME`，我们可以使用`/etc/profile`或`/etc/environment`(首选)。

在任何文本编辑器中打开`/etc/environment`并添加以下内容:

```java
JAVA_HOME=/path/to/java_installation
```

请注意，`/etc/environment`不是一个脚本，而是一个赋值表达式列表(这就是为什么没有使用`export`)。该文件在登录时被读取。

要使用`/etc/profile`设置`JAVA_HOME`，我们将在文件中添加以下内容:

```java
export JAVA_HOME=/path/to/java_installation
```

运行 source 命令加载变量:

```java
source /etc/profile
```

现在我们可以检查`JAVA_HOME`变量的值:

```java
echo $JAVA_HOME
```

结果应该是 JDK 安装的路径:

```java
/usr/lib/jvm/java-8-oracle
```

## 5。结论

在本文中，我们介绍了在 Windows、Mac OS X 和 Linux 上设置`JAVA_HOME`环境变量的方法。