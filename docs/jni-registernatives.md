# 如何使用 JNI 的 RegisterNatives()方法？

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/jni-registernatives>

## 1.概观

在这个简短的教程中，我们将看看 [JNI](/web/20221208143921/https://www.baeldung.com/jni) `RegisterNatives()`方法，它用于创建 Java 和 C++函数之间的映射。

首先，我们将解释 JNI `RegisterNatives()` 是如何工作的`.` ，然后，我们将展示如何在`java.lang.Object'`的`registerNatives()`方法中使用它。最后，我们将展示如何在我们自己的 Java 和 C++代码中使用该功能。

## 2.JNI `RegisterNatives`法

**JVM 有两种方法可以找到并链接[本地](/web/20221208143921/https://www.baeldung.com/java-native)方法和 Java 代码。**第一个是**以特定的方式调用一个本地函数**以便 JVM 可以找到它。另一种方法是**使用 JNI `RegisterNatives()`方法**。

顾名思义，`RegisterNatives()` 用作为参数传递的类注册本地方法。通过使用这种方法，我们可以随意命名我们的 C++函数。

事实上，`java.lang.Object'` s `registerNatives()`方法使用了第二种方法。我们来看一个来自 C 中 [OpenJDK 8](https://web.archive.org/web/20221208143921/http://hg.openjdk.java.net/jdk8/jdk8/jdk/file/687fd7c7986d/src/share/native/java/lang/Object.c) 的`java.lang.Object'` s `registerNatives()`方法实现:

```java
static JNINativeMethod methods[] = {
    {"hashCode",    "()I",                    (void *)&JVM;_IHashCode},
    {"wait",        "(J)V",                   (void *)&JVM;_MonitorWait},
    {"notify",      "()V",                    (void *)&JVM;_MonitorNotify},
    {"notifyAll",   "()V",                    (void *)&JVM;_MonitorNotifyAll},
    {"clone",       "()Ljava/lang/Object;",   (void *)&JVM;_Clone},
};

JNIEXPORT void JNICALL
Java_java_lang_Object_registerNatives(JNIEnv *env, jclass cls)
{
    (*env)->RegisterNatives(env, cls,
                            methods, sizeof(methods)/sizeof(methods[0]));
} 
```

首先， `method[]`数组被初始化以存储 Java 和 C++函数名之间的映射。然后，我们看到一个以非常特殊的方式命名的方法，`Java_java_lang_Object_registerNatives`。

通过这样做，JVM 能够将它链接到本地的`java.lang.Object'` s `registerNatives()`方法。在它内部，`RegisterNatives()`方法调用中使用了`method[]`数组。

现在，让我们看看如何在我们自己的代码中使用它。

## 3.使用`RegisterNatives` 方法

让我们从 Java 类开始:

```java
public class RegisterNativesHelloWorldJNI {

    public native void register();
    public native String sayHello();

    public static void main(String[] args) {
        RegisterNativesHelloWorldJNI helloWorldJNI = new RegisterNativesHelloWorldJNI();
        helloWorldJNI.register();
        helloWorldJNI.sayHello();
    }
} 
```

我们定义了两个本机方法，`register()`和`sayHello(). `前者将使用 R `egisterNatives()`方法注册一个自定义 C++函数，以便在调用本机`sayHello()`方法时使用。

让我们看看 Java 的`register()` 原生方法的 C++实现:

```java
static JNINativeMethod methods[] = {
  {"sayHello", "()Ljava/lang/String;", (void*) &hello; },
};

JNIEXPORT void JNICALL Java_com_baeldung_jni_RegisterNativesHelloWorldJNI_register (JNIEnv* env, jobject thsObject) {
    jclass clazz = env->FindClass("com/baeldung/jni/RegisterNativesHelloWorldJNI");

    (env)->RegisterNatives(clazz, methods, sizeof(methods)/sizeof(methods[0]));
} 
```

类似于`java.lang.Object`的例子，我们首先创建一个数组来保存 Java 和 C++方法之间的映射。

然后，我们看到一个用完全限定的`Java_com_baeldung_jni_RegisterNativesHelloWorldJNI_register`名调用的函数。不幸的是，为了让 JVM 找到它并将其与 Java 代码链接起来，必须以这种方式调用它。

该函数做两件事。首先，它找到所需的 Java 类。然后，它调用`RegisterNatives()`方法，并将类和映射数组传递给它。

现在，我们可以随意调用第二个本地方法`sayHello()`:

```java
JNIEXPORT jstring JNICALL hello (JNIEnv* env, jobject thisObject) {
    std::string hello = "Hello from registered native C++ !!";
    std::cout << hello << std::endl;
    return env->NewStringUTF(hello.c_str());
}
```

我们没有使用完全限定名，而是使用了一个更短、更有意义的名称。

最后，让我们运行来自`RegisterNativesHelloWorldJNI`类的`main()`方法:

```java
Hello from registered native C++ !!
```

## 4.结论

在本文中，我们讨论了 JNI `RegisterNatives()`法法。首先，我们解释了`java.lang.Object.registerNatives()`方法的作用。然后，我们讨论了为什么使用 JNI `RegisterNatives()`方法可能有用。最后，我们展示了如何在我们自己的 Java 和 C++代码中使用它。

和往常一样，GitHub 上的[提供了这篇文章的完整源代码。](https://web.archive.org/web/20221208143921/https://github.com/eugenp/tutorials/tree/master/java-native)