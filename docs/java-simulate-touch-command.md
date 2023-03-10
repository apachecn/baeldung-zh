# 在 Java 中模拟触摸命令

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-simulate-touch-command>

## 1.概观

Linux 中的 [`touch`](/web/20221013193919/https://www.baeldung.com/linux/touch-command) 命令是一种改变文件或目录的访问时间和修改时间的简便方法。它还可以用来快速创建一个空文件。

在这个简短的教程中，我们将看到如何在 Java 中模拟这个命令。

## 2.使用普通 Java

### 2.1.创建我们的`touch`方法

让我们用 Java 创建我们的`touch`方法。如果文件不存在，这个方法将创建一个空文件。它可以更改文件的访问时间和/或修改时间。

此外，它还可以使用从 input 传入的自定义时间:

```java
public static void touch(String path, String... args) throws IOException, ParseException {
    File file = new File(path);
    if (!file.exists()) {
        file.createNewFile();
        if (args.length == 0) {
            return;
        }
    }
    long timeMillis = args.length < 2 ? System.currentTimeMillis() : new SimpleDateFormat("dd-MM-yyyy hh:mm:ss").parse(args[1]).getTime();
    if (args.length > 0) {
        // change access time only
        if ("a".equals(args[0])) {
            FileTime accessFileTime = FileTime.fromMillis(timeMillis);
            Files.setAttribute(file.toPath(), "lastAccessTime", accessFileTime);
            return;
        }
        // change modification time only
        if ("m".equals(args[0])) {
            file.setLastModified(timeMillis);
            return;
        }
    }
    // other inputs will change both
    FileTime accessFileTime = FileTime.fromMillis(timeMillis);
    Files.setAttribute(file.toPath(), "lastAccessTime", accessFileTime);
    file.setLastModified(timeMillis);
}
```

从上面可以看出，我们的方法使用了 [varargs](/web/20221013193919/https://www.baeldung.com/java-varargs) 来避免重载，我们可以将一个自定义时间以“dd-MM-yyyy hh:mm:ss”格式传递给这个方法。

### 2.2.使用我们的`touch`方法

让我们用我们的方法创建一个空文件:

```java
touch("test.txt");
```

并使用 Linux 中的`stat`命令查看文件信息:

```java
stat test.txt
```

我们可以在`stat`输出中看到文件的访问和修改时间:

```java
Access: 2021-12-07 10:42:16.474007513 +0700
Modify: 2021-12-07 10:42:16.474007513 +0700
```

现在，让我们用我们的方法改变它的访问时间:

```java
touch("test.txt", "a", "16-09-2020 08:00:00");
```

然后，我们将再次使用`stat`命令获取这个文件信息:

```java
Access: 2020-09-16 08:00:00.000000000 +0700
Modify: 2021-12-07 10:42:16.474007000 +0700 
```

## 3.使用 Apache Commons Lang

我们也可以使用来自 [Apache Commons Lang](/web/20221013193919/https://www.baeldung.com/java-commons-lang-3) 库的`FileUtils`类。这个类有一个易于使用的`touch()`方法，如果文件还不存在，它也会创建一个空文件:

```java
FileUtils.touch(new File("/home/baeldung/test.txt"));
```

注意**如果文件已经存在，这个方法只会更新文件**的修改时间，而不会更新访问时间。

## 4.结论

在本文中，我们已经看到了如何在 Java 中模拟 Linux `touch`命令。

和往常一样，本文的示例代码可以在 GitHub 上找到[。](https://web.archive.org/web/20221013193919/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-io-4)