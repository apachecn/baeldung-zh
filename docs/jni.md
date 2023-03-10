# JNI 指南(Java 本地接口)

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/jni>

## 1。简介

正如我们所知，Java 的主要优势之一是它的可移植性——这意味着一旦我们编写和编译代码，这个过程的结果就是独立于平台的字节码。

简而言之，它可以在任何能够运行 Java 虚拟机的机器或设备上运行，并且会像我们预期的那样无缝地工作。

然而，有时**我们确实需要使用针对特定架构**而本地编译的代码。

需要使用本机代码可能有一些原因:

*   需要处理一些硬件
*   非常苛刻的过程的性能改进
*   我们希望重用现有的库，而不是用 Java 重写它。

为了实现这一点，JDK 在 JVM 中运行的字节码和本机代码(通常用 C 或 C++编写)之间引入了一座桥梁。

这个工具叫做 Java 本地接口。在本文中，我们将看到如何用它编写一些代码。

## 2。工作原理

### 2.1。本地方法:JVM 遇到编译的代码

Java 提供了关键字`native`,用于指示方法实现将由本地代码提供。

通常，当制作一个本地可执行程序时，我们可以选择使用静态或共享库:

*   静态库——在链接过程中，所有的库二进制文件都将包含在我们的可执行文件中。因此，我们不再需要这些库，但是它会增加我们的可执行文件的大小。
*   共享库——最终的可执行文件只有对库的引用，而没有代码本身。它要求我们运行可执行程序的环境能够访问我们的程序使用的所有库文件。

后者对 JNI 有意义，因为我们不能将字节码和本地编译的代码混合到同一个二进制文件中。

因此，我们的共享库将本地代码单独保存在它的`.so/.dll/.dylib`文件中(取决于我们使用的操作系统),而不是作为我们类的一部分。

**`native`关键字将我们的方法转换成一种抽象方法:**

```java
private native void aNativeMethod();
```

主要区别在于**不是由另一个 Java 类实现，而是在一个独立的本地共享库**中实现。

我们将构建一个表，表中的指针指向我们所有本机方法的实现，这样就可以从我们的 Java 代码中调用它们。

### 2.2。所需组件

下面是我们需要考虑的关键组件的简要描述。我们将在本文后面进一步解释它们

*   Java 代码——我们的类。它们将包括至少一个`native`方法。
*   本机代码——我们的本机方法的实际逻辑，通常用 C 或 C++编写。
*   JNI 头文件——这个 C/C++的头文件(`include/jni.h`进入 JDK 目录)包含了我们可能在本地程序中使用的 JNI 元素的所有定义。
*   C/C++编译器——我们可以在 GCC、Clang、Visual Studio 或任何其他我们喜欢的编译器之间进行选择，只要它能够为我们的平台生成本地共享库。

### 2.3。代码中的 JNI 元素(Java 和 C/C++)

Java 元素:

*   “native”关键字——正如我们已经提到的，任何标记为 native 的方法都必须在一个本地的共享库中实现。
*   `System.loadLibrary(String libname)`–一个静态方法，将共享库从文件系统加载到内存中，并使其导出的函数可用于我们的 Java 代码。

C/C++元素(其中许多是在`jni.h`中定义的)

*   jnie export——将共享库中的函数标记为可导出，这样它就会包含在函数表中，这样 JNI 就可以找到它
*   JNI call——结合`JNIEXPORT`,它确保我们的方法可用于 JNI 框架
*   JNI env——一个包含方法的结构，我们可以使用本地代码来访问 Java 元素
*   JavaVM——一种结构，让我们操纵一个正在运行的 JVM(或者甚至启动一个新的 JVM ),给它添加线程，破坏它等等…

## 3。你好世界 JNI

接下来，让我们看看 JNI 在实践中是如何工作的。

在本教程中，我们将使用 C++作为本地语言，G++作为编译器和链接器。

我们可以使用我们喜欢的任何其他编译器，但是下面是在 Ubuntu、Windows 和 MacOS 上安装 G++的方法:

