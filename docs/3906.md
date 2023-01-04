# 使用 Jimfs 模拟文件系统

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/jimfs-file-system-mocking>

## 1.概观

通常，当测试大量使用 I/O 操作的组件时，我们的测试会遇到几个问题，比如性能差、平台依赖性和意外状态。

在本教程中，**我们将看看如何使用内存文件系统 [Jimfs](https://web.archive.org/web/20220817220633/https://github.com/google/jimfs) 来缓解这些问题。**

## 2.Jimfs 简介

**Jimfs 是一个内存文件系统，它实现了 [Java NIO API](/web/20220817220633/https://www.baeldung.com/java-nio-2-file-api)** ，并且几乎支持它的所有特性。这特别有用，因为这意味着我们可以模拟虚拟内存文件系统，并使用现有的`java.nio`层与之交互。

正如我们将要看到的，使用模拟的文件系统而不是真实的文件系统可能是有益的，以便:

*   避免依赖于当前运行测试的文件系统
*   确保文件系统在每次测试运行时都达到预期的状态
*   帮助加快我们的测试

由于文件系统差异很大，使用 Jimfs 还可以方便地测试来自不同操作系统的文件系统。

## 3.Maven 依赖性

首先，让我们添加我们的示例需要的项目依赖项:

```
<dependency>
    <groupId>com.google.jimfs</groupId>
    <artifactId>jimfs</artifactId>
    <version>1.1</version>
</dependency>
```

jimfs 依赖项包含了我们使用模拟文件系统所需的一切。此外，我们将使用[6 月 5 日](/web/20220817220633/https://www.baeldung.com/junit-5)编写测试。

## 4.一个简单的文件库

我们将从定义一个简单的`FileRepository`类开始，它实现了一些标准的 CRUD 操作:

```
public class FileRepository {

    void create(Path path, String fileName) {
        Path filePath = path.resolve(fileName);
        try {
            Files.createFile(filePath);
        } catch (IOException ex) {
            throw new UncheckedIOException(ex);
        }
    }

    String read(Path path) {
        try {
            return new String(Files.readAllBytes(path));
        } catch (IOException ex) {
            throw new UncheckedIOException(ex);
        }
    }

    String update(Path path, String newContent) {
        try {
            Files.write(path, newContent.getBytes());
            return newContent;
        } catch (IOException ex) {
            throw new UncheckedIOException(ex);
        }
    }

    void delete(Path path) {
        try {
            Files.deleteIfExists(path);
        } catch (IOException ex) {
            throw new UncheckedIOException(ex);
        }
    }
}
```

正如我们所见，每个方法都使用了标准的`java.nio`类。

## 4.1.创建文件

在这一节中，我们将编写一个测试来测试我们存储库中的`create`方法:

```
@Test
@DisplayName("Should create a file on a file system")
void givenUnixSystem_whenCreatingFile_thenCreatedInPath() {
    FileSystem fileSystem = Jimfs.newFileSystem(Configuration.unix());
    String fileName = "newFile.txt";
    Path pathToStore = fileSystem.getPath("");

    fileRepository.create(pathToStore, fileName);

    assertTrue(Files.exists(pathToStore.resolve(fileName)));
}
```

在这个例子中，我们使用了`static`方法`Jimfs.newFileSystem()`来创建一个新的内存文件系统。**我们传递一个配置对象`Configuration.unix()`，它为 Unix 文件系统**创建一个不可变的配置。这包括重要的特定于操作系统的信息，如路径分隔符和有关符号链接的信息。

现在我们已经创建了一个文件，我们能够检查该文件是否在基于 Unix 的系统上成功创建。

## 4.2.读取文件

接下来，我们将测试读取文件内容的方法:

```
@Test
@DisplayName("Should read the content of the file")
void givenOSXSystem_whenReadingFile_thenContentIsReturned() throws Exception {
    FileSystem fileSystem = Jimfs.newFileSystem(Configuration.osX());
    Path resourceFilePath = fileSystem.getPath(RESOURCE_FILE_NAME);
    Files.copy(getResourceFilePath(), resourceFilePath);

    String content = fileRepository.read(resourceFilePath);

    assertEquals(FILE_CONTENT, content);
}
```

这一次，我们检查了是否有可能通过简单地使用不同类型的配置— `Jimfs.newFileSystem(Configuration.osX())` 在 **macOS(以前的 OSX)系统上读取文件的内容。**

## 4.3.更新文件

我们还可以使用 Jimfs 来测试更新文件内容的方法:

```
@Test
@DisplayName("Should update the content of the file")
void givenWindowsSystem_whenUpdatingFile_thenContentHasChanged() throws Exception {
    FileSystem fileSystem = Jimfs.newFileSystem(Configuration.windows());
    Path resourceFilePath = fileSystem.getPath(RESOURCE_FILE_NAME);
    Files.copy(getResourceFilePath(), resourceFilePath);
    String newContent = "I'm updating you.";

    String content = fileRepository.update(resourceFilePath, newContent);

    assertEquals(newContent, content);
    assertEquals(newContent, fileRepository.read(resourceFilePath));
}
```

同样，这次我们已经通过使用 `**Jimfs.newFileSystem(Configuration.windows())**.`检查了该方法在基于**的系统上的行为**

## 4.4.删除文件

为了结束对 CRUD 操作的测试，让我们测试删除文件的方法:

```
@Test
@DisplayName("Should delete file")
void givenCurrentSystem_whenDeletingFile_thenFileHasBeenDeleted() throws Exception {
    FileSystem fileSystem = Jimfs.newFileSystem();
    Path resourceFilePath = fileSystem.getPath(RESOURCE_FILE_NAME);
    Files.copy(getResourceFilePath(), resourceFilePath);

    fileRepository.delete(resourceFilePath);

    assertFalse(Files.exists(resourceFilePath));
}
```

与前面的例子不同，我们使用了`Jimfs.newFileSystem()` 而没有指定文件系统配置。在这种情况下，Jimfs 将使用适合当前操作系统的默认配置创建一个新的内存文件系统。

## 5.移动文件

在这一节中，我们将学习如何测试将文件从一个目录移动到另一个目录的方法。

首先，让我们使用标准的`java.nio.file.File`类来实现`move`方法:

```
void move(Path origin, Path destination) {
    try {
        Files.createDirectories(destination);
        Files.move(origin, destination, StandardCopyOption.REPLACE_EXISTING);
    } catch (IOException ex) {
        throw new UncheckedIOException(ex);
    }
}
```

我们将使用一个参数化测试来确保这个方法在几个不同的文件系统上工作:

```
private static Stream<Arguments> provideFileSystem() {
    return Stream.of(
            Arguments.of(Jimfs.newFileSystem(Configuration.unix())),
            Arguments.of(Jimfs.newFileSystem(Configuration.windows())),
            Arguments.of(Jimfs.newFileSystem(Configuration.osX())));
}

@ParameterizedTest
@DisplayName("Should move file to new destination")
@MethodSource("provideFileSystem")
void givenEachSystem_whenMovingFile_thenMovedToNewPath(FileSystem fileSystem) throws Exception {
    Path origin = fileSystem.getPath(RESOURCE_FILE_NAME);
    Files.copy(getResourceFilePath(), origin);
    Path destination = fileSystem.getPath("newDirectory", RESOURCE_FILE_NAME);

    fileManipulation.move(origin, destination);

    assertFalse(Files.exists(origin));
    assertTrue(Files.exists(destination));
}
```

正如我们所看到的，我们还能够使用 Jimfs 来测试我们可以从一个单元测试中在各种不同的文件系统上移动文件。

## 6.操作系统相关测试

为了演示使用 Jimfs 的另一个好处，让我们创建一个`FilePathReader`类。该类负责返回实际的系统路径，当然，这取决于操作系统:

```
class FilePathReader {

    String getSystemPath(Path path) {
        try {
            return path
              .toRealPath()
              .toString();
        } catch (IOException ex) {
            throw new UncheckedIOException(ex);
        }
    }
}
```

现在，让我们为这个类添加一个测试:

```
class FilePathReaderUnitTest {

    private static String DIRECTORY_NAME = "baeldung";

    private FilePathReader filePathReader = new FilePathReader();

    @Test
    @DisplayName("Should get path on windows")
    void givenWindowsSystem_shouldGetPath_thenReturnWindowsPath() throws Exception {
        FileSystem fileSystem = Jimfs.newFileSystem(Configuration.windows());
        Path path = getPathToFile(fileSystem);

        String stringPath = filePathReader.getSystemPath(path);

        assertEquals("C:\\work\\" + DIRECTORY_NAME, stringPath);
    }

    @Test
    @DisplayName("Should get path on unix")
    void givenUnixSystem_shouldGetPath_thenReturnUnixPath() throws Exception {
        FileSystem fileSystem = Jimfs.newFileSystem(Configuration.unix());
        Path path = getPathToFile(fileSystem);

        String stringPath = filePathReader.getSystemPath(path);

        assertEquals("/work/" + DIRECTORY_NAME, stringPath);
    }

    private Path getPathToFile(FileSystem fileSystem) throws Exception {
        Path path = fileSystem.getPath(DIRECTORY_NAME);
        Files.createDirectory(path);

        return path;
    }
}
```

正如我们所见，正如我们所料，Windows 的输出与 Unix 的不同。此外，**我们不需要使用两个不同的文件系统来运行这些测试——Jimfs 会自动为我们模拟它**。

值得一提的是 **Jimfs 不支持返回一个`java.io.File`** 的`toFile()`方法。这是`Path`类中唯一不被支持的方法。因此，给一只`InputStream`做手术可能比给一只`File`做手术更好。

## 7.结论

在本文中，我们从单元测试中学习了如何使用内存文件系统 Jimfs 来模拟文件系统交互。

首先，我们从定义一个包含几个 CRUD 操作的简单文件存储库开始。然后我们看到了如何使用不同的文件系统类型测试每种方法的例子。最后，我们看到了一个如何使用 Jimfs 测试依赖于操作系统的文件系统处理的例子。

和往常一样，这些例子的代码可以在 Github 的[上找到。](https://web.archive.org/web/20220817220633/https://github.com/eugenp/tutorials/tree/master/testing-modules/mocks)