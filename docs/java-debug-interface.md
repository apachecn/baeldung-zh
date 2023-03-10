# Java 调试接口介绍(JDI)

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-debug-interface>

## 1.概观

我们可能想知道像 IntelliJ IDEA 和 Eclipse 这样被广泛认可的 ide 是如何实现 T2 调试功能的。这些工具非常依赖 Java 平台调试器架构(JPDA)。

在这篇介绍性文章中，我们将讨论 JPDA 下的 Java 调试接口 API (JDI)。

同时，**我们将一步一步地编写一个定制的调试器程序**，让我们自己熟悉方便的 JDI 界面。

## 2.JPDA 简介

Java 平台调试器架构(JPDA)是一组设计良好的接口和协议，用于调试 Java。

它提供了三个专门设计的接口，为桌面系统的开发环境实现定制的调试器。

首先，Java 虚拟机工具接口(JVMTI)帮助我们交互和控制在 [JVM](/web/20220714040905/https://www.baeldung.com/jvm-vs-jre-vs-jdk) 中运行的应用程序的执行。

然后是 Java 调试线协议(JDWP ),它定义了被测应用程序(被调试程序)和调试器之间使用的协议。

最后，使用 Java 调试接口(JDI)实现了调试器应用程序。

## 3.什么是`JDI`？

Java 调试接口 API 是 Java 提供的一组接口，用来实现调试器的前端。JDI 是 JPDA 的最高层。

用 JDI 构建的调试器可以调试运行在任何支持 JPDA 的 JVM 上的应用程序。同时，我们可以将其与任何调试层挂钩。

它提供了访问 VM 及其状态以及访问被调试程序变量的能力。同时，它允许设置断点，步进，观察点和处理线程。

## 4.设置

我们需要两个独立的程序——一个调试程序和一个调试器——来理解 JDI 的实现。

首先，我们将编写一个示例程序作为调试程序。

让我们用几个`String`变量和`println`语句创建一个`JDIExampleDebuggee`类:

```java
public class JDIExampleDebuggee {
    public static void main(String[] args) {
        String jpda = "Java Platform Debugger Architecture";
        System.out.println("Hi Everyone, Welcome to " + jpda); // add a break point here

        String jdi = "Java Debug Interface"; // add a break point here and also stepping in here
        String text = "Today, we'll dive into " + jdi;
        System.out.println(text);
    }
}
```

然后，我们将编写一个调试器程序。

让我们创建一个`JDIExampleDebugger`类，它具有保存调试程序(`debugClass`)和断点行号(`breakPointLines`)的属性:

```java
public class JDIExampleDebugger {
    private Class debugClass; 
    private int[] breakPointLines;

    // getters and setters
}
```

### 4.1.`LaunchingConnector`

首先，调试器需要一个连接器来建立与目标虚拟机(VM)的连接。

然后，我们需要将被调试程序设置为连接器的`main`参数。最后，连接器应该启动 VM 进行调试。

为此，JDI 提供了一个`Bootstrap`类，它给出了`LaunchingConnector`的一个实例。`LaunchingConnector`提供了[默认参数](https://web.archive.org/web/20220714040905/https://docs.oracle.com/en/java/javase/11/docs/specs/jpda/conninv.html#connectors)的映射，我们可以在其中设置`main` 参数。

因此，让我们将`connectAndLaunchVM`方法添加到`JDIDebuggerExample`类中:

```java
public VirtualMachine connectAndLaunchVM() throws Exception {

    LaunchingConnector launchingConnector = Bootstrap.virtualMachineManager()
      .defaultConnector();
    Map<String, Connector.Argument> arguments = launchingConnector.defaultArguments();
    arguments.get("main").setValue(debugClass.getName());
    return launchingConnector.launch(arguments);
}
```

现在，我们将把`main`方法添加到`JDIDebuggerExample`类中来调试`JDIExampleDebuggee:`

```java
public static void main(String[] args) throws Exception {

    JDIExampleDebugger debuggerInstance = new JDIExampleDebugger();
    debuggerInstance.setDebugClass(JDIExampleDebuggee.class);
    int[] breakPoints = {6, 9};
    debuggerInstance.setBreakPointLines(breakPoints);
    VirtualMachine vm = null;
    try {
        vm = debuggerInstance.connectAndLaunchVM();
        vm.resume();
    } catch(Exception e) {
        e.printStackTrace();
    }
}
```

让我们编译我们的两个类，`JDIExampleDebuggee` (调试对象)和`JDIExampleDebugger` (调试器):

```java
javac -g -cp "/Library/Java/JavaVirtualMachines/jdk1.8.0_131.jdk/Contents/Home/lib/tools.jar" 
com/baeldung/jdi/*.java
```

我们来详细讨论一下这里使用的`javac`命令。

**`-g`选项产生所有的调试信息**，没有这些信息，我们可能会看到`AbsentInformationException`。

**和`-cp`会在类路径中添加`tools.jar`来编译类。** ****

**所有的 JDI 图书馆都可以在 JDK 下使用。**因此，确保在编译和执行时在类路径中添加`tools.jar`。

就这样，现在我们准备好执行我们的定制调试器 `JDIExampleDebugger:`

```java
java -cp "/Library/Java/JavaVirtualMachines/jdk1.8.0_131.jdk/Contents/Home/lib/tools.jar:." 
JDIExampleDebugger
```

请注意“:”使用`tools.jar.`这将把`tools.jar`附加到当前运行时的类路径中(使用"；."在 windows 上)。

### 4.2.`Bootstrap`和`ClassPrepareRequest`

在这里执行调试器程序不会产生任何结果，因为我们还没有为调试准备好类并设置断点。

`VirtualMachine`类有`eventRequestManager`方法来创建各种请求，如`ClassPrepareRequest`、`BreakpointRequest`和 `StepEventRequest.`

所以，让我们将`enableClassPrepareRequest`方法添加到`JDIExampleDebugger`类中。

这将过滤`JDIExampleDebuggee`类并启用`ClassPrepareRequest:`

```java
public void enableClassPrepareRequest(VirtualMachine vm) {
    ClassPrepareRequest classPrepareRequest = vm.eventRequestManager().createClassPrepareRequest();
    classPrepareRequest.addClassFilter(debugClass.getName());
    classPrepareRequest.enable();
}
```

### 4.3.`ClassPrepareEvent` 和`BreakpointRequest`

一旦`JDIExampleDebuggee`类的`ClassPrepareRequest` 被启用，虚拟机的事件队列将开始拥有`ClassPrepareEvent`的实例。

使用`ClassPrepareEvent,` 我们可以获得设置断点的位置并创建一个`BreakPointRequest`。

为此，让我们将`setBreakPoints`方法添加到`JDIExampleDebugger`类中:

```java
public void setBreakPoints(VirtualMachine vm, ClassPrepareEvent event) throws AbsentInformationException {
    ClassType classType = (ClassType) event.referenceType();
    for(int lineNumber: breakPointLines) {
        Location location = classType.locationsOfLine(lineNumber).get(0);
        BreakpointRequest bpReq = vm.eventRequestManager().createBreakpointRequest(location);
        bpReq.enable();
    }
}
```

### 4.4.`BreakPointEvent` 和`StackFrame`

到目前为止，我们已经为调试准备好了类并设置了断点。现在，我们需要捕捉`BreakPointEvent`并显示变量。

JDI 提供了 `StackFrame`类，来获取被调试程序的所有可见变量的列表。

因此，让我们将`displayVariables`方法添加到`JDIExampleDebugger`类中:

```java
public void displayVariables(LocatableEvent event) throws IncompatibleThreadStateException, 
AbsentInformationException {
    StackFrame stackFrame = event.thread().frame(0);
    if(stackFrame.location().toString().contains(debugClass.getName())) {
        Map<LocalVariable, Value> visibleVariables = stackFrame
          .getValues(stackFrame.visibleVariables());
        System.out.println("Variables at " + stackFrame.location().toString() +  " > ");
        for (Map.Entry<LocalVariable, Value> entry : visibleVariables.entrySet()) {
            System.out.println(entry.getKey().name() + " = " + entry.getValue());
        }
    }
}
```

## 5.调试目标

在这一步，我们只需要更新`JDIExampleDebugger` 的`main`方法就可以开始调试了。

因此，我们将使用已经讨论过的方法，如`enableClassPrepareRequest`、`setBreakPoints`和`displayVariables:` 

```java
try {
    vm = debuggerInstance.connectAndLaunchVM();
    debuggerInstance.enableClassPrepareRequest(vm);
    EventSet eventSet = null;
    while ((eventSet = vm.eventQueue().remove()) != null) {
        for (Event event : eventSet) {
            if (event instanceof ClassPrepareEvent) {
                debuggerInstance.setBreakPoints(vm, (ClassPrepareEvent)event);
            }
            if (event instanceof BreakpointEvent) {
                debuggerInstance.displayVariables((BreakpointEvent) event);
            }
            vm.resume();
        }
    }
} catch (VMDisconnectedException e) {
    System.out.println("Virtual Machine is disconnected.");
} catch (Exception e) {
    e.printStackTrace();
}
```

现在首先，让我们用已经讨论过的`javac`命令再次编译`JDIDebuggerExample`类。

最后，我们将执行调试器程序以及所有更改，以查看输出:

```java
Variables at com.baeldung.jdi.JDIExampleDebuggee:6 > 
args = instance of java.lang.String[0] (id=93)
Variables at com.baeldung.jdi.JDIExampleDebuggee:9 > 
jpda = "Java Platform Debugger Architecture"
args = instance of java.lang.String[0] (id=93)
Virtual Machine is disconnected.
```

万岁！我们已经成功调试了`JDIExampleDebuggee`类。同时，我们显示了断点位置(第 6 行和第 9 行)的变量值。

因此，我们的定制调试器已经准备好了。

### 5.1.`StepRequest`

调试还需要单步调试代码，并在后续步骤中检查变量的状态。因此，我们将在断点处创建一个步骤请求。

在创建`StepRequest,`的实例时，我们必须提供步长的大小和深度。我们将分别定义 [`STEP_LINE`](https://web.archive.org/web/20220714040905/https://docs.oracle.com/en/java/javase/11/docs/api/jdk.jdi/com/sun/jdi/request/StepRequest.html#STEP_LINE) 和 [`STEP_OVER`](https://web.archive.org/web/20220714040905/https://docs.oracle.com/en/java/javase/11/docs/api/jdk.jdi/com/sun/jdi/request/StepRequest.html#STEP_OVER) 。

让我们编写一个方法来启用步骤请求。

为简单起见，我们将从最后一个断点(第 9 行)开始单步执行:

```java
public void enableStepRequest(VirtualMachine vm, BreakpointEvent event) {
    // enable step request for last break point
    if (event.location().toString().
        contains(debugClass.getName() + ":" + breakPointLines[breakPointLines.length-1])) {
        StepRequest stepRequest = vm.eventRequestManager()
            .createStepRequest(event.thread(), StepRequest.STEP_LINE, StepRequest.STEP_OVER);
        stepRequest.enable();    
    }
}
```

现在，我们可以更新`JDIExampleDebugger`的`main`方法，当它是一个`BreakPointEvent`时启用步骤请求:

```java
if (event instanceof BreakpointEvent) {
    debuggerInstance.enableStepRequest(vm, (BreakpointEvent)event);
}
```

### 5.2.`StepEvent`

与`BreakPointEvent`类似，我们也可以在`StepEvent`显示变量。

让我们相应地更新`main`方法:

```java
if (event instanceof StepEvent) {
    debuggerInstance.displayVariables((StepEvent) event);
}
```

最后，我们将执行调试器来查看变量的状态，同时单步执行代码:

```java
Variables at com.baeldung.jdi.JDIExampleDebuggee:6 > 
args = instance of java.lang.String[0] (id=93)
Variables at com.baeldung.jdi.JDIExampleDebuggee:9 > 
args = instance of java.lang.String[0] (id=93)
jpda = "Java Platform Debugger Architecture"
Variables at com.baeldung.jdi.JDIExampleDebuggee:10 > 
args = instance of java.lang.String[0] (id=93)
jpda = "Java Platform Debugger Architecture"
jdi = "Java Debug Interface"
Variables at com.baeldung.jdi.JDIExampleDebuggee:11 > 
args = instance of java.lang.String[0] (id=93)
jpda = "Java Platform Debugger Architecture"
jdi = "Java Debug Interface"
text = "Today, we'll dive into Java Debug Interface"
Variables at com.baeldung.jdi.JDIExampleDebuggee:12 > 
args = instance of java.lang.String[0] (id=93)
jpda = "Java Platform Debugger Architecture"
jdi = "Java Debug Interface"
text = "Today, we'll dive into Java Debug Interface"
Virtual Machine is disconnected.
```

如果我们比较输出，我们将意识到调试器从第 9 行开始，并显示所有后续步骤中的变量。

## 6.读取执行输出

我们可能会注意到`JDIExampleDebuggee`类的`println`语句不是调试器输出的一部分。

As per the JDI documentation, if we launch the VM through `LaunchingConnector,` its output and error streams must be read by the `Process` object.

因此，让我们将它添加到我们的`main`方法的`finally`子句中:

```java
finally {
    InputStreamReader reader = new InputStreamReader(vm.process().getInputStream());
    OutputStreamWriter writer = new OutputStreamWriter(System.out);
    char[] buf = new char[512];
    reader.read(buf);
    writer.write(buf);
    writer.flush();
}
```

现在，执行调试器程序还会将来自`JDIExampleDebuggee`类的`println`语句添加到调试输出中:

```java
Hi Everyone, Welcome to Java Platform Debugger Architecture
Today, we'll dive into Java Debug Interface
```

## 7.结论

在本文中，我们探索了 Java 平台调试器架构(JPDA)下可用的 Java 调试接口(JDI) API。

在这个过程中，我们利用 JDI 提供的便捷接口构建了一个定制调试器。同时，我们还为调试器增加了步进功能。

由于这只是对 JDI 的介绍，建议看看在 [JDI API](https://web.archive.org/web/20220714040905/https://github.com/openjdk-mirror/jdk7u-jdk) 下可用的其他接口的实现。

像往常一样，所有的代码实现都可以在 GitHub 上获得[。](https://web.archive.org/web/20220714040905/https://github.com/eugenp/tutorials/tree/master/java-jdi)