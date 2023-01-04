# 运行时扫描 Java 注释

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-scan-annotations-runtime>

## 1.介绍

正如我们所知，在 Java 世界中，注释是获取关于类和方法的元信息的一种非常有用的方式。

在本教程中，我们将讨论在运行时扫描 [Java 注释](/web/20220810172110/https://www.baeldung.com/java-custom-annotation)。

## 2.定义自定义注释

让我们首先定义一个示例注释和一个使用我们的自定义注释的示例类:

```
@Target({ METHOD, TYPE })
@Retention(RUNTIME)
public @interface SampleAnnotation {
    String name();
}

@SampleAnnotation(name = "annotatedClass")
public class SampleAnnotatedClass {

    @SampleAnnotation(name = "annotatedMethod")
    public void annotatedMethod() {
        //Do something
    }

    public void notAnnotatedMethod() {
        //Do something
    }
} 
```

现在，我们准备好**在基于类和基于方法的用法**上解析这个定制注释的“name”属性。

## 3.用 Java 反射扫描注释

借助 Java [`Reflection`](/web/20220810172110/https://www.baeldung.com/java-reflection) ，**我们可以扫描一个特定的注释类或者一个特定类的注释方法。**

为了实现这个目标，我们需要通过使用 [`ClassLoader`](/web/20220810172110/https://www.baeldung.com/java-classloaders) 来加载类。因此，当我们知道在哪个(些)类中扫描注释时，这个方法是有用的:

```
Class<?> clazz = ClassLoader.getSystemClassLoader()
  .loadClass("com.baeldung.annotation.scanner.SampleAnnotatedClass");
SampleAnnotation classAnnotation = clazz.getAnnotation(SampleAnnotation.class);
Assert.assertEquals("SampleAnnotatedClass", classAnnotation.name());
Method[] methods = clazz.getMethods();
List<String> annotatedMethods = new ArrayList<>();
for (Method method : methods) {
    SampleAnnotation annotation = method.getAnnotation(SampleAnnotation.class);
    if (annotation != null) {
        annotatedMethods.add(annotation.name());
    }
}
Assert.assertEquals(1, annotatedMethods.size());
Assert.assertEquals("annotatedMethod", annotatedMethods.get(0));
```

## 4.用 Spring 上下文库扫描注释

扫描注释的另一种方式是使用包含在 Spring 上下文库中的`ClassPathScanningCandidateComponentProvider `类。

让我们从添加`[spring-context](https://web.archive.org/web/20220810172110/https://search.maven.org/search?q=g:org.springframework%20a:spring-context)`依赖项开始:

```
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-context</artifactId>
    <version>5.3.22</version>
</dependency>
```

让我们继续看一个简单的例子:

```
ClassPathScanningCandidateComponentProvider provider =
  new ClassPathScanningCandidateComponentProvider(false);
provider.addIncludeFilter(new AnnotationTypeFilter(SampleAnnotation.class));

Set<BeanDefinition> beanDefs = provider
  .findCandidateComponents("com.baeldung.annotation.scanner");
List<String> annotatedBeans = new ArrayList<>();
for (BeanDefinition bd : beanDefs) {
    if (bd instanceof AnnotatedBeanDefinition) {
        Map<String, Object> annotAttributeMap = ((AnnotatedBeanDefinition) bd)
          .getMetadata()
          .getAnnotationAttributes(SampleAnnotation.class.getCanonicalName());
        annotatedBeans.add(annotAttributeMap.get("name").toString());
    }
}

Assert.assertEquals(1, annotatedBeans.size());
Assert.assertEquals("SampleAnnotatedClass", annotatedBeans.get(0));
```

与 Java `Reflections`、**完全不同，我们可以扫描所有的类，而不需要知道具体的类名**。

## 5.使用 Spring Core 库扫描注释

尽管 Spring Core 没有直接提供对我们代码中所有注释的全面扫描，**我们仍然可以通过使用这个库**的一些实用类来开发我们自己的全面注释扫描器。

首先，我们需要添加`[spring-core](https://web.archive.org/web/20220810172110/https://search.maven.org/search?q=g:org.springframework%20a:spring-core)`依赖关系:

```
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-core</artifactId>
    <version>5.3.22</version>
</dependency>
```

这里有一个简单的例子:

```
Class<?> userClass = ClassUtils.getUserClass(SampleAnnotatedClass.class);

List<String> annotatedMethods = Arrays.stream(userClass.getMethods())
  .filter(method -> AnnotationUtils
  .getAnnotation(method, SampleAnnotation.class) != null)
  .map(method -> method.getAnnotation(SampleAnnotation.class)
  .name())
  .collect(Collectors.toList());

Assert.assertEquals(1, annotatedMethods.size());
Assert.assertEquals("annotatedMethod", annotatedMethods.get(0));
```

在`AnnotationUtils`和`ClassUtils`的帮助下，可以找到用特定注释注释的方法和类。

## 6.使用反射库扫描注释

[`Reflections`](/web/20220810172110/https://www.baeldung.com/reflections-library) 是一个据说是按照`Scannotations`库的精神编写的库。**它扫描并索引项目的类路径元数据**。

让我们添加`[reflections](https://web.archive.org/web/20220810172110/https://search.maven.org/search?q=g:org.reflections%20a:reflections)`依赖关系:

```
<dependency>
    <groupId>org.reflections</groupId>
    <artifactId>reflections</artifactId>
    <version>0.10.2</version>
</dependency>
```

现在我们已经准备好使用这个库来搜索带注释的类、方法、字段和类型了:

```
Reflections reflections = new Reflections("com.baeldung.annotation.scanner");

Set<Method> methods = reflections
  .getMethodsAnnotatedWith(SampleAnnotation.class);
List<String> annotatedMethods = methods.stream()
  .map(method -> method.getAnnotation(SampleAnnotation.class)
  .name())
  .collect(Collectors.toList());

Assert.assertEquals(1, annotatedMethods.size());
Assert.assertEquals("annotatedMethod", annotatedMethods.get(0));

Set<Class<?>> types = reflections
  .getTypesAnnotatedWith(SampleAnnotation.class);
List<String> annotatedClasses = types.stream()
  .map(clazz -> clazz.getAnnotation(SampleAnnotation.class)
  .name())
  .collect(Collectors.toList());

Assert.assertEquals(1, annotatedClasses.size());
Assert.assertEquals("SampleAnnotatedClass", annotatedClasses.get(0));
```

正如我们所见，`Reflections`库提供了一种灵活的方式来扫描所有带注释的类和方法。所以我们不需要从`SampleAnnotatedClass`开始。

## 7.使用 Jandex 库扫描注释

现在让我们看看另一个名为`Jandex`的库，我们可以通过读取代码生成的`Jandex`文件来使用**在运行时扫描注释。**

这个库引入了一个 [`Maven`插件](https://web.archive.org/web/20220810172110/https://search.maven.org/search?q=g:org.jboss.jandex%20a:jandex-maven-plugin)来生成一个包含与我们的项目相关的元信息的`Jandex`文件:

```
<plugin>
    <groupId>org.jboss.jandex</groupId>
    <artifactId>jandex-maven-plugin</artifactId>
    <version>1.2.3</version>
    <executions>
        <execution>
            <phase>compile</phase>
            <id>make-index</id>
            <goals>
                <goal>jandex</goal>
            </goals>
            <configuration>
                <fileSets>
                    <fileSet>
                        <directory>${project.build.outputDirectory}</directory>
                    </fileSet>
                </fileSets>
            </configuration>
        </execution>
    </executions>
</plugin>
```

正如我们所看到的，在运行了`maven-install`命令之后，在`META-INF`目录下的类路径中生成了名为`jandex.idx.`的文件，如果需要的话，我们还可以使用 Maven 的 rename 插件来修改文件名。

现在我们准备扫描任何类型的注释。首先，我们需要添加`[jandex](https://web.archive.org/web/20220810172110/https://search.maven.org/search?q=g:org.jboss%20a:jandex)`依赖项:

```
<dependency>
    <groupId>org.jboss</groupId>
    <artifactId>jandex</artifactId>
    <version>2.4.3.Final</version>
</dependency>
```

现在该扫描注释了:

```
IndexReader reader = new IndexReader(appFile.getInputStream());
Index jandexFile = reader.read();
List<AnnotationInstance> appAnnotationList = jandexFile
  .getAnnotations(DotName
  .createSimple("com.baeldung.annotation.scanner.SampleAnnotation"));

List<String> annotatedMethods = new ArrayList<>();
List<String> annotatedClasses = new ArrayList<>();
for (AnnotationInstance annotationInstance : appAnnotationList) {
    if (annotationInstance.target().kind() == AnnotationTarget.Kind.METHOD) {
        annotatedMethods.add(annotationInstance.value("name")
          .value()
          .toString());
    }
    if (annotationInstance.target().kind() == AnnotationTarget.Kind.CLASS) {
        annotatedClasses.add(annotationInstance.value("name")
          .value()
          .toString());
    }
}

Assert.assertEquals(1, annotatedMethods.size()); 
Assert.assertEquals("annotatedMethod", annotatedMethods.get(0));
Assert.assertEquals(1, annotatedClasses.size());
Assert.assertEquals("SampleAnnotatedClass", annotatedClasses.get(0));
```

## 8.结论

取决于我们的要求；运行时扫描注释有多种方式。这些方法各有利弊。我们可以考虑我们需要什么来决定。

GitHub 上的[提供了这些例子的实现。](https://web.archive.org/web/20220810172110/https://github.com/eugenp/tutorials/tree/master/spring-boot-modules/spring-boot-libraries-2)