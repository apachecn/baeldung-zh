# 用@DependsOn 注释控制 Bean 的创建顺序

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-depends-on>

## 1.概观

默认情况下，Spring 管理 beans 的生命周期并安排它们的初始化顺序。

但是，我们仍然可以根据自己的需要对其进行定制。**我们可以选择`SmartLifeCycle`接口或者`@DependsOn`注释来管理初始化命令**。

本教程探索了`@DependsOn`注释及其在缺少 bean 或[循环依赖](/web/20220629001313/https://www.baeldung.com/circular-dependencies-in-spring)的情况下的行为。或者只需要在初始化一个 bean 之前初始化另一个 bean。

## 2.专家

首先，让我们在`pom.xml`文件中导入 spring-context 依赖关系。我们应该经常参考 Maven Central 来了解依赖关系的最新版本:

```
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-context</artifactId>
    <version>5.2.8.RELEASE</version>
</dependency>
```

## 3.`@DependsOn`

我们应该使用这个注释来指定 bean 的依赖关系。Spring 保证在尝试初始化当前 bean 之前，已定义的 bean 将被初始化。

假设我们有一个依赖于 T1 和 T2 的 T0。在这种情况下，`FileReader`和`FileWriter`应该在`FileProcessor`之前初始化。

## 4.配置

配置文件是一个纯 Java 类，带有`@Configuration`注释:

```
@Configuration
@ComponentScan("com.baeldung.dependson")
public class Config {

    @Bean
    @DependsOn({"fileReader","fileWriter"})
    public FileProcessor fileProcessor(){
        return new FileProcessor();
    }

    @Bean("fileReader")
    public FileReader fileReader() {
        return new FileReader();
    }

    @Bean("fileWriter")
    public FileWriter fileWriter() {
        return new FileWriter();
    }   
}
```

**`FileProcessor`用`@DependsOn`** 指定其从属关系。我们也可以用`@DependsOn:`来注释一个`Component`

```
@Component
@DependsOn({"filereader", "fileWriter"})
public class FileProcessor {}
```

## 5.使用

让我们创建一个类`File`。每个 beans 更新`File`中的文本。`FileReader`更新为已读。`FileWriter`将它更新为 write，`FileProcessor`将文本更新为 processed:

```
@Test
public void WhenFileProcessorIsCreated_FileTextContains_Processed() {
    FileProcessor processor = context.getBean(FileProcessor.class);
    assertTrue(processor.process().endsWith("processed"));
}
```

### 5.1.缺少依赖关系

在缺少依赖的情况下，Spring 抛出一个带有基本异常`NoSuchBeanDefinitionException`的`BeanCreationException`。在这里阅读更多关于`NoSuchBeanDefinitionException`。

例如，`dummyFileProcessor` bean 依赖于一个`dummyFileWriter` bean。由于`dummyFileWriter`不存在，它抛出`BeanCreationException:`

```
@Test(expected=NoSuchBeanDefinitionException.class)
public void whenDependentBeanNotAvailable_ThrowsNosuchBeanDefinitionException(){
    context.getBean("dummyFileProcessor");
}
```

### 5.2.循环依赖

同样，在这种情况下，它抛出`BeanCreationException` 并强调 beans 具有循环依赖:

```
@Bean("dummyFileProcessorCircular")
@DependsOn({"dummyFileReaderCircular"})
@Lazy
public FileProcessor dummyFileProcessorCircular() {
    return new FileProcessor(file);
}
```

**如果 bean 最终依赖于自身**，则会发生循环依赖，从而形成一个循环:

```
Bean1 -> Bean4 -> Bean6 -> Bean1
```

## 6.要点

最后，在使用`@DependsOn`注释时，有几点需要注意:

*   使用`@DependsOn,`时，我们必须使用组件扫描
*   如果通过 XML 声明了一个带`DependsOn`注释的类，那么`DependsOn`注释元数据将被忽略

## 7.结论

当构建具有复杂依赖需求的系统时,`@DependsOn`变得特别有用。

它促进了依赖注入，确保 Spring 在加载我们的依赖类之前已经处理了那些必需的 Beans 的所有初始化。

和往常一样，代码可以在 GitHub 上找到[。](https://web.archive.org/web/20220629001313/https://github.com/eugenp/tutorials/tree/master/spring-di)