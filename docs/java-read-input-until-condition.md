# 读取用户输入，直到满足条件

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-read-input-until-condition>

## 1.概观

当我们编写 Java 应用程序来接受用户输入时，可能有两种变体:单行输入和多行输入。

在单行输入的情况下，处理起来非常简单。我们读取输入，直到看到换行。然而，我们需要以不同的方式管理多行用户输入。

在本教程中，我们将讨论如何在 Java 中处理多行用户输入。

## 2.解决问题的想法

在 Java 中，我们可以使用`[Scanner](/web/20221208143956/https://www.baeldung.com/java-scanner)`类从用户输入中读取数据。因此，从用户输入中读取数据对我们来说不是一个挑战。但是，如果我们允许用户输入多行数据，我们应该知道用户何时给出了我们应该接受的所有数据。换句话说，我们需要一个事件来知道何时应该停止读取用户输入。

一种常用的方法是**我们检查用户发送的数据。如果数据符合定义的条件，我们停止读取输入数据。**实际上，这一条件会根据要求而变化。

解决问题的一个想法是编写一个[无限循环](/web/20221208143956/https://www.baeldung.com/infinite-loops-java)来保持逐行读取用户输入。在循环中，我们检查用户发送的每一行。一旦条件满足，我们就打破了无限循环:

```java
while (true) {
    String line = ... //get one input line
    if (matchTheCondition(line)) {
        break;
    }
    ... save or use the input data ...
}
```

接下来，让我们创建一个方法来实现我们的想法。

## 3.使用无限循环解决问题

为了简单起见，**在本教程中，一旦我们的应用程序接收到字符串“`bye`”(不区分大小写)，我们就停止读取输入**。

因此，按照我们之前谈到的想法，我们可以创建一个方法来解决这个问题:

```java
public static List<String> readUserInput() {
    List<String> userData = new ArrayList<>();
    System.out.println("Please enter your data below: (send 'bye' to exit) ");
    Scanner input = new Scanner(System.in);
    while (true) {
        String line = input.nextLine();
        if ("bye".equalsIgnoreCase(line)) {
            break;
        }
        userData.add(line);
    }
    return userData;
} 
```

如上面的代码所示，`readUserInput`方法从`System.in`读取用户输入，并将数据存储在` userData List`中。

一旦我们收到来自用户的`“bye”`，我们就打破了无限的`while`循环。换句话说，我们停止读取用户输入，并返回`userData`进行进一步处理。

接下来，让我们将[中的`readUserInput`方法称为`main`方法](/web/20221208143956/https://www.baeldung.com/java-main-method):

```java
public static void main(String[] args) {
    List<String> userData = readUserInput();
    System.out.printf("User Input Data:\n%s", String.join("\n", userData));
} 
```

正如我们在`main`方法中看到的，在我们调用`readUserInput`之后，我们打印出接收到的用户输入数据。

现在，让我们启动应用程序，看看它是否如预期的那样工作。

当应用程序启动时，它等待我们输入提示:

```java
Please enter your data below: (send 'bye' to exit)
```

所以，我们发一些文字，最后发“`bye`”:

```java
Hello there,
Today is 19\. Mar. 2022.
Have a nice day!
bye
```

在我们输入“`bye`”并按下`Enter`后，应用程序输出我们收集的用户输入数据并退出:

```java
User Input Data:
Hello there,
Today is 19\. Mar. 2022.
Have a nice day!
```

正如我们所看到的，该方法如预期的那样工作。

## 4.单元测试解决方案

我们已经解决了这个问题，并进行了手动测试。但是，我们可能需要不时调整方法以适应一些新的要求。因此，如果能自动测试该方法就好了。

编写单元测试来测试`readUserInput`方法与常规测试有点不同。这是因为**当`readUserInput`方法被调用时，应用程序被阻塞并等待用户输入**。

接下来，我们先来看看测试方法，然后再来说明问题是如何解决的:

```java
@Test
public void givenDataInSystemIn_whenCallingReadUserInputMethod_thenHaveUserInputData() {
    String[] inputLines = new String[]{
        "The first line.",
        "The second line.",
        "The last line.",
        "bye",
        "anything after 'bye' will be ignored"
    };
    String[] expectedLines = Arrays.copyOf(inputLines, inputLines.length - 2);
    List<String> expected = Arrays.stream(expectedLines).collect(Collectors.toList());

    InputStream stdin = System.in;
    try {
        System.setIn(new ByteArrayInputStream(String.join("\n", inputLines).getBytes()));
        List<String> actual = UserInputHandler.readUserInput();
        assertThat(actual).isEqualTo(expected);
    } finally {
        System.setIn(stdin);
    }
} 
```

现在，让我们快速浏览一下这个方法，并理解它是如何工作的。

在最开始，我们创建了一个`String`数组`inputLines`来保存我们想要用作用户输入的行。然后，我们已经初始化了`expected` `List`，其中包含了预期的数据。

接下来，棘手的部分来了。在我们将当前的`System.in`对象备份到`stdin`变量中之后，我们通过调用`System.setIn`方法重新分配了系统标准输入。

在本例中，**我们想用`inputLines`数组来模拟用户输入**。

因此，我们[将数组转换为`InputStream`](/web/20221208143956/https://www.baeldung.com/convert-byte-array-to-input-stream) ，在本例中是一个`ByteArrayInputStream`对象，并将`InputStream`对象重新分配为系统标准输入。

然后，我们可以调用目标方法并测试结果是否如预期的那样。

最后，**我们不要忘记恢复原来的`stdin`对象作为系统标准输入**。因此，我们将`System.setIn(stdin);`放在一个`[finally](/web/20221208143956/https://www.baeldung.com/java-finally-keyword)`块中，以确保它无论如何都会被执行。

如果我们运行这个测试方法，它会通过而不需要任何人工干预。

## 5.结论

在本文中，我们探讨了如何编写一个 Java 方法来读取用户输入，直到满足某个条件。

这两项关键技术是:

*   使用标准 Java API 中的`Scanner`类读取用户输入
*   检查无限循环中的每个输入行；如果条件满足，则中断循环

此外，我们还讨论了如何编写一个测试方法来自动测试我们的解决方案。

和往常一样，本教程中使用的源代码可以从 GitHub 上的[处获得。](https://web.archive.org/web/20221208143956/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-io-4)