*   Ubuntu Linux–在终端中运行命令`“sudo apt-get install build-essential”`
*   windows—[安装 MinGW](https://web.archive.org/web/20221104023238/http://www.mingw.org/)
*   MAC OS–在终端中运行命令`“g++”`,如果它还不存在，它将安装它。

### 3.1。创建 Java 类

让我们通过实现一个经典的“Hello World”开始创建我们的第一个 JNI 程序。

首先，我们创建下面的 Java 类，它包含将执行工作的本地方法:

```java
package com.baeldung.jni;

public class HelloWorldJNI {

    static {
        System.loadLibrary("native");
    }

    public static void main(String[] args) {
        new HelloWorldJNI().sayHello();
    }

    // Declare a native method sayHello() that receives no arguments and returns void
    private native void sayHello();
}
```

正如我们所见，**我们将共享库加载到一个静态块**中。这确保了它将在我们需要它的时候，从我们需要它的任何地方准备好。

或者，在这个简单的程序中，我们可以在调用我们的本地方法之前加载这个库，因为我们没有在其他地方使用这个本地库。

### 3.2.在 C++中实现方法

现在，我们需要在 C++中创建本地方法的实现。

在 C++中，定义和实现通常分别存储在`.h`和`.cpp`文件中。

首先，**为了创建方法的定义，我们必须使用 Java 编译器**的`-h`标志:

```java
javac -h . HelloWorldJNI.java
```

这将生成一个`com_baeldung_jni_HelloWorldJNI.h`文件，其中包含作为参数传递的类中的所有本机方法，在本例中，只有一个:

```java
JNIEXPORT void JNICALL Java_com_baeldung_jni_HelloWorldJNI_sayHello
  (JNIEnv *, jobject); 
```

正如我们看到的，函数名是使用完全限定的包、类和方法名自动生成的。

此外，我们可以注意到的有趣的事情是，我们将两个参数传递给了我们的函数；一个指向当前`JNIEnv;`的指针，也是这个方法附加到的 Java 对象，我们的`HelloWorldJNI`类的实例。

现在，我们必须创建一个新的`.cpp`文件来实现`sayHello`函数。**我们将在这里执行将“Hello World”打印到控制台的操作。**

我们将用与。h 一个包含头的函数，并添加以下代码来实现本机函数:

```java
JNIEXPORT void JNICALL Java_com_baeldung_jni_HelloWorldJNI_sayHello
  (JNIEnv* env, jobject thisObject) {
    std::cout << "Hello from C++ !!" << std::endl;
} 
```

### 3.3.编译和链接

至此，我们已经拥有了所有需要的部件，并在它们之间建立了连接。

我们需要从 C++代码构建我们的共享库并运行它！

要做到这一点，我们必须使用 G++编译器，**不要忘记包括来自我们的 Java JDK 安装**的 JNI 头文件。

Ubuntu 版本:

```java
g++ -c -fPIC -I${JAVA_HOME}/include -I${JAVA_HOME}/include/linux com_baeldung_jni_HelloWorldJNI.cpp -o com_baeldung_jni_HelloWorldJNI.o
```

Windows 版本:

```java
g++ -c -I%JAVA_HOME%\include -I%JAVA_HOME%\include\win32 com_baeldung_jni_HelloWorldJNI.cpp -o com_baeldung_jni_HelloWorldJNI.o
```

MacOS 版本；

```java
g++ -c -fPIC -I${JAVA_HOME}/include -I${JAVA_HOME}/include/darwin com_baeldung_jni_HelloWorldJNI.cpp -o com_baeldung_jni_HelloWorldJNI.o
```

一旦我们将平台的代码编译到文件`com_baeldung_jni_HelloWorldJNI.o`中，我们必须将它包含在一个新的共享库中。**无论我们决定给它起什么名字，它都是传递给方法`System.loadLibrary`的参数。**

我们将我们的命名为“native ”,我们将在运行 Java 代码时加载它。

G++链接器然后将 C++目标文件链接到我们的桥接库中。

Ubuntu 版本:

```java
g++ -shared -fPIC -o libnative.so com_baeldung_jni_HelloWorldJNI.o -lc
```

Windows 版本:

```java
g++ -shared -o native.dll com_baeldung_jni_HelloWorldJNI.o -Wl,--add-stdcall-alias
```

MacOS 版本:

```java
g++ -dynamiclib -o libnative.dylib com_baeldung_jni_HelloWorldJNI.o -lc
```

就是这样！

我们现在可以从命令行运行我们的程序。

然而，**我们需要将完整路径添加到包含我们刚刚生成的库的目录中。**这样 Java 将知道在哪里寻找我们的本地库:

```java
java -cp . -Djava.library.path=/NATIVE_SHARED_LIB_FOLDER com.baeldung.jni.HelloWorldJNI
```

控制台输出:

```java
Hello from C++ !!
```

## 4。使用高级 JNI 功能

说你好很好，但不是很有用。通常，我们希望在 Java 和 C++代码之间交换数据，并在我们的程序中管理这些数据。

### 4.1.向我们的本地方法添加参数

我们将向本地方法添加一些参数。让我们创建一个名为`ExampleParametersJNI`的新类，它有两个本地方法，使用不同类型的参数和返回:

```java
private native long sumIntegers(int first, int second);

private native String sayHelloToMe(String name, boolean isFemale);
```

然后，重复该过程创建一个新的。h 文件，就像我们之前做的那样。

现在创建相应的。cpp 文件与新 C++方法的实现:

```java
...
JNIEXPORT jlong JNICALL Java_com_baeldung_jni_ExampleParametersJNI_sumIntegers 
  (JNIEnv* env, jobject thisObject, jint first, jint second) {
    std::cout << "C++: The numbers received are : " << first << " and " << second << std::endl;
    return (long)first + (long)second;
}
JNIEXPORT jstring JNICALL Java_com_baeldung_jni_ExampleParametersJNI_sayHelloToMe 
  (JNIEnv* env, jobject thisObject, jstring name, jboolean isFemale) {
    const char* nameCharPointer = env->GetStringUTFChars(name, NULL);
    std::string title;
    if(isFemale) {
        title = "Ms. ";
    }
    else {
        title = "Mr. ";
    }

    std::string fullName = title + nameCharPointer;
    return env->NewStringUTF(fullName.c_str());
}
...
```

**我们已经使用了类型为 `JNIEnv`的指针`*env` 来访问由 JNI 环境实例提供的方法。**

在这种情况下,`JNIEnv`允许我们将 Java `Strings`传递到我们的 C++代码中并返回，而不用担心实现。

**我们可以在 [Oracle 官方文档中检查 Java 类型和 C JNI 类型的等价性。](https://web.archive.org/web/20221104023238/https://docs.oracle.com/en/java/javase/11/docs/specs/jni/types.html)**

为了测试我们的代码，我们必须重复前面的`HelloWorld`例子的所有编译步骤。

### 4.2。使用对象并从本机代码调用 Java 方法

在最后一个例子中，我们将看到如何将 Java 对象操作到本地 C++代码中。

我们将开始创建一个新的类`UserData`,用于存储一些用户信息:

```java
package com.baeldung.jni;

public class UserData {

    public String name;
    public double balance;

    public String getUserInfo() {
        return "[name]=" + name + ", [balance]=" + balance;
    }
}
```

然后，我们将创建另一个名为`ExampleObjectsJNI`的 Java 类，使用一些本地方法来管理类型为`UserData`的对象:

```java
...
public native UserData createUser(String name, double balance);

public native String printUserData(UserData user); 
```

再来一次，让我们创建`.h`头文件，然后在新的`.cpp`文件上创建本地方法的 C++实现:

```java
JNIEXPORT jobject JNICALL Java_com_baeldung_jni_ExampleObjectsJNI_createUser
  (JNIEnv *env, jobject thisObject, jstring name, jdouble balance) {

    // Create the object of the class UserData
    jclass userDataClass = env->FindClass("com/baeldung/jni/UserData");
    jobject newUserData = env->AllocObject(userDataClass);

    // Get the UserData fields to be set
    jfieldID nameField = env->GetFieldID(userDataClass , "name", "Ljava/lang/String;");
    jfieldID balanceField = env->GetFieldID(userDataClass , "balance", "D");

    env->SetObjectField(newUserData, nameField, name);
    env->SetDoubleField(newUserData, balanceField, balance);

    return newUserData;
}

JNIEXPORT jstring JNICALL Java_com_baeldung_jni_ExampleObjectsJNI_printUserData
  (JNIEnv *env, jobject thisObject, jobject userData) {

    // Find the id of the Java method to be called
    jclass userDataClass=env->GetObjectClass(userData);
    jmethodID methodId=env->GetMethodID(userDataClass, "getUserInfo", "()Ljava/lang/String;");

    jstring result = (jstring)env->CallObjectMethod(userData, methodId);
    return result;
} 
```

同样，我们使用`JNIEnv *env`指针从正在运行的 JVM 中访问所需的类、对象、字段和方法。

通常，我们只需要提供完整的类名来访问 Java 类，或者提供正确的方法名和签名来访问对象方法。

我们甚至在本地代码中创建了一个类`com.baeldung.jni.UserData`的实例。一旦我们有了实例，我们可以用类似于 Java 反射的方式操作它的所有属性和方法。

我们可以把`JNIEnv`的所有其他方法都查进[甲骨文官方文档](https://web.archive.org/web/20221104023238/https://docs.oracle.com/en/java/javase/11/docs/specs/jni/functions.html)。

## 4.使用 JNI 的缺点

JNI 桥接确实有其缺陷。

主要缺点是对底层平台的依赖；我们基本上失去了 Java 的“一次编写，随处运行”的特性。这意味着我们必须为我们想要支持的平台和架构的每个新组合构建一个新的库。想象一下，如果我们支持 Windows、Linux、Android、MacOS，这会对构建过程产生怎样的影响…

JNI 不仅给我们的程序增加了一层复杂性。它还在运行到 JVM 中的代码和我们的本地代码之间增加了一个昂贵的通信层:我们需要在一个编组/解组过程中转换 Java 和 C++之间双向交换的数据。

有时甚至没有类型之间的直接转换，所以我们必须写我们的对等物。

## 5。结论

为特定平台编译代码(通常)比运行字节码更快。

当我们需要加速一个高要求的过程时，这就很有用。此外，当我们没有其他选择时，例如当我们需要使用管理设备的库时。

然而，这是有代价的，因为我们必须为我们支持的每个不同平台维护额外的代码。

这就是为什么只有在没有 Java 替代品的情况下使用 JNI 通常是个好主意。

一如既往，这篇文章的代码可以在 GitHub 的[上找到。](https://web.archive.org/web/20221104023238/https://github.com/eugenp/tutorials/tree/master/java-native)