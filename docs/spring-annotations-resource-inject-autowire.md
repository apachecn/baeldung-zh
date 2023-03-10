# Spring 中的连线:@Autowired、@Resource 和@Inject

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-annotations-resource-inject-autowire>

## 1。概述

在这个 Spring 框架教程中，我们将演示如何使用与依赖注入相关的注释，即`@Resource`、`@Inject`和`@Autowired`注释。这些注释为类提供了一种解决依赖关系的声明性方法:

```java
@Autowired 
ArbitraryClass arbObject;
```

与直接实例化它们相反(命令式方式):

```java
ArbitraryClass arbObject = new ArbitraryClass();
```

三个注释中的两个属于 Java 扩展包:`javax.annotation.Resource`和`javax.inject.Inject`。`@Autowired`注释属于`org.springframework.beans.factory.annotation`包。

这些注释中的每一个都可以通过字段注入或 setter 注入来解析依赖关系。我们将使用一个简单但实用的示例，根据每个注释所采用的执行路径来演示这三个注释之间的区别。

示例将集中在如何在集成测试期间使用三个注入注释。测试所需的依赖项可以是任意文件，也可以是任意类。

## 延伸阅读:

## [Spring 中的构造函数依赖注入](/web/20220720170719/https://www.baeldung.com/constructor-injection-in-spring)

