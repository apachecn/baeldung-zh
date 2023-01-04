# 常见的 Java 异常

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-common-exceptions>

## 1。简介

本教程关注一些常见的 Java 异常。

我们将从讨论什么是异常开始。稍后，我们将详细讨论不同类型的检查和未检查异常。

## 2。例外情况

异常是指在程序执行过程中，代码序列中出现的异常情况。当程序在运行时违反某些约束时，就会出现这种异常情况。

所有异常类型都是类`Exception`的子类。这个类又被细分为检查异常和未检查异常。我们将在随后的部分中详细考虑它们。

## 3。检查异常

**检查异常是必须处理的。**它们是`Exception`类的直接子类。

关于它们的重要性有一场辩论，值得一看。

让我们详细定义一些检查过的异常。

### 3.1。`IOException`

**当任何输入/输出操作失败时，方法抛出一个`IOException`或它的直接子类。** 

这些 I/O 操作的典型用途包括:

*   使用`java.io`包处理文件系统或数据流
*   使用`java.net`包创建网络应用程序

`**FileNotFoundException**`

[`FileNotFoundException`](/web/20221101183553/https://www.baeldung.com/java-filenotfound-exception) 是使用文件系统时常见的`IOException`类型:

```java
try {
    new FileReader(new File("/invalid/file/location"));
} catch (FileNotFoundException e) {
    LOGGER.info("FileNotFoundException caught!");
}
```

`**MalformedURLException**`

当使用 URL 时，如果我们的 URL 无效，我们可能会遇到`MalformedURLException – `。

```java
try {
    new URL("malformedurl");
} catch (MalformedURLException e) {
    LOGGER.error("MalformedURLException caught!");
}
```

### 3.2。`ParseException`

Java 使用文本解析根据给定的`String.` **创建一个对象，如果解析导致错误，它抛出一个`ParseException`。**

例如，我们可以用不同的方式表示`Date`，例如`dd/mm/yyyy`或`dd,mm,yyyy,` ，但是尝试用不同的格式解析`string`:

```java
try {
    new SimpleDateFormat("MM, dd, yyyy").parse("invalid-date");
} catch (ParseException e) {
    LOGGER.error("ParseException caught!");
}
```

在这里，`String`是畸形的，并导致了一个`ParseException`。

### 3.3。`InterruptedException`

每当 Java 线程调用`join(), sleep()`或`wait()`时，它要么进入`WAITING`状态，要么进入`TIMED_WAITING`状态。

另外，一个线程可以通过调用另一个线程的`interrupt()`方法来中断另一个线程。

因此，****线程在处于`WAITING`或`TIMED_WAITING`状态时，如果另一个线程中断它，就会抛出一个`InterruptedException`。****

 **考虑以下两个线程的示例:

*   主线程启动子线程并中断它
*   子线程启动并调用`sleep()`

这种情况会导致`InterruptedException:`

```java
class ChildThread extends Thread {

    public void run() {
        try {
            Thread.sleep(1000);
        } catch (InterruptedException e) {
            LOGGER.error("InterruptedException caught!");
        }
    }
}

public class MainThread {

    public static void main(String[] args) 
      throws InterruptedException {
        ChildThread childThread = new ChildThread();
        childThread.start();
        childThread.interrupt();
    }
}
```

## 4。未检查的异常

**对于未检查的异常，编译器在编译过程中不检查。因此，该方法处理这些异常并不是强制性的。**

所有未检查的异常都扩展了类`RuntimeException.`

让我们详细讨论一些未检查的异常。

### 4.1。`NullPointerException`

**如果一个应用程序试图在它实际需要一个对象实例的地方使用`null`，这个方法将抛出一个`NullPointerException`。**

非法使用`null`导致`NullPointerException.`有不同的情况，让我们考虑其中的一些。

调用没有对象实例的类的方法:

```java
String strObj = null;
strObj.equals("Hello World"); // throws NullPointerException.
```

同样，如果一个应用程序试图访问或修改一个带有`null`引用的实例变量，我们会得到一个`NullPointerException:`

```java
Person personObj = null;
String name = personObj.personName; // Accessing the field of a null object
personObj.personName = "Jon Doe"; // Modifying the field of a null object
```

### 4.2。`ArrayIndexOutOfBoundsException`

数组以连续的方式存储其元素。因此，我们可以通过索引来访问它的元素。

然而**，如果一段代码试图访问一个数组的非法索引，相应的方法抛出一个** `**ArrayIndexOutOfBoundException.**`

让我们看几个抛出`ArrayIndexOutOfBoundException`的例子:

```java
int[] nums = new int[] {1, 2, 3};
int numFromNegativeIndex = nums[-1]; // Trying to access at negative index
int numFromGreaterIndex = nums[4];   // Trying to access at greater index
int numFromLengthIndex = nums[3];    // Trying to access at index equal to size of the array
```

### 4.3。`StringIndexOutOfBoundsException`

Java 中的`String`类提供了访问字符串中特定字符或从`String.`中截取字符数组的方法。当我们使用这些方法时，它在内部将`String`转换为字符数组。

同样，在这个数组中可能存在非法使用索引的情况。在这种情况下，`String` 类的这些方法抛出`StringIndexOutOfBoundsException`。

这个异常**表明该索引大于或等于`String.`** `StringIndexOutOfBoundsException`扩展`IndexOutOfBoundsException`的大小。

当我们试图访问索引长度等于`String's`的字符或者其他非法索引时，类`String`的方法`charAt(index)`抛出这个异常:

```java
String str = "Hello World";
char charAtNegativeIndex = str.charAt(-1); // Trying to access at negative index
char charAtLengthIndex = str.charAt(11);   // Trying to access at index equal to size of the string 
```

### 4.4。`NumberFormatException`

一个应用程序经常以数字数据结束在一个`String`中。为了将这些数据解释为数字，Java 允许将`String`转换为数字类型。包装类如`Integer, Float, etc.`包含了用于此目的的实用方法。

然而，**如果`String`在转换过程中没有合适的格式，该方法会抛出一个`NumberFormatException.`**

让我们考虑下面的片段。

这里，我们用一个字母数字数据声明一个`String`。此外，我们尝试使用`Integer`包装类的方法将数据解释为数字。

因此，这导致了`NumberFormatException:`

```java
String str = "100ABCD";
int x = Integer.parseInt(str); // Throws NumberFormatException
int y = Integer.valueOf(str); //Throws NumberFormatException
```

### 4.5。`ArithmeticException`

**当一个程序计算一个算术运算并导致一些异常情况时，它抛出`ArithmeticException`。**另外，`ArithmeticException`只适用于`int `和`long`数据类型。

例如，如果我们试图将一个整数除以零，我们得到一个`ArithmeticException`:

```java
int illegalOperation = 30/0; // Throws ArithmeticException
```

### 4.6。`ClassCastException`

Java 允许在对象之间进行[类型转换](/web/20221101183553/https://www.baeldung.com/java-type-casting),以支持继承和多态。我们可以向上投射或向下投射一个对象。

在向上造型中，我们将一个对象造型为它的超类型。在向下转换中，我们将一个对象转换为它的一个子类型。

然而，**在运行时，如果代码试图将一个对象向下转换为它不是实例的子类型，该方法抛出一个`ClassCastException`。**

运行时实例是类型转换中真正重要的东西。考虑`Animal`、`Dog, and Lion`之间的如下继承关系:

```java
class Animal {}

class Dog extends Animal {}

class Lion extends Animal {} 
```

此外，在驱动程序类中，我们将包含一个`Lion`实例的`Animal`引用转换为一个`Dog`。

然而，在运行时，JVM 注意到实例`Lion`与类`Dog`的子类型不兼容。

这导致了`ClassCastException:`

```java
Animal animal = new Lion(); // At runtime the instance is Lion
Dog tommy = (Dog) animal; // Throws ClassCastException
```

### 4.7。`IllegalArgumentException`

如果我们用一些非法或不恰当的参数调用一个方法，它会抛出一个`IllegalArgumentException`。

例如，`Thread`类的`sleep()`方法期望正的时间，而我们传递负的时间间隔作为参数。这导致了`IllegalArgumentException`:

```java
Thread.currentThread().sleep(-10000); // Throws IllegalArgumentException
```

### 4.8。`IllegalStateException`

**`IllegalStateException`表示一个方法在非法或不适当的时间被调用。**

每个 Java 对象都有一个状态(实例变量)和一些行为(方法)。因此，`IllegalStateException`意味着用当前状态变量调用这个对象的行为是非法的。

然而，对于一些不同的状态变量，它可能是合法的。

例如，我们使用迭代器来迭代一个列表。每当我们初始化一个，它在内部设置它的状态变量`lastRet`为-1。

在这个上下文中，程序试图调用列表中的`remove`方法:

```java
//Initialized with index at -1
Iterator<Integer> intListIterator = new ArrayList<>().iterator(); 

intListIterator.remove(); // IllegalStateException 
```

在内部，`remove`方法检查状态变量`lastRet`，如果它小于 0，它在这里抛出`IllegalStateException. `，变量仍然指向值-1。

结果，我们得到一个`IllegalStateException`。

## 5。结论

在本文中，我们首先讨论了什么是异常。`exception`是一个事件，它发生在程序的执行过程中，扰乱了程序指令的正常流程。

然后，我们将异常分为检查异常和未检查异常。

接下来，我们讨论了编译时或运行时可能出现的不同类型的异常。

我们可以在 GitHub 上找到这篇文章[的代码。](https://web.archive.org/web/20221101183553/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-exceptions)**