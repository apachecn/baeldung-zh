# 用 Java 读写用户输入

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-console-input-output>

## 1。简介

在这个快速教程中，我们将**演示在 Java** 中使用控制台进行用户输入和输出的几种方法。

我们将看看 [`Scanner`类](/web/20221022005027/https://www.baeldung.com/java-scanner)处理输入的一些方法，然后我们将展示一些使用`System.out`的简单输出。

最后，我们将看到如何使用从 Java 6 开始就可用的`Console`类进行控制台输入和输出。

## 2。`System.in`读自

对于我们的第一个例子，我们将使用`java.util `包中的`Scanner `类从`System.in` 获得输入——“标准”输入流:

```java
Scanner scanner = new Scanner(System.in);
```

让**使用`nextLine()`方法读取一整行输入作为`String`** 并前进到下一行:

```java
String nameSurname = scanner.nextLine();
```

我们还可以使用`next() `方法从流中获取下一个输入令牌:

```java
String gender = scanner.next();
```

如果我们期待数字输入，我们可以**使用`nextInt()`获得下一个输入作为一个`int`** 原语，同样，我们可以**使用`nextDouble()`获得一个`double`** 类型的变量:

```java
int age = scanner.nextInt();
double height = scanner.nextDouble();
```

`Scanner `类还提供了 **`hasNext_Prefix() `方法，如果下一个令牌可以被解释为相应的数据类型**，这些方法将返回`true`。

例如，我们可以使用`hasNextInt()` 方法来检查下一个令牌是否可以解释为整数:

```java
while (scanner.hasNextInt()) {
    int nmbr = scanner.nextInt();
    //...
}
```

此外，我们可以使用` hasNext(Pattern pattern) `方法来**检查以下输入令牌是否匹配模式**:

```java
if (scanner.hasNext(Pattern.compile("www.baeldung.com"))) {         
    //...
}
```

除了使用`Scanner`、**类，我们还可以使用带有`System.in `的`Input`、`StreamReader`从控制台**获取输入:

```java
BufferedReader buffReader = new BufferedReader(new InputStreamReader(System.in));
```

然后我们可以读取输入并将其解析为一个整数:

```java
int i = Integer.parseInt(buffReader.readLine()); 
```

## 3.写至`System.out`

对于控制台输出，我们可以使用**`System.out`——`PrintStream`类的一个实例，**是`OutputStream`的一种。

在我们的示例中，我们将使用控制台输出来提示用户输入，并向用户显示最终消息。

让**使用`println() `方法打印一个`String`并终止行**:

```java
System.out.println("Please enter your name and surname: ");
```

或者，我们可以**使用`print() `方法，其工作方式类似于`println()`，但是不终止线路**:

```java
System.out.print("Have a good");
System.out.print(" one!");
```

## 4。使用`Console`类进行输入和输出

在 JDK 6 和更高版本中，我们可以使用`java.io `包中的`Console `类来读写控制台。

要获取`Console`对象，我们将调用`System.console()`:

```java
Console console = System.console();
```

接下来，让我们使用`Console `类的`readLine() `方法向**控制台写入一行，然后从控制台读取一行**:

```java
String progLanguauge = console.readLine("Enter your favourite programming language: "); 
```

如果我们需要读取敏感信息，比如密码，我们可以使用`readPassword()`方法来**提示用户输入密码，并在禁用回显的情况下从控制台读取密码**:

```java
char[] pass = console.readPassword("To finish, enter password: "); 
```

我们还可以**使用`Console`类将输出写到控制台，例如，使用带有`String`参数的`printf() `方法**:

```java
console.printf(progLanguauge + " is very interesting!"); 
```

## 5。结论

在本文中，我们展示了如何使用几个 Java 类来执行控制台用户输入和输出。

和往常一样，GitHub 上的[提供了本教程的代码示例。](https://web.archive.org/web/20221022005027/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-console)