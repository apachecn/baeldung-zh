# 从 Shell 脚本调用 JMX MBean 方法

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/jmx-mbean-shell-access>

## 1.介绍

在本教程中，我们将看到如何从一个 shell 脚本和最常用的可用工具访问[mbean](/web/20221125062820/https://www.baeldung.com/java-management-extensions)。

**首先要注意的是 [JMX](/web/20221125062820/https://www.baeldung.com/java-management-extensions) 是基于 [RMI](/web/20221125062820/https://www.baeldung.com/java-rmi) 的。所以，协议的处理是在 Java 中。但是我们可以将它包装在一个 shell 脚本中，从命令行调用它。换句话说，如果我们想实现自动化，这一点尤其有用。**

尽管易于实现，大多数 JMX 工具被放弃或变得不可用。所以让我们检查几个工具，然后编写我们自己的。

## 2.编写一个简单的 MBean

为了测试我们的工具，我们需要一个 MBean。**首先，让我们创建一个简单的计算器，从它的接口开始:**

```java
public interface CalculatorMBean {

    Integer sum();

    Integer getA();

    void setA(Integer a);

    Integer getB();

    void setB(Integer b);
}
```

然后，让我们看看实现:

```java
public class Calculator implements CalculatorMBean {

    private Integer a = 0;
    private Integer b = 0;

    // getters and setters

    @Override
    public Integer sum() {
        return a + b;
    }
}
```

**现在让我们创建一个简单的 CLI 应用程序来注册我们的 MBean。**这段代码是标准的，关键部分是我们为 MBean 选择的实现类和名称。**我们将通过在`MBeanServer`上调用`registerMBean()`，传递一个`Calculator`的实例，并实例化一个`ObjectName`来实现。**

为了澄清，`ObjectName`接收一个任意的`String.`，但是为了[遵循惯例](https://web.archive.org/web/20221125062820/https://www.oracle.com/java/technologies/javase/management-extensions-best-practices.html)，我们将包含我们的包名作为域名，以及一个“`key=value`对列表:

```java
public class JmxCalculatorMain {

    public static void main(String[] args) throws Exception {
        MBeanServer server = ManagementFactory.getPlatformMBeanServer();
        server.registerMBean(
          new Calculator(), new ObjectName("com.baeldung.jxmshell:type=basic,name=calculator")
        );

        // ...
    }
} 
```

此外，唯一需要的键是“`type`”，这在我们的场景中是不相关的。但是这将我们的 MBean 与域中的其他 MBean 组合在一起。

**最后，为了防止我们的应用程序终止，我们将让它等待用户输入:**

```java
try (Scanner scanner = new Scanner(System.in)) {
    System.out.println("<press enter to terminate>");
    scanner.nextLine();
}
```

为了简化我们的示例，我们将在[端口](/web/20221125062820/https://www.baeldung.com/jmx-ports) `11234.`上运行我们的应用程序。同样，让我们使用这些 [JVM 参数](/web/20221125062820/https://www.baeldung.com/jvm-parameters)禁用身份验证和 SSL:

```java
-Dcom.sun.management.jmxremote.port=11234
-Dcom.sun.management.jmxremote.authenticate=false
-Dcom.sun.management.jmxremote.ssl=false
```

现在，我们准备尝试一些工具。

## 3.从 Shell 脚本使用 Jmxterm

[Jmxterm](https://web.archive.org/web/20221125062820/https://docs.cyclopsgroup.org/jmxterm) 是一个交互式 CLI 工具，它是用 [JConsole](https://web.archive.org/web/20221125062820/https://docs.oracle.com/javase/7/docs/technotes/guides/management/jconsole.html) 监控 MBeans 的替代工具。但是，通过一些输入操作，我们可以创建一个脚本来以非交互的方式使用它。先从下载 [jmxterm-1.0.4-uber.jar](https://web.archive.org/web/20221125062820/https://github.com/jiaqi/jmxterm/releases/tag/v1.0.4) 开始，放到`/tmp`文件夹。

### 3.1.将 Jmxterm 包装在脚本中

**我们将在脚本中使用 Jmxterm jar 的一些特性:执行 MBean 操作并获取或设置属性值。**为了简化它，我们将为它的`jar`位置、MBean `address`和`name, operation`以及最终的`command`定义默认值。

所以，让我们创建一个名为`jmxterm.sh:`的文件

```java
#!/bin/sh

jar='/tmp/jmxterm-1.0.4-uber.jar'
address='localhost:11234'

mbean='com.baeldung.jxmshell:name=calculator,type=basic'
operation='sum'
command="info -b $mbean"
```

我们应该注意到，`address`只有在运行`JmxCalculatorMain`之后才可以访问。我们将在`operation`变量中保存 MBean 的方法名或属性。同时，`command`变量保存发送给 Jmxterm 的最终命令。**作为默认值，`info`命令显示可用的操作和属性，`-b`指定要显示的 MBean。**

然后，我们将做一些[参数解析](/web/20221125062820/https://www.baeldung.com/linux/bash-parse-command-line-arguments)，通过构建`command`变量来处理`–run`、`–set`和`–get`选项:

```java
while test $# -gt 0
do
    case "$1" in
    --run)
        shift; operation=$1
        command="run -b ${mbean} ${operation}"
    ;;
    --set)
        shift; operation="$1"
        shift; attribute_value="$1"

        command="set -b ${mbean} ${operation} ${attribute_value}"
    ;;
    --get)
        shift; operation="$1"

        command="get -b ${mbean} ${operation}"
    ;;
    esac
    shift
done
```

**用`—` `run`，我们执行一个 MBean 方法。同样，我们用`—`和`set,`设置一个属性值。并且，用`—` `get`，我们得到它的当前值。**

最后，我们`[echo](/web/20221125062820/https://www.baeldung.com/linux/echo-command#:~:text=The%20echo%20command%20writes%20text%20to%20standard%20output%20(stdout).&text=Some%20common%20usages%20of%20the,echo%20command%20in%20later%20sections.)`将我们的`command`、[管道](/web/20221125062820/https://www.baeldung.com/linux/pipes-redirection)连同我们正在运行的应用程序的`address`一起传送到 Jmxterm jar。`-v silent`选项关闭详细性，而`-n`告诉 Jmxterm 不要打印用户提示。因此，这个选项很有帮助，因为我们没有交互地使用它:

```java
echo $command | java -jar $jar -l $address -n -v silent 
```

### 3.2.操纵我们的 MBean

在使我们的脚本[可执行](/web/20221125062820/https://www.baeldung.com/linux/chown-chmod-permissions)之后，我们将不带参数地运行它，以查看它的默认行为。假设脚本在[当前目录](/web/20221125062820/https://www.baeldung.com/linux/run-script-different-working-dir)中，让我们运行它:

```java
./jmxterm.sh
```

**这将查询我们的 MBean 上的信息:**

```java
# attributes
  %0   - A (java.lang.Integer, rw)
  %1   - B (java.lang.Integer, rw)
# operations
  %0   - java.lang.Integer sum()
```

我们可以注意到,`set`从我们的 setter 名称中移除，它们出现在`attributes`部分。

**那么，我们就叫`setA()` :**

```java
./jmxterm.sh --set A 1
```

因此，如果一切正常，就没有输出。**所以现在让我们调用`getA()`来检查当前值:**

```java
./jmxterm.sh --get A
```

下面是我们得到的输出:

```java
A = 1;
```

**现在，让我们将`B`设置为 2，并调用`sum()` :**

```java
./jmxter.sh --set B 2
./jmxter.sh --run sum
```

这是输出结果:

```java
3
```

这种解决方案对于简单的 MBean 操作非常有效。

## 4.从命令行调用 cmdline-jmxclient

我们的下一个工具是 [cmdline-jmxclient](https://web.archive.org/web/20221125062820/http://crawler.archive.org/cmdline-jmxclient) 。我们将使用版本 [0.10.3](https://web.archive.org/web/20221125062820/http://crawler.archive.org/cmdline-jmxclient/cmdline-jmxclient-0.10.3.jar) ，它的工作方式类似于 Jmxterm，但是没有交互选项。

为了简洁起见，让我们看一个如何从命令行调用它的例子。首先，我们将设置属性值，然后执行一个操作:

```java
jar=cmdline-jmxclient-0.10.3.jar
address=localhost:11234
mbean=com.baeldung.jxmshell:name=calculator,type=basic
java -jar $jar - $address $mbean A=1
java -jar $jar - $address $mbean B=1
java -jar $jar - $address $mbean sum
```

**值得注意的是，因为我们的 MBean 不需要认证，所以我们传递“`–`”而不是认证参数。**

以下是运行这些命令的输出:

```java
11/11/2022 22:10:15 -0300 org.archive.jmx.Client sum: 2
```

与此工具的主要区别在于其输出格式。此外，它要轻得多，大小约为 20 KB，而 Jmxterm 的大小超过 6 MB。

## 5.编写自定义解决方案

**因为可用的选项已经很老了，所以这是了解 MBeans 如何工作的绝佳时机。**所以让我们开始实现我们的解决方案，用一个包装器包装我们需要的来自`javax.management`包的类:

```java
public class JmxConnectionWrapper {

    private final Map<String, MBeanAttributeInfo> attributeMap;
    private final MBeanServerConnection connection;
    private final ObjectName objectName;

    public JmxConnectionWrapper(String url, String beanName) throws Exception {
        objectName = new ObjectName(beanName);

        connection = JMXConnectorFactory.connect(new JMXServiceURL(url))
          .getMBeanServerConnection();

        MBeanInfo bean = connection.getMBeanInfo(objectName);
        MBeanAttributeInfo[] attributes = bean.getAttributes();

        attributeMap = Stream.of(attributes)
          .collect(Collectors.toMap(MBeanAttributeInfo::getName, Function.identity()));
    }

    // ...
}
```

首先，我们的构造函数接收一个要连接的 URL 和一个 MBean 名称。我们保留一个对`ObjectName`的引用，以便在将来的调用中使用，然后开始与`JMXConnectorFactory`的连接。然后，我们得到一个`MBeanInfo`引用，从中提取属性。最后，我们将`MBeanAttributeInfo[]`转换成地图。**我们稍后将使用这个映射来确定一个参数是属性还是操作。**

现在，让我们写一些助手方法。我们将从获取和设置属性值的方法开始。当我们收到一个值时，我们在返回当前值之前设置它:

```java
public Object attributeValue(String name, String value) throws Exception {
    if (value != null)
        connection.setAttribute(objectName, new Attribute(name, Integer.valueOf(value)));

    return connection.getAttribute(objectName, name);
}
```

因为我们知道我们的 MBean 只包含`Integer`属性，所以我们使用`Integer.valueOf()`来构造我们的`Attribute` 值。**但是，对于一个更健壮的解决方案，我们需要使用`attributeMap`中的信息来确定正确的类型。**

同样，因为我们知道我们的 MBean 只包含无参数操作，所以我们在我们的`connection`上调用`invoke()`，并为`params`和`signature`传递空数组:

```java
public Object invoke(String operation) throws Exception {
    Object[] params = new Object[] {};
    String[] signature = new String[] {};
    return connection.invoke(objectName, operation, params, signature);
}
```

我们将使用它来调用 MBean 中与属性无关的任何方法。

### 5.1 编写 CLI 部分

最后，让我们创建一个 [CLI 应用程序](/web/20221125062820/https://www.baeldung.com/java-command-line-arguments)来从 shell 操作我们的 MBean:

```java
public class JmxInvoker {

    public static void main(String... args) throws Exception {
        String attributeValue = null;
        if (args.length > 3) {
            attributeValue = args[3];
        }

        String result = execute(args[0], args[1], args[2], attributeValue);
        System.out.println(result);
    }

    public static String execute(
      String url, String mBeanName, String operation, String attributeValue) {
        JmxConnectionWrapper connection = new JmxConnectionWrapper(url, mBeanName);

        // ...
    }
}
```

我们的`main()`方法将参数传递给`execute()`，后者对参数进行处理以决定我们是要执行操作还是获取/设置属性:

```java
if (connection.hasAttribute(operation)) {
    Object value = connection.attributeValue(operation, attributeValue);
    return operation + "=" + value;
} else {
    Object result = connection.invoke(operation);
    return operation + "(): " + result;
}
```

**如果我们的 MBean 包含一个与我们的操作同名的属性，我们就调用`attributeValue()`。否则我们叫`invoke()`。**

### 5.2.从命令行调用它

假设我们[将我们的应用程序打包在一个 jar](/web/20221125062820/https://www.baeldung.com/java-create-jar) 中，并将其放在由`jar`变量指定的位置，让我们定义我们的默认值:

```java
jar=/tmp/jmx-invoker.jar
address='service:jmx:rmi:///jndi/rmi://localhost:11234/jmxrmi'
invoker='com.baeldung.jmxshell.custom.JmxInvoker'
mbean='com.baeldung.jxmshell:name=calculator,type=basic' 
```

**然后，我们运行这些命令来设置属性，并在我们的`MBean`中执行`sum()`方法，与前面的解决方案类似:**

```java
$ java -cp $jar $invoker $address $mbean A 1
A=1

$ java -cp $jar $invoker $address $mbean B 1
B=1
$ java -cp $jar $invoker $address $mbean sum
sum(): 2
```

这个自定义的`JmxInvoker`有助于构建 mbean 处理程序，并在解析 mbean 时拥有完全的控制权。

## 6.结论

在本文中，我们看到了用于管理 MBeans 的工具，以及如何从 shell 脚本中使用它们。然后，我们创建了一个自定义工具来处理它们。最后，存储库中提供了更多的脚本示例。

和往常一样，源代码可以从 GitHub 上的[处获得。](https://web.archive.org/web/20221125062820/https://github.com/eugenp/tutorials/tree/master//core-java-modules/core-java-perf)