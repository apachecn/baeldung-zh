# 将文件读入数组列表

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-file-to-arraylist>

## 1。概述

在本教程中，我们将讨论**将文件读入`ArrayList`T2 的不同方式。**

Java 中有很多方法[读取一个文件。一旦我们读取了一个文件，我们就可以对该文件的内容执行许多操作。](/web/20221208143830/https://www.baeldung.com/java-read-file)

其中一些操作，如排序，可能需要将文件的全部内容处理到内存中。为了执行这样的操作，我们可能需要将文件作为行或字的`Array`或`List`来读取。

## 2.使用`FileReader`

在 Java 中读取文件最基本的方式是使用`FileReader`。根据定义， **`FileReader`是一个从`File.`** 中读取字符流的便利类

有多个构造函数可用于初始化一个`FileReader:`

```
FileReader f = new FileReader(String filepath);
FileReader f = new FileReader(File f);
FileReader f = new FileReader(FileDescriptor fd);
```

所有这些构造函数都假设默认的字符编码和默认的字节缓冲区大小是合适的。

但是，如果我们想提供自定义字符编码和字节缓冲区大小，我们可以使用`InputStreamReader`或`FileInputStream`。

在下面的代码中，我们将演示如何使用`FileReader:`将文件中的行读入`ArrayList,`

```
ArrayList<String> result = new ArrayList<>();

try (FileReader f = new FileReader(filename)) {
    StringBuffer sb = new StringBuffer();
    while (f.ready()) {
        char c = (char) f.read();
        if (c == '\n') {
            result.add(sb.toString());
            sb = new StringBuffer();
        } else {
            sb.append(c);
        }
    }
    if (sb.length() > 0) {
        result.add(sb.toString());
    }
}       
return result;
```

## 3.使用`BufferedReader`

虽然`FileReader`很容易使用，但在读取文件时，最好总是用`BuffereReader`来包装它。

这是因为 **`BufferedReader`使用一个 char 缓冲区从字符输入流**中同时读取多个值，从而减少了底层`FileStream`调用`read()`的次数。

`BufferedReader`的构造函数将`Reader`作为输入。此外，我们还可以在构造函数中提供缓冲区大小，但是，对于大多数用例，默认大小已经足够大了:

```
BufferedReader br = new BufferedReader(new FileReader(filename));
BufferedReader br = new BufferedReader(new FileReader(filename), size);
```

除了从`Reader`类继承的方法之外，`BufferedReader also `还提供了`readLine()`方法，将一整行作为`String:`来读取

```
ArrayList<String> result = new ArrayList<>();

try (BufferedReader br = new BufferedReader(new FileReader(filename))) {
    while (br.ready()) {
        result.add(br.readLine());
    }
} 
```

## 4.使用`Scanner`

另一种常见的读取文件的方式是通过`Scanner`。

**`Scanner`是一个简单的文本扫描器，用于解析原语类型和字符串，使用正则表达式。**

读取文件时，使用`File`或`FileReader`对象初始化`Scanner`:

```
Scanner s = new Scanner(new File(filename));
Scanner s = new Scanner(new FileReader(filename));
```

类似于`BufferedReader, Scanner` 提供了 `readLine()`方法来读取整行`.` 另外`, `还提供了`hasNext()`方法来指示是否有更多的值可供读取:

```
ArrayList<String> result = new ArrayList<>();

try (Scanner s = new Scanner(new FileReader(filename))) {
    while (s.hasNext()) {
        result.add(s.nextLine());
    }
    return result;
}
```

`Scanner`使用一个分隔符将其输入分成多个记号，默认分隔符是空白。通过使用各种可用的`next ` ( `nextInt`、`nextLong`等)方法，可以将这些令牌转换为不同类型的值:

```
ArrayList<Integer> result = new ArrayList<>();

try (Scanner s = new Scanner(new FileReader(filename))) {
    while (s.hasNext()) {
        result.add(s.nextInt());
    }
    return result;
}
```

## 5.使用`Files.readAllLines`

读取文件并将其所有行解析成一个`ArrayList`的最简单的方法可能是使用`Files`:中可用的 **`readAllLines() `方法**

```
List<String> result = Files.readAllLines(Paths.get(filename));
```

这个方法也可以接受一个 charset 参数，按照特定的字符编码读取:

```
Charset charset = Charset.forName("ISO-8859-1");
List<String> result = Files.readAllLines(Paths.get(filename), charset);
```

## 6.结论

总而言之，我们讨论了将`File`的内容读入`ArrayList`的一些常见方法。此外，我们还讨论了各种方法的优缺点。

例如，我们可以使用`BufferedReader`来缓冲字符以提高效率。或者，我们可以使用`Scanner`来读取使用分隔符的原语。或者，我们可以简单地使用`Files.readAllLines(),` ,而不用担心底层的实现。

像往常一样，代码可以在我们的 [GitHub 库](https://web.archive.org/web/20221208143830/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-io)中获得。