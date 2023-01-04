# Java 扫描仪

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-scanner>

## 1。`Scanner`概述

在这个快速教程中，我们将演示如何使用 **Java `Scanner`** 类来读取输入，查找和跳过不同分隔符的模式。

## 2。扫描文件

首先，让我们看看如何使用`Scanner`读取文件。

在下面的例子中——我们将一个包含“`Hello world`”的文件读入令牌:

```java
@Test
public void whenReadFileWithScanner_thenCorrect() throws IOException{
    Scanner scanner = new Scanner(new File("test.txt"));

    assertTrue(scanner.hasNext());
    assertEquals("Hello", scanner.next());
    assertEquals("world", scanner.next());

    scanner.close();
}
```

注意，`next()`方法在这里返回下一个`String`标记。

另外，请注意我们在使用完扫描仪后是如何关闭的。

## 3。将`InputStream`转换为`String`

接下来，让我们看看如何使用`Scanner`将`InputStream`转换成`String`:

```java
@Test
public void whenConvertInputStreamToString_thenConverted()
  throws IOException {
    String expectedValue = "Hello world";
    FileInputStream inputStream 
      = new FileInputStream("test.txt");

    Scanner scanner = new Scanner(inputStream);
    scanner.useDelimiter("A");

    String result = scanner.next();
    assertEquals(expectedValue, result);

    scanner.close();
}
```

与前面的例子类似，我们使用了`Scanner`来标记从开始到下一个正则表达式“A”的整个流——它匹配完整的输入。

## 4。`Scanner`对`BufferedReader`对

现在，让我们讨论一下`Scanner`和`BufferedReader`的区别，我们通常使用:

*   `BufferedReader`当我们想把输入**读入行**时
*   `Scanner`将输入**读入令牌**

在下面的例子中，我们使用`BufferedReader`将文件读入行中:

```java
@Test
public void whenReadUsingBufferedReader_thenCorrect() 
  throws IOException {
    String firstLine = "Hello world";
    String secondLine = "Hi, John";
    BufferedReader reader 
      = new BufferedReader(new FileReader("test.txt"));

    String result = reader.readLine();
    assertEquals(firstLine, result);

    result = reader.readLine();
    assertEquals(secondLine, result);

    reader.close();
}
```

现在，让我们使用`Scanner`将同一个文件读入令牌:

```java
@Test
public void whenReadUsingScanner_thenCorrect() 
  throws IOException {
    String firstLine = "Hello world";
    FileInputStream inputStream 
      = new FileInputStream("test.txt");
    Scanner scanner = new Scanner(inputStream);

    String result = scanner.nextLine();
    assertEquals(firstLine, result);

    scanner.useDelimiter(", ");
    assertEquals("Hi", scanner.next());
    assertEquals("John", scanner.next());

    scanner.close();
}
```

注意我们如何使用`Scanner``nextLine()`API—**来读取整行**。

## 5。使用`New Scanner(System.in)` 扫描来自控制台的输入

接下来——让我们看看如何使用`Scanner`实例从控制台读取输入:

```java
@Test
public void whenReadingInputFromConsole_thenCorrect() {
    String input = "Hello";
    InputStream stdin = System.in;
    System.setIn(new ByteArrayInputStream(input.getBytes()));

    Scanner scanner = new Scanner(System.in);

    String result = scanner.next();
    assertEquals(input, result);

    System.setIn(stdin);
    scanner.close();
}
```

注意，我们使用了`System.setIn(…)`来模拟来自控制台的一些输入。

### 5.1.  `nextLine()` API

该方法只返回当前行的字符串:

```java
scanner.nextLine();
```

这将读取当前行的内容并返回它，除了末尾的任何行分隔符——在本例中是新的行字符。

读取内容后，`Scanner`将其位置设置为下一行的开头。需要记住的重要一点是， **`nextLine() ` API 消耗行分隔符并将`Scanner`的位置移动到下一行**。

所以下一次如果我们通读`Scanner`，我们将从下一行的开头开始读。

### 5.2.  `nextInt()` API

这个方法扫描输入的下一个标记作为一个`int:`

```java
scanner.nextInt();
```

API 读取下一个可用的整数标记。

在这种情况下，如果下一个令牌是一个整数，并且在该整数之后有一个行分隔符，请始终记住 **`nextInt() `不会消耗行分隔符。相反，扫描仪的位置将是行分隔符本身**。

因此，如果我们有一系列的操作，其中第一个操作是一个`scanner.nextInt()`，然后是`scanner.nextLine()`，作为输入，如果我们提供一个整数并按下换行符，这两个操作都将被执行。

`nextInt()` API 将使用整数，`nextLine()` API 将使用行分隔符，并将`Scanner`放在下一行的开始处。

## 6。验证输入

