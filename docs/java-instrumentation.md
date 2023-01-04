# Java 工具指南

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-instrumentation>

## 1。简介

在本教程中，我们将讨论 [Java Instrumentation API。](https://web.archive.org/web/20221114003540/https://docs.oracle.com/en/java/javase/11/docs/api/java.instrument/java/lang/instrument/Instrumentation.html)它提供了向已编译的 Java 类添加字节码的能力。

我们还将讨论 java 代理以及如何使用它们来检测我们的代码。

## 2。设置

在整篇文章中，我们将使用工具构建一个应用程序。

我们的应用程序将由两个模块组成:

1.  允许我们取钱的 ATM 应用程序
2.  还有一个 Java 代理，它将允许我们通过测量花费的时间来测量我们的 ATM 的性能

Java 代理将修改 ATM 字节码，使我们无需修改 ATM 应用程序即可测量取款时间。

我们的项目将具有以下结构:

```java
<groupId>com.baeldung.instrumentation</groupId>
<artifactId>base</artifactId>
<version>1.0.0</version>
<packaging>pom</packaging>
<modules>
    <module>agent</module>
    <module>application</module>
</modules>
```

在深入研究插装的细节之前，让我们看看什么是 java 代理。

## 3。什么是 Java 代理

一般来说，java 代理只是一个特制的 jar 文件。它利用 JVM 提供的[检测 API](https://web.archive.org/web/20221114003540/https://docs.oracle.com/en/java/javase/11/docs/api/java.instrument/java/lang/instrument/Instrumentation.html) 来改变 JVM 中加载的现有字节码。

为了让代理工作，我们需要定义两个方法:

*   `premain`–将在 JVM 启动时使用-javaagent 参数静态加载代理
*   `agentmain`–将使用 [Java Attach API](https://web.archive.org/web/20221114003540/https://docs.oracle.com/en/java/javase/11/docs/api/jdk.attach/module-summary.html) 将代理动态加载到 JVM 中

需要记住的一个有趣的概念是，JVM 实现，如 Oracle、OpenJDK 等，可以提供一种动态启动代理的机制，但这不是必需的。

首先，让我们看看如何使用现有的 Java 代理。

之后，我们将看看如何从头创建一个来添加我们在字节码中需要的功能。

## 4。加载 Java 代理

为了能够使用 Java 代理，我们必须首先加载它。

我们有两种类型的负载:

*   静态——通过使用-javaagent 选项，利用`premain`来加载代理
*   动态——利用`agentmain` 将代理加载到使用 [Java 附加 API](https://web.archive.org/web/20221114003540/https://docs.oracle.com/en/java/javase/11/docs/api/jdk.attach/module-summary.html) 的 JVM 中

接下来，我们将了解每种类型的负载，并解释其工作原理。

### 4.1。静态负载

在应用程序启动时加载 Java 代理称为静态加载。在执行任何代码之前，静态加载会在启动时修改字节码。

请记住，静态加载使用`premain`方法，它将在任何应用程序代码运行之前运行，为了让它运行，我们可以执行:

```java
java -javaagent:agent.jar -jar application.jar
```

需要注意的是，我们应该始终将–`javaagent `参数放在–`jar `参数之前。

以下是我们命令的日志:

```java
22:24:39.296 [main] INFO - [Agent] In premain method
22:24:39.300 [main] INFO - [Agent] Transforming class MyAtm
22:24:39.407 [main] INFO - [Application] Starting ATM application
22:24:41.409 [main] INFO - [Application] Successful Withdrawal of [7] units!
22:24:41.410 [main] INFO - [Application] Withdrawal operation completed in:2 seconds!
22:24:53.411 [main] INFO - [Application] Successful Withdrawal of [8] units!
22:24:53.411 [main] INFO - [Application] Withdrawal operation completed in:2 seconds!
```

我们可以看到`premain`方法何时运行，以及`MyAtm `类何时被转换。我们还看到了两个 ATM 取款交易日志，其中包含了完成每项操作所需的时间。

请记住，在我们最初的应用程序中，我们没有事务的这个完成时间，它是由我们的 Java 代理添加的。

### 4.2。动态负载

将 Java 代理加载到已经运行的 JVM 中的过程称为动态加载。使用 [Java 附加 API](https://web.archive.org/web/20221114003540/https://docs.oracle.com/en/java/javase/11/docs/api/jdk.attach/module-summary.html) 来附加代理。

一个更复杂的场景是，我们已经在生产环境中运行了 ATM 应用程序，我们希望在应用程序不停机的情况下动态增加交易的总时间。

让我们写一小段代码来完成这个任务，我们称这个类为`AgentLoader. `为了简单起见，我们将这个类放在应用程序 jar 文件中。因此，我们的应用程序 jar 文件既可以启动我们的应用程序，又可以将我们的代理附加到 ATM 应用程序:

```java
VirtualMachine jvm = VirtualMachine.attach(jvmPid);
jvm.loadAgent(agentFile.getAbsolutePath());
jvm.detach();
```

现在我们有了`AgentLoader`，我们开始我们的应用程序，确保在事务之间的十秒钟暂停中，我们将使用`AgentLoader`动态连接我们的 Java 代理。

让我们也添加胶水，这将允许我们启动应用程序或加载代理。

我们将这个类称为`Launcher`，它将是我们的主 jar 文件类:

```java
public class Launcher {
    public static void main(String[] args) throws Exception {
        if(args[0].equals("StartMyAtmApplication")) {
            new MyAtmApplication().run(args);
        } else if(args[0].equals("LoadAgent")) {
            new AgentLoader().run(args);
        }
    }
}
```

#### 启动应用

```java
java -jar application.jar StartMyAtmApplication
22:44:21.154 [main] INFO - [Application] Starting ATM application
22:44:23.157 [main] INFO - [Application] Successful Withdrawal of [7] units!
```

#### 附加 Java 代理

在第一次操作之后，我们将 java 代理连接到我们的 JVM:

```java
java -jar application.jar LoadAgent
22:44:27.022 [main] INFO - Attaching to target JVM with PID: 6575
22:44:27.306 [main] INFO - Attached to target JVM and loaded Java agent successfully 
```

#### 查看应用日志

现在我们将代理连接到 JVM，我们将看到第二个 ATM 取款操作的总完成时间。

这意味着我们在应用程序运行的同时动态添加了我们的功能:

```java
22:44:27.229 [Attach Listener] INFO - [Agent] In agentmain method
22:44:27.230 [Attach Listener] INFO - [Agent] Transforming class MyAtm
22:44:33.157 [main] INFO - [Application] Successful Withdrawal of [8] units!
22:44:33.157 [main] INFO - [Application] Withdrawal operation completed in:2 seconds!
```

## 5。创建 Java 代理

在学习了如何使用代理之后，让我们看看如何创建一个代理。我们将看看[如何使用 Javassist](/web/20221114003540/https://www.baeldung.com/javassist) 来改变字节码，我们将把它与一些工具 API 方法结合起来。

因为 java 代理使用了 [Java Instrumentation API](https://web.archive.org/web/20221114003540/https://docs.oracle.com/en/java/javase/11/docs/api/java.instrument/java/lang/instrument/Instrumentation.html) ，所以在深入创建我们的代理之前，让我们先来看看这个 API 中一些最常用的方法以及它们的作用的简短描述:

*   `addTransformer`–向仪器引擎添加变压器
*   `getAllLoadedClasses`–返回当前由 JVM 加载的所有类的数组
*   `retransformClasses`–通过添加字节码来促进已加载类的检测
*   `removeTransformer`–取消注册所提供的变压器
*   `redefineClasses`–使用提供的类文件重新定义提供的类集，这意味着该类将被完全替换，而不是像`retransformClasses`那样被修改

### 5.1。创建`Premain`和`Agentmain`方法

我们知道每个 Java 代理至少需要一个`premain`或`agentmain`方法。后者用于动态加载，而前者用于将 java 代理静态加载到 JVM 中。

让我们在代理中定义它们，这样我们就能够静态和动态地加载这个代理:

```java
public static void premain(
  String agentArgs, Instrumentation inst) {

    LOGGER.info("[Agent] In premain method");
    String className = "com.baeldung.instrumentation.application.MyAtm";
    transformClass(className,inst);
}
public static void agentmain(
  String agentArgs, Instrumentation inst) {

    LOGGER.info("[Agent] In agentmain method");
    String className = "com.baeldung.instrumentation.application.MyAtm";
    transformClass(className,inst);
}
```

在每个方法中，我们声明我们想要改变的类，然后使用`transformClass`方法进行转换。

下面是我们定义的用于帮助我们转换`MyAtm`类的 `transformClass`方法的代码。

在这个方法中，我们使用`transform `方法找到我们想要转换的类。此外，我们将转换器添加到检测引擎中:

```java
private static void transformClass(
  String className, Instrumentation instrumentation) {
    Class<?> targetCls = null;
    ClassLoader targetClassLoader = null;
    // see if we can get the class using forName
    try {
        targetCls = Class.forName(className);
        targetClassLoader = targetCls.getClassLoader();
        transform(targetCls, targetClassLoader, instrumentation);
        return;
    } catch (Exception ex) {
        LOGGER.error("Class [{}] not found with Class.forName");
    }
    // otherwise iterate all loaded classes and find what we want
    for(Class<?> clazz: instrumentation.getAllLoadedClasses()) {
        if(clazz.getName().equals(className)) {
            targetCls = clazz;
            targetClassLoader = targetCls.getClassLoader();
            transform(targetCls, targetClassLoader, instrumentation);
            return;
        }
    }
    throw new RuntimeException(
      "Failed to find class [" + className + "]");
}

private static void transform(
  Class<?> clazz, 
  ClassLoader classLoader,
  Instrumentation instrumentation) {
    AtmTransformer dt = new AtmTransformer(
      clazz.getName(), classLoader);
    instrumentation.addTransformer(dt, true);
    try {
        instrumentation.retransformClasses(clazz);
    } catch (Exception ex) {
        throw new RuntimeException(
          "Transform failed for: [" + clazz.getName() + "]", ex);
    }
}
```

这样一来，让我们为`MyAtm`类定义转换器。

### 5.2。`Transformer`定义我们的

类转换器必须实现`ClassFileTransformer`并实现转换方法。

我们将使用 [Javassist](/web/20221114003540/https://www.baeldung.com/javassist) 向`MyAtm`类添加字节码，并添加一个包含 ATW 取款交易总时间的日志:

```java
public class AtmTransformer implements ClassFileTransformer {
    @Override
    public byte[] transform(
      ClassLoader loader, 
      String className, 
      Class<?> classBeingRedefined, 
      ProtectionDomain protectionDomain, 
      byte[] classfileBuffer) {
        byte[] byteCode = classfileBuffer;
        String finalTargetClassName = this.targetClassName
          .replaceAll("\\.", "/"); 
        if (!className.equals(finalTargetClassName)) {
            return byteCode;
        }

        if (className.equals(finalTargetClassName) 
              && loader.equals(targetClassLoader)) {

            LOGGER.info("[Agent] Transforming class MyAtm");
            try {
                ClassPool cp = ClassPool.getDefault();
                CtClass cc = cp.get(targetClassName);
                CtMethod m = cc.getDeclaredMethod(
                  WITHDRAW_MONEY_METHOD);
                m.addLocalVariable(
                  "startTime", CtClass.longType);
                m.insertBefore(
                  "startTime = System.currentTimeMillis();");

                StringBuilder endBlock = new StringBuilder();

                m.addLocalVariable("endTime", CtClass.longType);
                m.addLocalVariable("opTime", CtClass.longType);
                endBlock.append(
                  "endTime = System.currentTimeMillis();");
                endBlock.append(
                  "opTime = (endTime-startTime)/1000;");

                endBlock.append(
                  "LOGGER.info(\"[Application] Withdrawal operation completed in:" +
                                "\" + opTime + \" seconds!\");");

                m.insertAfter(endBlock.toString());

                byteCode = cc.toBytecode();
                cc.detach();
            } catch (NotFoundException | CannotCompileException | IOException e) {
                LOGGER.error("Exception", e);
            }
        }
        return byteCode;
    }
}
```

### 5.3.创建代理清单文件

最后，为了获得一个有效的 Java 代理，我们需要一个带有几个属性的清单文件。

因此，我们可以在[工具包](https://web.archive.org/web/20221114003540/https://docs.oracle.com/en/java/javase/11/docs/api/java.instrument/java/lang/instrument/package-summary.html)官方文档中找到清单属性的完整列表。

在最终的 Java 代理 jar 文件中，我们将在清单文件中添加以下几行:

```java
Agent-Class: com.baeldung.instrumentation.agent.MyInstrumentationAgent
Can-Redefine-Classes: true
Can-Retransform-Classes: true
Premain-Class: com.baeldung.instrumentation.agent.MyInstrumentationAgent
```

我们的 Java 插装代理现在已经完成了。要运行它，请参考本文的[加载 Java 代理](#loading-a-java-agent)一节。

## 6。结论

在本文中，我们讨论了 Java Instrumentation API。我们研究了如何静态和动态地将 Java 代理加载到 JVM 中。

我们还研究了如何从头开始创建我们自己的 Java 代理。

和往常一样，这个例子的完整实现可以在 Github 上找到[。](https://web.archive.org/web/20221114003540/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-jvm)