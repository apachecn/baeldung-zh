# 从 Spring 应用程序中的类路径访问文件

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-classpath-file-access>

## 1。简介

在本教程中，我们将演示使用 Spring 访问和加载类路径中的文件内容的各种方法。

## 延伸阅读:

## [资源包指南](/web/20221202171327/https://www.baeldung.com/java-resourcebundle)

It's always challenging to maintain and extend multilingual applications. This article covers how to use the ResourceBundle to cope with the varieties that come up when you need to provide the same content to different cultures.[Read more](/web/20221202171327/https://www.baeldung.com/java-resourcebundle) →

## [在 Spring 中以字符串形式加载资源](/web/20221202171327/https://www.baeldung.com/spring-load-resource-as-string)

Learn how to inject the contents of a resource file into our beans as a String, with Spring's Resource class making this very easy.[Read more](/web/20221202171327/https://www.baeldung.com/spring-load-resource-as-string) →

## 2。使用`Resource`

`Resource `接口有助于抽象对低级资源的访问。事实上，它支持以统一的方式处理各种文件资源。

让我们从查看获得一个`Resource`实例的各种方法开始。

### 2.1。手动

为了从类路径访问资源，我们可以简单地使用`ClassPathResource`:

```
public Resource loadEmployeesWithClassPathResource() {
    return new ClassPathResource("data/employees.dat");
}
```

默认情况下，`ClassPathResource`通过在线程的上下文类加载器和默认的系统类加载器之间进行选择来删除一些样板文件。

但是，我们也可以指示类加载器直接使用:

```
return new ClassPathResource("data/employees.dat", this.getClass().getClassLoader());
```

或间接通过指定的类:

```
return new ClassPathResource(
  "data/employees.dat", 
  Employee.class.getClassLoader());
```

注意，从`Resource`开始，我们可以很容易地跳到 Java 标准表示，比如`InputStream `或`File`。

这里需要注意的另一点是，上述方法只适用于绝对路径。如果我们想指定一个相对路径，我们可以传递第二个`class`参数。该路径将相对于此类:

```
new ClassPathResource("../../../data/employees.dat", Example.class).getFile();
```

上面的文件路径是相对于`Example` 类`.`的

### 2.2。使用`@Value`

我们也可以用`@Value`注入一个`Resource`:

```
@Value("classpath:data/resource-data.txt")
Resource resourceFile;
```

`@Value`也支持其他前缀，比如`file:`和`url:`。

### 2.3。使用资源加载器

如果我们想延迟加载我们的资源，我们可以使用`ResourceLoader`:

```
@Autowired
ResourceLoader resourceLoader;
```

然后我们用`getResource`检索我们的资源:

```
public Resource loadEmployeesWithResourceLoader() {
    return resourceLoader.getResource(
      "classpath:data/employees.dat");
}
```

还要注意的是，`ResourceLoader`是由所有具体的`ApplicationContext`实现的，这意味着如果`ApplicationContext `更适合我们的情况，我们也可以简单地依赖它:

```
ApplicationContext context;

public Resource loadEmployeesWithApplicationContext() {
    return context.getResource("classpath:data/employees.dat");
}
```

## 3。使用`ResourceUtils`

提醒一下，在 Spring 中还有另一种检索资源的方法，但是 [`ResourceUtils` Javadoc](https://web.archive.org/web/20221202171327/https://docs.spring.io/spring/docs/current/javadoc-api/org/springframework/util/ResourceUtils.html) 很清楚这个类主要是供内部使用的。

如果我们在代码中看到`ResourceUtils `的用法:

```
public File loadEmployeesWithSpringInternalClass() 
  throws FileNotFoundException {
    return ResourceUtils.getFile(
      "classpath:data/employees.dat");
}
```

我们应该仔细考虑基本原理，因为使用上述标准方法之一可能更好。

## 4。正在读取资源数据

一旦我们有了一个`Resource, `,我们就很容易读懂它的内容。正如我们已经讨论过的，我们可以很容易地从`Resource`获得一个`File`或一个`InputStream`引用。

假设我们在类路径`:`中有下面的文件 `data/employees.dat`

```
Joe Employee,Jan Employee,James T. Employee
```

### 4.1。`File`读书当

现在我们可以通过调用`getFile:`来读取它的内容

```
@Test
public void whenResourceAsFile_thenReadSuccessful() 
  throws IOException {

    File resource = new ClassPathResource(
      "data/employees.dat").getFile();
    String employees = new String(
      Files.readAllBytes(resource.toPath()));
    assertEquals(
      "Joe Employee,Jan Employee,James T. Employee", 
      employees);
}
```

但是，应该注意的是，这种方法**期望资源出现在文件系统中，而不是在 jar 文件中。**

### 4.2。`InputStream`读书如一个

假设我们的资源`is`在一个罐子里。

那么我们可以把一个`Resource`读成一个`InputStream`:

```
@Test
public void whenResourceAsStream_thenReadSuccessful() 
  throws IOException {
    InputStream resource = new ClassPathResource(
      "data/employees.dat").getInputStream();
    try ( BufferedReader reader = new BufferedReader(
      new InputStreamReader(resource)) ) {
        String employees = reader.lines()
          .collect(Collectors.joining("\n"));

        assertEquals("Joe Employee,Jan Employee,James T. Employee", employees);
    }
}
```

## 5。结论

在这篇简短的文章中，我们研究了几种使用 Spring 从类路径中访问和读取资源的方法。这包括在文件系统或 jar 中的急切加载和延迟加载。

和往常一样，所有这些例子都可以在 GitHub 上[找到。](https://web.archive.org/web/20221202171327/https://github.com/eugenp/tutorials/tree/master/spring-core)