# Java 注释处理和创建构建器

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-annotation-processing-builder>

## 1。简介

本文是对 Java 源代码级注释处理的介绍，并提供了在编译期间使用这种技术生成额外源文件的例子。

## 2。注释处理的应用

源代码级的注释处理最早出现在 Java 5 中。对于在编译阶段生成额外的源文件来说，这是一种方便的技术。

源文件不一定是 Java 文件——您可以基于源代码中的注释生成任何类型的描述、元数据、文档、资源或任何其他类型的文件。

注释处理在许多无处不在的 Java 库中被积极地使用，例如，在 QueryDSL 和 JPA 中生成元类，在 Lombok 库中用样板代码扩充类。

需要注意的一件重要事情是**注释处理 API 的限制——它只能用于生成新文件，而不能改变现有文件**。

值得注意的例外是 [Lombok](https://web.archive.org/web/20221005021228/https://projectlombok.org/) 库，它使用注释处理作为引导机制，将自己包含到编译过程中，并通过一些内部编译器 API 修改 AST。这种技巧与注释处理的预期目的无关，因此不在本文中讨论。

## 3。注释处理 API

注释处理在多轮中完成。每一轮都从编译器在源文件中搜索注释并选择适合这些注释的注释处理器开始。每个注释处理器依次在相应的源上被调用。

如果在此过程中生成了任何文件，则以生成的文件作为输入开始另一轮。这个过程一直持续到在处理阶段没有新文件生成。

每个注释处理器依次在相应的源上被调用。如果在此过程中生成了任何文件，则以生成的文件作为输入开始另一轮。这个过程一直持续到在处理阶段没有新文件生成。

注释处理 API 位于`javax.annotation.processing`包中。您必须实现的主接口是`Processor`接口，它有一个以`AbstractProcessor`类形式的部分实现。这个类是我们将要扩展来创建我们自己的注释处理器的类。

## 4。设置项目

为了演示注释处理的可能性，我们将开发一个简单的处理器来为带注释的类生成流畅的对象构建器。

我们将把我们的项目分成两个 Maven 模块。其中一个模块`annotation-processor`将包含处理器本身和注释，另一个模块`annotation-user`将包含带注释的类。这是注释处理的典型用例。

`annotation-processor`模块的设置如下。我们将使用谷歌的[自动服务](https://web.archive.org/web/20221005021228/https://github.com/google/auto/tree/master/service)库来生成处理器元数据文件，这将在后面讨论，而`maven-compiler-plugin`针对 Java 8 源代码进行了调整。这些依赖项的版本被提取到属性部分。

最新版本的[自动服务](https://web.archive.org/web/20221005021228/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22com.google.auto.service%22%20AND%20a%3A%22auto-service%22)库和 [maven-compiler-plugin](https://web.archive.org/web/20221005021228/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22org.apache.maven.plugins%22%20AND%20a%3A%22maven-compiler-plugin%22) 可以在 Maven Central repository 中找到:

```java
<properties>
    <auto-service.version>1.0-rc2</auto-service.version>
    <maven-compiler-plugin.version>
      3.5.1
    </maven-compiler-plugin.version>
</properties>

<dependencies>

    <dependency>
        <groupId>com.google.auto.service</groupId>
        <artifactId>auto-service</artifactId>
        <version>${auto-service.version}</version>
        <scope>provided</scope>
    </dependency>

</dependencies>

<build>
    <plugins>

        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-compiler-plugin</artifactId>
            <version>${maven-compiler-plugin.version}</version>
            <configuration>
                <source>1.8</source>
                <target>1.8</target>
            </configuration>
        </plugin>

    </plugins>
</build>
```

带有注释源代码的`annotation-user` Maven 模块不需要任何特殊的调优，除了在 dependencies 部分添加对注释处理器模块的依赖:

```java
<dependency>
    <groupId>com.baeldung</groupId>
    <artifactId>annotation-processing</artifactId>
    <version>1.0.0-SNAPSHOT</version>
</dependency>
```

## 5。定义注释

假设在我们的`annotation-user`模块中有一个简单的 POJO 类，它有几个字段:

```java
public class Person {

    private int age;

    private String name;

    // getters and setters …

}
```

我们想创建一个生成器助手类来更流畅地实例化`Person`类:

```java
Person person = new PersonBuilder()
  .setAge(25)
  .setName("John")
  .build();
```

这个`PersonBuilder`类显然是一代的选择，因为它的结构完全由`Person` setter 方法定义。

让我们在`annotation-processor`模块中为 setter 方法创建一个`@BuilderProperty`注释。它将允许我们为每个带有 setter 方法注释的类生成`Builder`类:

```java
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.SOURCE)
public @interface BuilderProperty {
}
```

带有`ElementType.METHOD`参数的`@Target`注释确保这个注释只能放在一个方法上。

`SOURCE`保留策略意味着该注释仅在源代码处理期间可用，在运行时不可用。

带有用`@BuilderProperty`注释标注的属性的`Person`类将如下所示:

```java
public class Person {

    private int age;

    private String name;

    @BuilderProperty
    public void setAge(int age) {
        this.age = age;
    }

    @BuilderProperty
    public void setName(String name) {
        this.name = name;
    }

    // getters …

}
```

## 6。实施一个`Processor`

### 6.1。创建一个`AbstractProcessor`子类

我们将从扩展`annotation-processor` Maven 模块中的`AbstractProcessor`类开始。

首先，我们应该指定这个处理器能够处理的注释，以及支持的源代码版本。这可以通过实现`Processor`接口的方法`getSupportedAnnotationTypes`和`getSupportedSourceVersion`或者用`@SupportedAnnotationTypes`和`@SupportedSourceVersion`注释来注释你的类来实现。

`@AutoService`注释是`auto-service`库的一部分，允许生成处理器元数据，这将在下面的章节中解释。

```java
@SupportedAnnotationTypes(
  "com.baeldung.annotation.processor.BuilderProperty")
@SupportedSourceVersion(SourceVersion.RELEASE_8)
@AutoService(Processor.class)
public class BuilderProcessor extends AbstractProcessor {

    @Override
    public boolean process(Set<? extends TypeElement> annotations, 
      RoundEnvironment roundEnv) {
        return false;
    }
}
```

您不仅可以指定具体的注释类名，还可以指定通配符，比如用`“com.baeldung.annotation.*”`来处理`com.baeldung.annotation`包及其所有子包中的注释，甚至用`“*”`来处理所有注释。

我们必须实现的唯一一个方法是自己进行处理的`process`方法。编译器为每个包含匹配注释的源文件调用它。

注释作为第一个`Set<? extends TypeElement> annotations`参数传递，关于当前处理回合的信息作为`RoundEnviroment roundEnv`参数传递。

如果您的注释处理器已经处理了所有传递的注释，并且您不希望将它们传递给列表中的其他注释处理器，那么返回值应该是。

### 6.2。收集数据

我们的处理器还没有真正做任何有用的事情，所以让我们用代码填充它。

首先，我们需要遍历在类中找到的所有注释类型——在我们的例子中，`annotations`集合将有一个对应于`@BuilderProperty`注释的元素，即使这个注释在源文件中出现多次。

尽管如此，出于完整性考虑，最好将`process`方法实现为一个迭代周期:

```java
@Override
public boolean process(Set<? extends TypeElement> annotations, 
  RoundEnvironment roundEnv) {

    for (TypeElement annotation : annotations) {
        Set<? extends Element> annotatedElements 
          = roundEnv.getElementsAnnotatedWith(annotation);

        // …
    }

    return true;
}
```

在这段代码中，我们使用`RoundEnvironment`实例接收用`@BuilderProperty`注释标注的所有元素。对于`Person`类，这些元素对应于`setName`和`setAge`方法。

注释的用户可能会错误地注释不是真正设置者的方法。setter 方法名应该以`set`开头，该方法应该接收一个参数。所以让我们把小麦和谷壳分开。

在下面的代码中，我们使用`Collectors.partitioningBy()`收集器将带注释的方法分成两个集合:带正确注释的 setters 和其他带错误注释的方法:

```java
Map<Boolean, List<Element>> annotatedMethods = annotatedElements.stream().collect(
  Collectors.partitioningBy(element ->
    ((ExecutableType) element.asType()).getParameterTypes().size() == 1
    && element.getSimpleName().toString().startsWith("set")));

List<Element> setters = annotatedMethods.get(true);
List<Element> otherMethods = annotatedMethods.get(false);
```

这里我们使用了`Element.asType()`方法来接收`TypeMirror`类的一个实例，这给了我们一些自省类型的能力，即使我们只是在源代码处理阶段。

我们应该警告用户不正确的注释方法，所以让我们使用可从`AbstractProcessor.processingEnv`受保护字段访问的`Messager`实例。在源处理阶段，以下行将为每个错误注释的元素输出一个错误:

```java
otherMethods.forEach(element ->
  processingEnv.getMessager().printMessage(Diagnostic.Kind.ERROR,
    "@BuilderProperty must be applied to a setXxx method " 
      + "with a single argument", element));
```

当然，如果正确的 setters 集合为空，那么继续当前的类型元素集合迭代就没有意义了:

```java
if (setters.isEmpty()) {
    continue;
}
```

如果 setters 集合至少有一个元素，我们将使用它从封闭元素中获得完全限定的类名，在 setter 方法的情况下，它看起来是源类本身:

```java
String className = ((TypeElement) setters.get(0)
  .getEnclosingElement()).getQualifiedName().toString();
```

生成构建器类所需的最后一点信息是设置器名称和它们的参数类型名称之间的映射:

```java
Map<String, String> setterMap = setters.stream().collect(Collectors.toMap(
    setter -> setter.getSimpleName().toString(),
    setter -> ((ExecutableType) setter.asType())
      .getParameterTypes().get(0).toString()
));
```

### 6.3。生成输出文件

现在我们有了生成一个构建器类所需的所有信息:源类的名称、它的所有 setter 名称以及它们的参数类型。

为了生成输出文件，我们将使用由`AbstractProcessor.processingEnv`受保护属性中的对象再次提供的`Filer`实例:

```java
JavaFileObject builderFile = processingEnv.getFiler()
  .createSourceFile(builderClassName);
try (PrintWriter out = new PrintWriter(builderFile.openWriter())) {
    // writing generated file to out …
}
```

下面提供了`writeBuilderFile`方法的完整代码。我们只需要计算包名、完全合格的构建器类名以及源类和构建器类的简单类名。代码的其余部分非常简单。

```java
private void writeBuilderFile(
  String className, Map<String, String> setterMap) 
  throws IOException {

    String packageName = null;
    int lastDot = className.lastIndexOf('.');
    if (lastDot > 0) {
        packageName = className.substring(0, lastDot);
    }

    String simpleClassName = className.substring(lastDot + 1);
    String builderClassName = className + "Builder";
    String builderSimpleClassName = builderClassName
      .substring(lastDot + 1);

    JavaFileObject builderFile = processingEnv.getFiler()
      .createSourceFile(builderClassName);

    try (PrintWriter out = new PrintWriter(builderFile.openWriter())) {

        if (packageName != null) {
            out.print("package ");
            out.print(packageName);
            out.println(";");
            out.println();
        }

        out.print("public class ");
        out.print(builderSimpleClassName);
        out.println(" {");
        out.println();

        out.print("    private ");
        out.print(simpleClassName);
        out.print(" object = new ");
        out.print(simpleClassName);
        out.println("();");
        out.println();

        out.print("    public ");
        out.print(simpleClassName);
        out.println(" build() {");
        out.println("        return object;");
        out.println("    }");
        out.println();

        setterMap.entrySet().forEach(setter -> {
            String methodName = setter.getKey();
            String argumentType = setter.getValue();

            out.print("    public ");
            out.print(builderSimpleClassName);
            out.print(" ");
            out.print(methodName);

            out.print("(");

            out.print(argumentType);
            out.println(" value) {");
            out.print("        object.");
            out.print(methodName);
            out.println("(value);");
            out.println("        return this;");
            out.println("    }");
            out.println();
        });

        out.println("}");
    }
}
```

## 7。运行示例

要查看代码生成的效果，您应该从共同的父根编译这两个模块，或者先编译`annotation-processor`模块，然后再编译`annotation-user`模块。

生成的 `PersonBuilder`类可以在`annotation-user/target/generated-sources/annotations/com/baeldung/annotation/PersonBuilder.java`文件中找到，应该如下所示:

```java
package com.baeldung.annotation;

public class PersonBuilder {

    private Person object = new Person();

    public Person build() {
        return object;
    }

    public PersonBuilder setName(java.lang.String value) {
        object.setName(value);
        return this;
    }

    public PersonBuilder setAge(int value) {
        object.setAge(value);
        return this;
    }
}
```

## 8。注册处理器的替代方法

要在编译阶段使用注释处理器，您有几个其他选项，这取决于您的用例以及您使用的工具。

### 8.1。使用注释处理器工具

`apt`工具是用于处理源文件的特殊命令行工具。它是 Java 5 的一部分，但从 Java 7 开始，它被弃用，取而代之的是其他选项，在 Java 8 中被完全删除。本文就不讨论了。

### 8.2。使用编译器键

`-processor`编译器密钥是一个标准的 JDK 工具，可以用您自己的注释处理器来扩充编译器的源代码处理阶段。

请注意，处理器本身和注释必须已经在单独的编译中编译为类，并且存在于类路径中，因此您应该做的第一件事是:

```java
javac com/baeldung/annotation/processor/BuilderProcessor
javac com/baeldung/annotation/processor/BuilderProperty
```

然后，用`-processor`键指定刚刚编译的注释处理器类，对源代码进行实际编译:

```java
javac -processor com.baeldung.annotation.processor.MyProcessor Person.java
```

要一次指定多个注释处理器，可以用逗号分隔它们的类名，如下所示:

```java
javac -processor package1.Processor1,package2.Processor2 SourceFile.java
```

### 8.3。使用 Maven

`maven-compiler-plugin`允许指定注释处理器作为其配置的一部分。

下面是一个为编译器插件添加注释处理器的例子。您还可以使用`generatedSourcesDirectory`配置参数指定将生成的源代码放入的目录。

请注意，`BuilderProcessor`类应该已经编译好了，例如，从构建依赖关系中的另一个 jar 导入:

```java
<build>
    <plugins>

        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-compiler-plugin</artifactId>
            <version>3.5.1</version>
            <configuration>
                <source>1.8</source>
                <target>1.8</target>
                <encoding>UTF-8</encoding>
                <generatedSourcesDirectory>${project.build.directory}
                  /generated-sources/</generatedSourcesDirectory>
                <annotationProcessors>
                    <annotationProcessor>
                        com.baeldung.annotation.processor.BuilderProcessor
                    </annotationProcessor>
                </annotationProcessors>
            </configuration>
        </plugin>

    </plugins>
</build>
```

### 8.4。将处理器 Jar 添加到类路径中

不需要在编译器选项中指定注释处理器，您可以简单地将带有处理器类的特殊结构的 jar 添加到编译器的类路径中。

为了自动获取它，编译器必须知道处理器类的名称。因此，您必须在`META-INF/services/javax.annotation.processing.Processor`文件中将其指定为处理器的完全限定类名:

```java
com.baeldung.annotation.processor.BuilderProcessor
```

您还可以从这个 jar 中指定几个要自动选取的处理器，方法是用一个新行将它们分开:

```java
package1.Processor1
package2.Processor2
package3.Processor3
```

如果您使用 Maven 构建这个 jar 并尝试将这个文件直接放入`src/main/resources/META-INF/services`目录，您将会遇到以下错误:

```java
[ERROR] Bad service configuration file, or exception thrown while 
constructing Processor object: javax.annotation.processing.Processor: 
Provider com.baeldung.annotation.processor.BuilderProcessor not found
```

这是因为当`BuilderProcessor`文件尚未编译时，编译器试图在模块本身的`source-processing`阶段使用这个文件。在 Maven 构建的资源复制阶段，必须将文件放在另一个资源目录中，并复制到`META-INF/services`目录，或者(更好)在构建期间生成。

Google `auto-service`库(将在下一节讨论)允许使用一个简单的注释来生成这个文件。

### 8.5。使用谷歌图书馆

要自动生成注册文件，您可以使用 Google 的`auto-service`库中的`@AutoService`注释，如下所示:

```java
@AutoService(Processor.class)
public BuilderProcessor extends AbstractProcessor {
    // …
}
```

该注释本身由自动服务库中的注释处理器处理。这个处理器生成包含`BuilderProcessor`类名的`META-INF/services/javax.annotation.processing.Processor`文件。

## 9。结论

在本文中，我们用一个为 POJO 生成构建器类的例子演示了源代码级的注释处理。我们还提供了几种在您的项目中注册注释处理器的替代方法。

这篇文章的源代码可以在 GitHub 的[上找到。](https://web.archive.org/web/20221005021228/https://github.com/eugenp/tutorials/tree/master/annotations)