现在，让我们看看如何使用`Scanner`来验证输入。在下面的例子中，我们使用`Scanner`方法`hasNextInt()`来检查输入是否是一个整数值:

```java
@Test
public void whenValidateInputUsingScanner_thenValidated() 
  throws IOException {
    String input = "2000";
    InputStream stdin = System.in;
    System.setIn(new ByteArrayInputStream(input.getBytes()));

    Scanner scanner = new Scanner(System.in);

    boolean isIntInput = scanner.hasNextInt();
    assertTrue(isIntInput);

    System.setIn(stdin);
    scanner.close();
}
```

## 7。扫描一张`String`

接下来，让我们看看如何使用`Scanner`扫描`String`:

```java
@Test
public void whenScanString_thenCorrect() 
  throws IOException {
    String input = "Hello 1 F 3.5";
    Scanner scanner = new Scanner(input);

    assertEquals("Hello", scanner.next());
    assertEquals(1, scanner.nextInt());
    assertEquals(15, scanner.nextInt(16));
    assertEquals(3.5, scanner.nextDouble(), 0.00000001);

    scanner.close();
}
```

注意:方法`nextInt(16)`将下一个令牌读取为十六进制整数值。

## 8。找到`Pattern`

现在，让我们看看如何使用`Scanner`找到一个`Pattern`。

在下面的例子中，我们使用`findInLine()`到**在整个输入中搜索与给定的`Pattern`** 相匹配的标记:

```java
@Test
public void whenFindPatternUsingScanner_thenFound() throws IOException {
    String expectedValue = "world";
    FileInputStream inputStream = new FileInputStream("test.txt");
    Scanner scanner = new Scanner(inputStream);

    String result = scanner.findInLine("wo..d");
    assertEquals(expectedValue, result);

    scanner.close();
}
```

我们还可以使用`findWithinHorizon()`在特定域中搜索`Pattern`，如下例所示:

```java
@Test
public void whenFindPatternInHorizon_thenFound() 
  throws IOException {
    String expectedValue = "world";
    FileInputStream inputStream = new FileInputStream("test.txt");
    Scanner scanner = new Scanner(inputStream);

    String result = scanner.findWithinHorizon("wo..d", 5);
    assertNull(result);

    result = scanner.findWithinHorizon("wo..d", 100);
    assertEquals(expectedValue, result);

    scanner.close();
}
```

注意，**搜索范围仅仅是执行搜索的字符数**。

## 9。跳过`Pattern`

接下来，让我们看看如何跳过`Scanner`中的`Pattern`。在使用`Scanner`读取输入时，我们可以跳过匹配特定模式的标记。

在下面的例子中，我们使用`Scanner`方法`skip()`跳过了“`Hello`”标记:

```java
@Test
public void whenSkipPatternUsingScanner_thenSkipped() 
  throws IOException {
    FileInputStream inputStream = new FileInputStream("test.txt");
    Scanner scanner = new Scanner(inputStream);

    scanner.skip(".e.lo");

    assertEquals("world", scanner.next());

    scanner.close();
}
```

## 10。更改`Scanner`分隔符

最后，让我们看看如何更改分隔符`Scanner`。在以下示例中，我们将默认的`Scanner`分隔符更改为“`o`”:

```java
@Test
public void whenChangeScannerDelimiter_thenChanged() 
  throws IOException {
    String expectedValue = "Hello world";
    String[] splited = expectedValue.split("o");

    FileInputStream inputStream = new FileInputStream("test.txt");
    Scanner scanner = new Scanner(inputStream);
    scanner.useDelimiter("o");

    assertEquals(splited[0], scanner.next());
    assertEquals(splited[1], scanner.next());
    assertEquals(splited[2], scanner.next());

    scanner.close();
}
```

我们也可以使用多个分隔符。在下面的示例中，我们使用逗号“`,`”和破折号“`–`”作为分隔符来扫描包含“`John,Adam-Tom`”的文件:

```java
@Test
public void whenReadWithScannerTwoDelimiters_thenCorrect() 
  throws IOException {
    Scanner scanner = new Scanner(new File("test.txt"));
    scanner.useDelimiter(",|-");

    assertEquals("John", scanner.next());
    assertEquals("Adam", scanner.next());
    assertEquals("Tom", scanner.next());

    scanner.close();
}
```

注意:**默认的`Scanner`分隔符**是空白。

## 11。结论

在本教程中，我们回顾了使用 **Java 扫描器**的多个真实例子。

我们学习了如何使用`Scanner`从文件、控制台或`String`读取输入；我们还学习了如何使用`Scanner`找到并跳过一个模式，以及如何改变`Scanner`分隔符。

这些例子的实现可以在 GitHub 的[中找到。](https://web.archive.org/web/20220625172303/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-io-apis "All Scanner examples on Github")