Quick and practical intro to Constructor based injection with Spring.[Read more](/web/20220720170719/https://www.baeldung.com/constructor-injection-in-spring) →

## [Spring 控制反转和依赖注入简介](/web/20220720170719/https://www.baeldung.com/inversion-control-and-dependency-injection-in-spring)

A quick introduction to the concepts of Inversion of Control and Dependency Injection, followed by a simple demonstration using the Spring Framework[Read more](/web/20220720170719/https://www.baeldung.com/inversion-control-and-dependency-injection-in-spring) →

## [在抽象类中使用@ Autowired](/web/20220720170719/https://www.baeldung.com/spring-autowired-abstract-class)

Learn the differences of using @Autowired on abstract classes vs. concrete classes[Read more](/web/20220720170719/https://www.baeldung.com/spring-autowired-abstract-class) →

## 2。`@Resource`一个一个**注解**

`@Resource`注释是 [JSR-250](https://web.archive.org/web/20220720170719/https://jcp.org/en/jsr/detail?id=250) 注释集合的一部分，与 Jakarta EE 打包在一起。此注释具有以下执行路径，按优先顺序列出:

1.  按名称匹配
2.  按类型匹配
3.  按限定符匹配

这些执行路径适用于 setter 和字段注入。

### 2.1。现场注射

我们可以通过用`@Resource`注释来注释实例变量，从而通过字段注入来解决依赖性。

2.1.1。按名称匹配

我们将使用下面的集成测试来演示按名称匹配字段注入:

```java
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(
  loader=AnnotationConfigContextLoader.class,
  classes=ApplicationContextTestResourceNameType.class)
public class FieldResourceInjectionIntegrationTest {

    @Resource(name="namedFile")
    private File defaultFile;

    @Test
    public void givenResourceAnnotation_WhenOnField_ThenDependencyValid(){
        assertNotNull(defaultFile);
        assertEquals("namedFile.txt", defaultFile.getName());
    }
}
```

让我们看一下代码。在`FieldResourceInjectionTest`集成测试中，在第 7 行，我们通过将 bean 名称作为属性值传递给`@Resource`注释来解决名称依赖性:

```java
@Resource(name="namedFile")
private File defaultFile;
```

此配置将使用按名称匹配的执行路径来解析依赖项。我们必须在`ApplicationContextTestResourceNameType`应用程序上下文中定义 bean `namedFile`。

请注意，bean id 和相应的引用属性值必须匹配:

```java
@Configuration
public class ApplicationContextTestResourceNameType {

    @Bean(name="namedFile")
    public File namedFile() {
        File namedFile = new File("namedFile.txt");
        return namedFile;
    }
}
```

如果我们未能在应用程序上下文中定义 bean，将导致抛出一个`org.springframework.beans.factory.NoSuchBeanDefinitionException`。我们可以通过更改传递到`ApplicationContextTestResourceNameType`应用程序上下文中的`@Bean`注释中的属性值，或者更改传递到`FieldResourceInjectionTest`集成测试中的`@Resource`注释中的属性值来演示这一点。

#### 2.1.2。按类型匹配

为了演示按类型匹配的执行路径，我们只是删除了第 7 行`FieldResourceInjectionTest`集成测试中的属性值:

```java
@Resource
private File defaultFile;
```

然后我们再次进行测试。

测试仍然会通过，因为如果`@Resource`注释没有接收到作为属性值的 bean 名称，Spring 框架将继续下一个优先级，按类型匹配，以便尝试解决依赖关系。

#### 2.1.3。按限定符匹配

为了演示按限定符匹配的执行路径，将修改集成测试场景，以便在`ApplicationContextTestResourceQualifier`应用程序上下文中定义两个 beans:

```java
@Configuration
public class ApplicationContextTestResourceQualifier {

    @Bean(name="defaultFile")
    public File defaultFile() {
        File defaultFile = new File("defaultFile.txt");
        return defaultFile;
    }

    @Bean(name="namedFile")
    public File namedFile() {
        File namedFile = new File("namedFile.txt");
        return namedFile;
    }
}
```

我们将使用`QualifierResourceInjectionTest` 集成测试来演示按限定符匹配的依赖解决方案。在这种情况下，需要将特定的 bean 依赖注入到每个引用变量中:

```java
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(
  loader=AnnotationConfigContextLoader.class,
  classes=ApplicationContextTestResourceQualifier.class)
public class QualifierResourceInjectionIntegrationTest {

    @Resource
    private File dependency1;

    @Resource
    private File dependency2;

    @Test
    public void givenResourceAnnotation_WhenField_ThenDependency1Valid(){
        assertNotNull(dependency1);
        assertEquals("defaultFile.txt", dependency1.getName());
    }

    @Test
    public void givenResourceQualifier_WhenField_ThenDependency2Valid(){
        assertNotNull(dependency2);
        assertEquals("namedFile.txt", dependency2.getName());
    }
}
```

当我们运行集成测试时，会抛出一个`org.springframework.beans.factory.NoUniqueBeanDefinitionException`。这将会发生，因为应用程序上下文将会找到两个类型为`File`的 bean 定义，并且不知道哪个 bean 应该解决依赖关系。

为了解决这个问题，我们需要参考`QualifierResourceInjectionTest`集成测试的第 7 行到第 10 行:

```java
@Resource
private File dependency1;

@Resource
private File dependency2;
```

我们必须添加以下几行代码:

```java
@Qualifier("defaultFile")

@Qualifier("namedFile")
```

因此代码块如下所示:

```java
@Resource
@Qualifier("defaultFile")
private File dependency1;

@Resource
@Qualifier("namedFile")
private File dependency2;
```

当我们再次运行集成测试时，它应该会通过。我们的测试表明，即使我们在一个应用程序上下文中定义了多个 beans，我们也可以使用`@Qualifier`注释通过允许我们将特定的依赖项注入到一个类中来消除任何混淆。

### 2.2。设定注射

在字段上注入依赖项时采用的执行路径也适用于基于 setter 的注入。

#### 2.2.1。按名称匹配

唯一的区别是`MethodResourceInjectionTest`集成测试有一个 setter 方法:

```java
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(
  loader=AnnotationConfigContextLoader.class,
  classes=ApplicationContextTestResourceNameType.class)
public class MethodResourceInjectionIntegrationTest {

    private File defaultFile;

    @Resource(name="namedFile")
    protected void setDefaultFile(File defaultFile) {
        this.defaultFile = defaultFile;
    }

    @Test
    public void givenResourceAnnotation_WhenSetter_ThenDependencyValid(){
        assertNotNull(defaultFile);
        assertEquals("namedFile.txt", defaultFile.getName());
    }
}
```

我们通过注释引用变量的相应 setter 方法，通过 setter 注入来解决依赖关系。然后，我们将 bean 依赖项的名称作为属性值传递给`@Resource`注释:

```java
private File defaultFile;

@Resource(name="namedFile")
protected void setDefaultFile(File defaultFile) {
    this.defaultFile = defaultFile;
}
```

在这个例子中，我们将重用`namedFile` bean 依赖。bean 名称和相应的属性值必须匹配。

当我们运行集成测试时，它会通过。

为了让我们验证按名称匹配的执行路径解决了依赖关系，我们需要将传递给`@Resource`注释的属性值更改为我们选择的值，并再次运行测试。这一次，测试将以`NoSuchBeanDefinitionException`失败。

#### 2.2.2。按类型匹配

为了演示基于 setter、按类型匹配的执行，我们将使用`MethodByTypeResourceTest`集成测试:

```java
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(
  loader=AnnotationConfigContextLoader.class,
  classes=ApplicationContextTestResourceNameType.class)
public class MethodByTypeResourceIntegrationTest {

    private File defaultFile;

    @Resource
    protected void setDefaultFile(File defaultFile) {
        this.defaultFile = defaultFile;
    }

    @Test
    public void givenResourceAnnotation_WhenSetter_ThenValidDependency(){
        assertNotNull(defaultFile);
        assertEquals("namedFile.txt", defaultFile.getName());
    }
}
```

当我们运行这个测试时，它会通过。

为了让我们验证按类型匹配的执行路径解析了`File`依赖，我们需要将`defaultFile`变量的类类型更改为另一个类类型，如`String`。然后我们可以再次执行`MethodByTypeResourceTest`集成测试，这一次将抛出一个`NoSuchBeanDefinitionException`。

该异常验证了按类型匹配确实用于解析`File`依赖关系。`NoSuchBeanDefinitionException`确认引用变量名不需要匹配 bean 名。相反，依赖关系解析依赖于 bean 的类类型与引用变量的类类型的匹配。

#### 2.2.3。按限定符匹配

我们将使用`MethodByQualifierResourceTest`集成测试来演示按限定符匹配的执行路径:

```java
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(
  loader=AnnotationConfigContextLoader.class,
  classes=ApplicationContextTestResourceQualifier.class)
public class MethodByQualifierResourceIntegrationTest {

    private File arbDependency;
    private File anotherArbDependency;

    @Test
    public void givenResourceQualifier_WhenSetter_ThenValidDependencies(){
      assertNotNull(arbDependency);
        assertEquals("namedFile.txt", arbDependency.getName());
        assertNotNull(anotherArbDependency);
        assertEquals("defaultFile.txt", anotherArbDependency.getName());
    }

    @Resource
    @Qualifier("namedFile")
    public void setArbDependency(File arbDependency) {
        this.arbDependency = arbDependency;
    }

    @Resource
    @Qualifier("defaultFile")
    public void setAnotherArbDependency(File anotherArbDependency) {
        this.anotherArbDependency = anotherArbDependency;
    }
}
```

我们的测试表明，即使我们在应用程序上下文中定义了特定类型的多个 bean 实现，我们也可以使用一个`@Qualifier`注释和一个`@Resource`注释来解决依赖性。

类似于基于字段的依赖注入，如果我们在一个应用程序上下文中定义多个 bean，我们必须使用一个`@Qualifier `注释来指定使用哪个 bean 来解析依赖，否则将会抛出一个`NoUniqueBeanDefinitionException` 。

## 3。`@Inject`注解

`@Inject`注解属于 [JSR-330](https://web.archive.org/web/20220720170719/https://jcp.org/en/jsr/detail?id=330) 注解集合。此注释具有以下执行路径，按优先顺序列出:

1.  按类型匹配
2.  按限定符匹配
3.  按名称匹配

这些执行路径适用于 setter 和字段注入。为了访问`@Inject`注释，我们必须将`javax.inject`库声明为 Gradle 或 Maven 依赖项。

度:

```java
testCompile group: 'javax.inject', name: 'javax.inject', version: '1'
```

对于 Maven:

```java
<dependency>
    <groupId>javax.inject</groupId>
    <artifactId>javax.inject</artifactId>
    <version>1</version>
</dependency>
```

### 3.1。现场注射

#### 3.1.1。按类型匹配

我们将修改集成测试示例，以使用另一种类型的依赖，即`ArbitraryDependency`类。`ArbitraryDependency`类依赖关系仅仅是一个简单的依赖关系，没有进一步的意义:

```java
@Component
public class ArbitraryDependency {

    private final String label = "Arbitrary Dependency";

    public String toString() {
        return label;
    }
}
```

这里是有问题的`FieldInjectTest`集成测试:

```java
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(
  loader=AnnotationConfigContextLoader.class,
  classes=ApplicationContextTestInjectType.class)
public class FieldInjectIntegrationTest {

    @Inject
    private ArbitraryDependency fieldInjectDependency;

    @Test
    public void givenInjectAnnotation_WhenOnField_ThenValidDependency(){
        assertNotNull(fieldInjectDependency);
        assertEquals("Arbitrary Dependency",
          fieldInjectDependency.toString());
    }
}
```

与首先通过名称解析依赖关系的`@Resource`注释不同，`@Inject`注释的默认行为是通过类型解析依赖关系。

这意味着即使类引用变量名不同于 bean 名，只要 bean 是在应用程序上下文中定义的，依赖关系仍然会被解析。注意在下面的测试中如何引用变量名:

```java
@Inject
private ArbitraryDependency fieldInjectDependency;
```

与应用程序上下文中配置的 bean 名称不同:

```java
@Bean
public ArbitraryDependency injectDependency() {
    ArbitraryDependency injectDependency = new ArbitraryDependency();
    return injectDependency;
}
```

当我们执行测试时，我们能够解决依赖性。

#### 3.1.2。按限定符匹配

如果一个特定的类类型有多个实现，并且某个类需要一个特定的 bean，该怎么办？让我们修改集成测试示例，以便它需要另一个依赖项。

在这个例子中，我们子类化在按类型匹配例子中使用的`ArbitraryDependency`类，以创建`AnotherArbitraryDependency`类:

```java
public class AnotherArbitraryDependency extends ArbitraryDependency {

    private final String label = "Another Arbitrary Dependency";

    public String toString() {
        return label;
    }
}
```

每个测试用例的目标是确保我们将每个依赖项正确地注入到每个引用变量中:

```java
@Inject
private ArbitraryDependency defaultDependency;

@Inject
private ArbitraryDependency namedDependency;
```

我们可以使用`FieldQualifierInjectTest`集成测试来演示按限定符匹配:

```java
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(
  loader=AnnotationConfigContextLoader.class,
  classes=ApplicationContextTestInjectQualifier.class)
public class FieldQualifierInjectIntegrationTest {

    @Inject
    private ArbitraryDependency defaultDependency;

    @Inject
    private ArbitraryDependency namedDependency;

    @Test
    public void givenInjectQualifier_WhenOnField_ThenDefaultFileValid(){
        assertNotNull(defaultDependency);
        assertEquals("Arbitrary Dependency",
          defaultDependency.toString());
    }

    @Test
    public void givenInjectQualifier_WhenOnField_ThenNamedFileValid(){
        assertNotNull(defaultDependency);
        assertEquals("Another Arbitrary Dependency",
          namedDependency.toString());
    }
}
```

如果我们在一个应用程序上下文中有一个特定类的多个实现，并且`FieldQualifierInjectTest`集成测试试图以下面列出的方式注入依赖项，将会抛出一个`NoUniqueBeanDefinitionException`:

```java
@Inject 
private ArbitraryDependency defaultDependency;

@Inject 
private ArbitraryDependency namedDependency;
```

抛出这个异常是 Spring 框架指出某个类有多个实现，并且不知道使用哪个实现的方式。为了澄清混淆，我们可以转到`FieldQualifierInjectTest`集成测试的第 7 行和第 10 行:

```java
@Inject
private ArbitraryDependency defaultDependency;

@Inject
private ArbitraryDependency namedDependency;
```

我们可以将所需的 bean 名称传递给`@Qualifier`注释，它与`@Inject`注释一起使用。这是代码块现在的样子:

```java
@Inject
@Qualifier("defaultFile")
private ArbitraryDependency defaultDependency;

@Inject
@Qualifier("namedFile")
private ArbitraryDependency namedDependency;
```

`@Qualifier`注释在接收 bean 名称时期望严格匹配。我们必须确保 bean 名称被正确地传递给`Qualifier`，否则，将会抛出一个`NoUniqueBeanDefinitionException`。如果我们再做一次测试，应该会通过。

#### 3.1.3。按名称匹配

用于演示按名称匹配的`FieldByNameInjectTest`集成测试类似于按类型匹配的执行路径。唯一的区别是现在我们需要一个特定的 bean，而不是特定的类型。在这个例子中，我们再次子类化`ArbitraryDependency`类以产生`YetAnotherArbitraryDependency`类:

```java
public class YetAnotherArbitraryDependency extends ArbitraryDependency {

    private final String label = "Yet Another Arbitrary Dependency";

    public String toString() {
        return label;
    }
}
```

为了演示按名称匹配的执行路径，我们将使用以下集成测试:

```java
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(
  loader=AnnotationConfigContextLoader.class,
  classes=ApplicationContextTestInjectName.class)
public class FieldByNameInjectIntegrationTest {

    @Inject
    @Named("yetAnotherFieldInjectDependency")
    private ArbitraryDependency yetAnotherFieldInjectDependency;

    @Test
    public void givenInjectQualifier_WhenSetOnField_ThenDependencyValid(){
        assertNotNull(yetAnotherFieldInjectDependency);
        assertEquals("Yet Another Arbitrary Dependency",
          yetAnotherFieldInjectDependency.toString());
    }
}
```

我们列出了应用程序上下文:

```java
@Configuration
public class ApplicationContextTestInjectName {

    @Bean
    public ArbitraryDependency yetAnotherFieldInjectDependency() {
        ArbitraryDependency yetAnotherFieldInjectDependency =
          new YetAnotherArbitraryDependency();
        return yetAnotherFieldInjectDependency;
    }
}
```

如果我们运行集成测试，它会通过。

为了验证我们通过名称匹配执行路径注入了依赖项，我们需要将传递给`@Named`注释的值`yetAnotherFieldInjectDependency`更改为我们选择的另一个名称。当我们再次运行测试时，会抛出一个`NoSuchBeanDefinitionException`。

### 3.2。设定注射

用于`@Inject`注释的基于设置器的注入类似于用于`@Resource`基于设置器的注入的方法。我们没有注释引用变量，而是注释了相应的 setter 方法。基于字段的依赖注入所遵循的执行路径也适用于基于 setter 的注入。

## 4。`@Autowired`注解

`@Autowired`注释的行为类似于`@Inject`注释。唯一的区别是`@Autowired`注释是 Spring 框架的一部分。该注释与`@Inject`注释具有相同的执行路径，按优先顺序列出:

1.  按类型匹配
2.  按限定符匹配
3.  按名称匹配

这些执行路径适用于 setter 和字段注入。

### 4.1。现场注射

#### 4.1.1。按类型匹配

用于演示`@Autowired`按类型匹配执行路径的集成测试示例将类似于用于演示`@Inject`按类型匹配执行路径的测试。我们使用下面的`FieldAutowiredTest`集成测试来演示使用`@Autowired`注释的按类型匹配:

```java
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(
  loader=AnnotationConfigContextLoader.class,
  classes=ApplicationContextTestAutowiredType.class)
public class FieldAutowiredIntegrationTest {

    @Autowired
    private ArbitraryDependency fieldDependency;

    @Test
    public void givenAutowired_WhenSetOnField_ThenDependencyResolved() {
        assertNotNull(fieldDependency);
        assertEquals("Arbitrary Dependency", fieldDependency.toString());
    }
}
```

我们列出了这个集成测试的应用程序上下文:

```java
@Configuration
public class ApplicationContextTestAutowiredType {

    @Bean
    public ArbitraryDependency autowiredFieldDependency() {
        ArbitraryDependency autowiredFieldDependency =
          new ArbitraryDependency();
        return autowiredFieldDependency;
    }
}
```

我们使用这个集成测试来证明按类型匹配优先于其他执行路径。注意`FieldAutowiredTest`集成测试第 8 行的引用变量名:

```java
@Autowired
private ArbitraryDependency fieldDependency;
```

这不同于应用程序上下文中的 bean 名称:

```java
@Bean
public ArbitraryDependency autowiredFieldDependency() {
    ArbitraryDependency autowiredFieldDependency =
      new ArbitraryDependency();
    return autowiredFieldDependency;
}
```

当我们运行测试时，它应该会通过。

为了确认依赖关系确实是使用按类型匹配的执行路径解决的，我们需要改变引用变量`fieldDependency`的类型，并再次运行集成测试。这一次，`FieldAutowiredTest`集成测试将失败，并抛出一个`NoSuchBeanDefinitionException`。这验证了我们使用了按类型匹配来解决依赖关系。

#### 4.1.2。按限定符匹配

如果我们面临这样一种情况，我们已经在应用程序上下文中定义了多个 bean 实现:

```java
@Configuration
public class ApplicationContextTestAutowiredQualifier {

    @Bean
    public ArbitraryDependency autowiredFieldDependency() {
        ArbitraryDependency autowiredFieldDependency =
          new ArbitraryDependency();
        return autowiredFieldDependency;
    }

    @Bean
    public ArbitraryDependency anotherAutowiredFieldDependency() {
        ArbitraryDependency anotherAutowiredFieldDependency =
          new AnotherArbitraryDependency();
        return anotherAutowiredFieldDependency;
    }
}
```

如果我们执行下面的`FieldQualifierAutowiredTest`集成测试，将会抛出一个`NoUniqueBeanDefinitionException`:

```java
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(
  loader=AnnotationConfigContextLoader.class,
  classes=ApplicationContextTestAutowiredQualifier.class)
public class FieldQualifierAutowiredIntegrationTest {

    @Autowired
    private ArbitraryDependency fieldDependency1;

    @Autowired
    private ArbitraryDependency fieldDependency2;

    @Test
    public void givenAutowiredQualifier_WhenOnField_ThenDep1Valid(){
        assertNotNull(fieldDependency1);
        assertEquals("Arbitrary Dependency", fieldDependency1.toString());
    }

    @Test
    public void givenAutowiredQualifier_WhenOnField_ThenDep2Valid(){
        assertNotNull(fieldDependency2);
        assertEquals("Another Arbitrary Dependency",
          fieldDependency2.toString());
    }
}
```

这个例外是由于应用程序上下文中定义的两个 beans 造成的模糊性。Spring 框架不知道哪个 bean 依赖项应该自动绑定到哪个引用变量。我们可以通过向`FieldQualifierAutowiredTest`集成测试的第 7 行和第 10 行添加`@Qualifier`注释来解决这个问题:

```java
@Autowired
private FieldDependency fieldDependency1;

@Autowired
private FieldDependency fieldDependency2;
```

因此代码块如下所示:

```java
@Autowired
@Qualifier("autowiredFieldDependency")
private FieldDependency fieldDependency1;

@Autowired
@Qualifier("anotherAutowiredFieldDependency")
private FieldDependency fieldDependency2;
```

当我们再次运行测试时，它会通过。

#### 4.1.3。按名称匹配

我们将使用相同的集成测试场景来演示使用`@Autowired`注释注入字段依赖的名称匹配执行路径。当按名称自动连接依赖关系时，`@ComponentScan`注释必须与应用程序上下文`ApplicationContextTestAutowiredName`一起使用:

```java
@Configuration
@ComponentScan(basePackages={"com.baeldung.dependency"})
    public class ApplicationContextTestAutowiredName {
}
```

我们使用`@ComponentScan`注释在包中搜索用`@Component`注释过的 Java 类。例如，在应用程序上下文中，`com.baeldung.dependency`包将被扫描以寻找已经用`@Component`注释标注的类。在这个场景中，Spring 框架必须检测到具有`@Component`注释的`ArbitraryDependency`类:

```java
@Component(value="autowiredFieldDependency")
public class ArbitraryDependency {

    private final String label = "Arbitrary Dependency";

    public String toString() {
        return label;
    }
}
```

传递到`@Component`注释中的属性值`autowiredFieldDependency`，告诉 Spring 框架`ArbitraryDependency`类是一个名为`autowiredFieldDependency`的组件。为了让`@Autowired`注释通过名称来解析依赖关系，组件名必须与`FieldAutowiredNameTest`集成测试中定义的字段名一致；请参考第 8 行:

```java
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(
  loader=AnnotationConfigContextLoader.class,
  classes=ApplicationContextTestAutowiredName.class)
public class FieldAutowiredNameIntegrationTest {

    @Autowired
    private ArbitraryDependency autowiredFieldDependency;

    @Test
    public void givenAutowired_WhenSetOnField_ThenDependencyResolved(){
        assertNotNull(autowiredFieldDependency);
        assertEquals("Arbitrary Dependency",
          autowiredFieldDependency.toString());
	}
}
```

当我们运行`FieldAutowiredNameTest`集成测试时，它会通过。

但是我们怎么知道`@Autowired`注释真的调用了按名称匹配的执行路径呢？我们可以将引用变量`autowiredFieldDependency`的名称改为我们选择的另一个名称，然后再次运行测试。

这一次，测试将失败，并抛出一个`NoUniqueBeanDefinitionException`。类似的检查是将`@Component`属性值`autowiredFieldDependency`更改为我们选择的另一个值，并再次运行测试。还会扔一个`NoUniqueBeanDefinitionException` 。

这个异常证明了如果我们使用不正确的 bean 名称，将找不到有效的 bean。这就是我们如何知道按名称匹配的执行路径被调用的。

### 4.2。设定注射

针对`@Autowired`注释的基于 Setter 的注入类似于针对`@Resource`基于 setter 的注入所演示的方法。我们不是用`@Inject`注释来注释引用变量，而是注释相应的 setter。基于字段的依赖注入所遵循的执行路径也适用于基于 setter 的注入。

## 5。应用这些注释

这就提出了应该使用哪种注释以及在什么情况下使用的问题。这些问题的答案取决于所讨论的应用程序所面临的设计场景，以及开发人员希望如何利用基于每个注释的默认执行路径的多态性。

### 5.1。通过多态性在应用程序范围内使用单例

如果设计是这样的，应用程序行为基于接口或抽象类的实现，并且这些行为在整个应用程序中使用，那么我们可以使用`@Inject`或`@Autowired`注释。

这种方法的好处是，当我们升级应用程序或应用补丁来修复 bug 时，可以在对整个应用程序行为产生最小负面影响的情况下交换类。在这种情况下，主要的默认执行路径是按类型匹配。

### 5.2。通过多态性进行细粒度的应用行为配置

如果设计是这样的，应用程序具有复杂的行为，每个行为基于不同的接口/抽象类，并且每个实现的用法在应用程序中各不相同，那么我们可以使用`@Resource`注释。在这种情况下，主要的默认执行路径是按名称匹配。

### 5.3。依赖注入应该由 Jakarta EE 平台单独处理

如果设计要求所有依赖项都由 Jakarta EE 平台注入，而不是 Spring，那么就要在`@Resource`注释和`@Inject`注释之间做出选择。我们应该根据哪一个缺省执行路径是必需的来缩小两个注释之间的最终决定。

### 5.4。依赖注入应该由 Spring 框架单独处理

如果要求所有依赖项都由 Spring 框架处理，那么唯一的选择就是`@Autowired`注释。

### 5.5。讨论总结

下表总结了我们的讨论。

| 方案 | @资源 | @注入 | @自动连线 |
| 通过多态性在应用程序范围内使用单例 | 一千 | ✔ | ✔ |
| 通过多态性进行细粒度的应用行为配置 | ✔ | 一千 | 一千 |
| 依赖注入应该由 Jakarta EE 平台单独处理 | ✔ | ✔ | 一千 |
| 依赖注入应该由 Spring 框架单独处理 | 一千 | 一千 | ✔ |

## 6。结论

在本文中，我们旨在对每个注释的行为提供更深入的了解。理解每个注释的行为将有助于更好的整体应用程序设计和维护。

讨论中使用的代码可以在 [GitHub](https://web.archive.org/web/20220720170719/https://github.com/eugenp/tutorials/tree/master/spring-di-2) 上找到。