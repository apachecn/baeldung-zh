# 实现自定义 Lombok 注释

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/lombok-custom-annotation>

## 1.概观

在本教程中，**我们将使用 Lombok 实现一个自定义注释，以移除应用程序中实现单例时的模板。**

Lombok 是一个强大的 Java 库，旨在减少 Java 中的大量代码。如果你不熟悉，这里可以找到[对 Lombok](/web/20220928092136/https://www.baeldung.com/intro-to-project-lombok) 所有特性的介绍。

重要提示:Lombok 1.14.8 是最新的兼容版本，我们可以用它来学习本教程。从版本 1.16.0 开始，Lombok 隐藏了它的内部 API，并且不再可能以我们在这里展示的方式创建自定义注释。

## 2.作为注释处理器的 Lombok

Java 允许应用程序开发人员在编译阶段处理注释；最重要的是，基于注释生成新文件。因此，像 Hibernate 这样的库允许开发人员减少模板代码，而使用注释。

注释处理在本[教程](/web/20220928092136/https://www.baeldung.com/java-annotation-processing-builder)中有详细介绍。

同样地， **Project Lombok 也是一个注释处理器。它通过将注释委托给特定的处理程序来处理注释。**

当委托时，**它将带注释代码的编译器抽象语法树(AST)发送给处理程序。因此，它允许处理程序通过扩展 AST 来修改代码。**

## 3.实现自定义注释

### 3.1.延伸龙目岛

令人惊讶的是，Lombok 并不容易扩展和添加自定义注释。

事实上，**Lombok 的新版本使用 Shadow ClassLoader (SCL)将 Lombok 中的`.class`文件隐藏为`.scl`文件。因此，它迫使开发人员派生 Lombok 源代码并在那里实现注释。**

从积极的方面来看，**它简化了使用实用函数扩展定制处理程序和 AST 修改的过程。**

### 3.2.单例注释

通常，实现单例类需要大量代码。对于不使用依赖注入框架的应用程序，这只是样板文件。

例如，下面是实现单例类的一种方法:

```java
public class SingletonRegistry {
    private SingletonRegistry() {}

    private static class SingletonRegistryHolder {
        private static SingletonRegistry registry = new SingletonRegistry();
    }

    public static SingletonRegistry getInstance() {
        return SingletonRegistryHolder.registry;
    }

    // other methods
}
```

相比之下，如果我们实现它的注释版本，它看起来会是这样的:

```java
@Singleton
public class SingletonRegistry {}
```

还有，`Singleton`注解:

```java
@Target(ElementType.TYPE)
public @interface Singleton {}
```

这里需要强调的是，Lombok Singleton 处理程序会通过修改 AST 来生成我们上面看到的实现代码。

因为每个编译器的 AST 都不同，所以每个编译器都需要一个定制的 Lombok 处理程序。 **Lombok 允许为`javac`(由 Maven/Gradle 和 Netbeans 使用)和 Eclipse 编译器定制处理程序。**

在下面几节中，我们将为每个编译器实现我们的注释处理程序。

## 4.为`javac`实现一个处理器

### 4.1.Maven 依赖性

让我们首先提取 [Lombok](https://web.archive.org/web/20220928092136/https://search.maven.org/search?q=g:org.projectlombok%20AND%20a:lombok&core=gav) 所需的依赖关系:

```java
<dependency>
    <groupId>org.projectlombok</groupId>
    <artifactId>lombok</artifactId>
    <version>1.14.8</version>
</dependency>
```

此外，我们还需要 Java 附带的`tools.jar` 来访问和修改`javac` AST。但是，它没有 Maven 存储库。将它包含在 Maven 项目中最简单的方法是将其添加到`Profile:`

```java
<profiles>
    <profile>
        <id>default-tools.jar</id>
            <activation>
                <property>
                    <name>java.vendor</name>
                    <value>Oracle Corporation</value>
                </property>
            </activation>
            <dependencies>
                <dependency>
                    <groupId>com.sun</groupId>
                    <artifactId>tools</artifactId>
                    <version>${java.version}</version>
                    <scope>system</scope>
                    <systemPath>${java.home}/../lib/tools.jar</systemPath>
                </dependency>
            </dependencies>
    </profile>
</profiles>
```

### 4.2.延伸`JavacAnnotationHandler`

为了实现自定义的`javac`处理程序，我们需要扩展 Lombok 的`JavacAnnotationHandler:`

```java
public class SingletonJavacHandler extends JavacAnnotationHandler<Singleton> {
    public void handle(
      AnnotationValues<Singleton> annotation,
      JCTree.JCAnnotation ast,
      JavacNode annotationNode) {}
}
```

接下来，我们将实现`handle()` 方法。这里，注释 AST 由 Lombok 作为一个参数提供。

### 4.3.修改 AST

这就是事情变得棘手的地方。通常，更改现有的 AST 并不简单。

幸运的是， **Lombok 在`JavacHandlerUtil` 和 `JavacTreeMaker`中提供了许多实用函数，用于生成代码并将其注入 AST。**记住这一点，让我们使用这些函数并为我们的`SingletonRegistry:` 创建代码

```java
public void handle(
  AnnotationValues<Singleton> annotation,
  JCTree.JCAnnotation ast,
  JavacNode annotationNode) {
    Context context = annotationNode.getContext();
    Javac8BasedLombokOptions options = Javac8BasedLombokOptions
      .replaceWithDelombokOptions(context);
    options.deleteLombokAnnotations();
    JavacHandlerUtil
      .deleteAnnotationIfNeccessary(annotationNode, Singleton.class);
    JavacHandlerUtil
      .deleteImportFromCompilationUnit(annotationNode, "lombok.AccessLevel");
    JavacNode singletonClass = annotationNode.up();
    JavacTreeMaker singletonClassTreeMaker = singletonClass.getTreeMaker();
    addPrivateConstructor(singletonClass, singletonClassTreeMaker);

    JavacNode holderInnerClass = addInnerClass(singletonClass, singletonClassTreeMaker);
    addInstanceVar(singletonClass, singletonClassTreeMaker, holderInnerClass);
    addFactoryMethod(singletonClass, singletonClassTreeMaker, holderInnerClass);
}
```

需要特别指出的是，Lombok 提供的****`deleteAnnotationIfNeccessary()` 和`deleteImportFromCompilationUnit()` 方法用于移除注释和它们的任何导入。****

 **现在，让我们看看如何实现其他私有方法来生成代码。首先，我们将生成私有构造函数:

```java
private void addPrivateConstructor(
  JavacNode singletonClass,
  JavacTreeMaker singletonTM) {
    JCTree.JCModifiers modifiers = singletonTM.Modifiers(Flags.PRIVATE);
    JCTree.JCBlock block = singletonTM.Block(0L, nil());
    JCTree.JCMethodDecl constructor = singletonTM
      .MethodDef(
        modifiers,
        singletonClass.toName("<init>"),
        null, nil(), nil(), nil(), block, null);

    JavacHandlerUtil.injectMethod(singletonClass, constructor);
}
```

接下来，内部的`SingletonHolder` 类:

```java
private JavacNode addInnerClass(
  JavacNode singletonClass,
  JavacTreeMaker singletonTM) {
    JCTree.JCModifiers modifiers = singletonTM
      .Modifiers(Flags.PRIVATE | Flags.STATIC);
    String innerClassName = singletonClass.getName() + "Holder";
    JCTree.JCClassDecl innerClassDecl = singletonTM
      .ClassDef(modifiers, singletonClass.toName(innerClassName),
      nil(), null, nil(), nil());
    return JavacHandlerUtil.injectType(singletonClass, innerClassDecl);
}
```

现在，我们将在 holder 类中添加一个实例变量:

```java
private void addInstanceVar(
  JavacNode singletonClass,
  JavacTreeMaker singletonClassTM,
  JavacNode holderClass) {
    JCTree.JCModifiers fieldMod = singletonClassTM
      .Modifiers(Flags.PRIVATE | Flags.STATIC | Flags.FINAL);

    JCTree.JCClassDecl singletonClassDecl
      = (JCTree.JCClassDecl) singletonClass.get();
    JCTree.JCIdent singletonClassType
      = singletonClassTM.Ident(singletonClassDecl.name);

    JCTree.JCNewClass newKeyword = singletonClassTM
      .NewClass(null, nil(), singletonClassType, nil(), null);

    JCTree.JCVariableDecl instanceVar = singletonClassTM
      .VarDef(
        fieldMod,
        singletonClass.toName("INSTANCE"),
        singletonClassType,
        newKeyword);
    JavacHandlerUtil.injectField(holderClass, instanceVar);
}
```

最后，让我们添加一个访问 singleton 对象的工厂方法:

```java
private void addFactoryMethod(
  JavacNode singletonClass,
  JavacTreeMaker singletonClassTreeMaker,
  JavacNode holderInnerClass) {
    JCTree.JCModifiers modifiers = singletonClassTreeMaker
      .Modifiers(Flags.PUBLIC | Flags.STATIC);

    JCTree.JCClassDecl singletonClassDecl
      = (JCTree.JCClassDecl) singletonClass.get();
    JCTree.JCIdent singletonClassType
      = singletonClassTreeMaker.Ident(singletonClassDecl.name);

    JCTree.JCBlock block
      = addReturnBlock(singletonClassTreeMaker, holderInnerClass);

    JCTree.JCMethodDecl factoryMethod = singletonClassTreeMaker
      .MethodDef(
        modifiers,
        singletonClass.toName("getInstance"),
        singletonClassType, nil(), nil(), nil(), block, null);
    JavacHandlerUtil.injectMethod(singletonClass, factoryMethod);
}
```

显然，工厂方法从 holder 类返回实例变量。让我们也实现它:

```java
private JCTree.JCBlock addReturnBlock(
  JavacTreeMaker singletonClassTreeMaker,
  JavacNode holderInnerClass) {

    JCTree.JCClassDecl holderInnerClassDecl
      = (JCTree.JCClassDecl) holderInnerClass.get();
    JavacTreeMaker holderInnerClassTreeMaker
      = holderInnerClass.getTreeMaker();
    JCTree.JCIdent holderInnerClassType
      = holderInnerClassTreeMaker.Ident(holderInnerClassDecl.name);

    JCTree.JCFieldAccess instanceVarAccess = holderInnerClassTreeMaker
      .Select(holderInnerClassType, holderInnerClass.toName("INSTANCE"));
    JCTree.JCReturn returnValue = singletonClassTreeMaker
      .Return(instanceVarAccess);

    ListBuffer<JCTree.JCStatement> statements = new ListBuffer<>();
    statements.append(returnValue);

    return singletonClassTreeMaker.Block(0L, statements.toList());
}
```

结果，我们为单例类修改了 AST。

### 4.4.向 SPI 注册处理程序

到目前为止，我们只实现了一个 Lombok 处理程序来为我们的`SingletonRegistry.` 生成一个 AST，这里需要强调的是，Lombok 是作为注释处理器工作的。

通常，注释处理器是通过`META-INF/services`发现的。Lombok 也以同样的方式维护一个处理程序列表。另外，**使用一个名为 SPI 的框架来自动更新处理程序列表**。

出于我们的目的，我们将使用`[metainf-services](https://web.archive.org/web/20220928092136/https://search.maven.org/search?q=g:org.kohsuke.metainf-services%20AND%20a:metainf-services&core=gav)`:

```java
<dependency>
    <groupId>org.kohsuke.metainf-services</groupId>
    <artifactId>metainf-services</artifactId>
    <version>1.8</version>
</dependency>
```

现在，我们可以向 Lombok 注册我们的处理程序:

```java
@MetaInfServices(JavacAnnotationHandler.class)
public class SingletonJavacHandler extends JavacAnnotationHandler<Singleton> {}
```

**这将在编译时生成一个`lombok.javac.JavacAnnotationHandler` 文件。**这种行为在所有 SPI 框架中都很常见。

## 5.实现 Eclipse IDE 的处理程序

### 5.1.Maven 依赖性

类似于我们为访问`javac`的 AST 而添加的`tools.jar` ，我们将为 Eclipse IDE 添加`[eclipse jdt](https://web.archive.org/web/20220928092136/https://search.maven.org/search?q=g:org.eclipse.jdt%20AND%20a:core&core=gav)`:

```java
<dependency>
    <groupId>org.eclipse.jdt</groupId>
    <artifactId>core</artifactId>
    <version>3.3.0-v_771</version>
</dependency>
```

### 5.2.延伸`EclipseAnnotationHandler`

我们现在将为 Eclipse 处理程序扩展`EclipseAnnotationHandler` :

```java
@MetaInfServices(EclipseAnnotationHandler.class)
public class SingletonEclipseHandler
  extends EclipseAnnotationHandler<Singleton> {
    public void handle(
      AnnotationValues<Singleton> annotation,
      Annotation ast,
      EclipseNode annotationNode) {}
}
```

与 SPI 注释`MetaInfServices`一起，这个处理程序充当我们的`Singleton` 注释的处理器。因此，**每当在 Eclipse IDE 中编译一个类时，处理程序都会将带注释的类转换成单例实现。**

### 5.3.修改 AST

在 SPI 中注册了处理程序之后，我们现在可以开始编辑 Eclipse 编译器的 AST 了:

```java
public void handle(
  AnnotationValues<Singleton> annotation,
  Annotation ast,
  EclipseNode annotationNode) {
    EclipseHandlerUtil
      .unboxAndRemoveAnnotationParameter(
        ast,
        "onType",
        "@Singleton(onType=", annotationNode);
    EclipseNode singletonClass = annotationNode.up();
    TypeDeclaration singletonClassType
      = (TypeDeclaration) singletonClass.get();

    ConstructorDeclaration constructor
      = addConstructor(singletonClass, singletonClassType);

    TypeReference singletonTypeRef 
      = EclipseHandlerUtil.cloneSelfType(singletonClass, singletonClassType);

    StringBuilder sb = new StringBuilder();
    sb.append(singletonClass.getName());
    sb.append("Holder");
    String innerClassName = sb.toString();
    TypeDeclaration innerClass
      = new TypeDeclaration(singletonClassType.compilationResult);
    innerClass.modifiers = AccPrivate | AccStatic;
    innerClass.name = innerClassName.toCharArray();

    FieldDeclaration instanceVar = addInstanceVar(
      constructor,
      singletonTypeRef,
      innerClass);

    FieldDeclaration[] declarations = new FieldDeclaration[]{instanceVar};
    innerClass.fields = declarations;

    EclipseHandlerUtil.injectType(singletonClass, innerClass);

    addFactoryMethod(
      singletonClass,
      singletonClassType,
      singletonTypeRef,
      innerClass,
      instanceVar);
}
```

接下来，私有构造函数:

```java
private ConstructorDeclaration addConstructor(
  EclipseNode singletonClass,
  TypeDeclaration astNode) {
    ConstructorDeclaration constructor
      = new ConstructorDeclaration(astNode.compilationResult);
    constructor.modifiers = AccPrivate;
    constructor.selector = astNode.name;

    EclipseHandlerUtil.injectMethod(singletonClass, constructor);
    return constructor;
}
```

对于实例变量:

```java
private FieldDeclaration addInstanceVar(
  ConstructorDeclaration constructor,
  TypeReference typeReference,
  TypeDeclaration innerClass) {
    FieldDeclaration field = new FieldDeclaration();
    field.modifiers = AccPrivate | AccStatic | AccFinal;
    field.name = "INSTANCE".toCharArray();
    field.type = typeReference;

    AllocationExpression exp = new AllocationExpression();
    exp.type = typeReference;
    exp.binding = constructor.binding;

    field.initialization = exp;
    return field;
} 
```

最后，工厂方法:

```java
private void addFactoryMethod(
  EclipseNode singletonClass,
  TypeDeclaration astNode,
  TypeReference typeReference,
  TypeDeclaration innerClass,
  FieldDeclaration field) {

    MethodDeclaration factoryMethod
      = new MethodDeclaration(astNode.compilationResult);
    factoryMethod.modifiers 
      = AccStatic | ClassFileConstants.AccPublic;
    factoryMethod.returnType = typeReference;
    factoryMethod.sourceStart = astNode.sourceStart;
    factoryMethod.sourceEnd = astNode.sourceEnd;
    factoryMethod.selector = "getInstance".toCharArray();
    factoryMethod.bits = ECLIPSE_DO_NOT_TOUCH_FLAG;

    long pS = factoryMethod.sourceStart;
    long pE = factoryMethod.sourceEnd;
    long p = (long) pS << 32 | pE;

    FieldReference ref = new FieldReference(field.name, p);
    ref.receiver = new SingleNameReference(innerClass.name, p);

    ReturnStatement statement
      = new ReturnStatement(ref, astNode.sourceStart, astNode.sourceEnd);

    factoryMethod.statements = new Statement[]{statement};

    EclipseHandlerUtil.injectMethod(singletonClass, factoryMethod);
}
```

此外，我们必须将这个处理程序插入 Eclipse 启动类路径。通常，这是通过将以下参数添加到`eclipse.ini:`来完成的

```java
-Xbootclasspath/a:singleton-1.0-SNAPSHOT.jar
```

## 6.IntelliJ 中的自定义批注

一般来说，每个编译器都需要一个新的 Lombok 处理程序，比如我们之前实现的`javac`和 Eclipse 处理程序。

相反，IntelliJ 不支持 Lombok 处理程序。相反，它通过一个[插件](https://web.archive.org/web/20220928092136/https://github.com/mplushnikov/lombok-intellij-plugin)提供龙目语支持。

因此，**任何新的注释都必须被插件明确支持。这也适用于添加到 Lombok 的任何注释。**

## 7.结论

在本文中，我们使用 Lombok 处理程序实现了一个自定义注释。我们还简要地看了不同编译器中对我们的`Singleton` 注释的 AST 修改，这些编译器在各种 ide 中都可用。

完整的源代码可以在 Github 的[上找到。](https://web.archive.org/web/20220928092136/https://github.com/eugenp/tutorials/tree/master/lombok-modules/lombok-custom)**