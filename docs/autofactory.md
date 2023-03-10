# 自动工厂简介

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/autofactory>

## 1。简介

在本教程中，我们将简要介绍来自谷歌的 [`AutoFactory`](https://web.archive.org/web/20220628151134/https://github.com/google/auto/tree/master/factory) 。

这是一个帮助生成工厂的源代码级代码生成器。

## 2。Maven 设置

在我们开始之前，让我们将下面的依赖项添加到`pom.xml:`

```java
<dependency>
    <groupId>com.google.auto.factory</groupId>
    <artifactId>auto-factory</artifactId>
    <version>1.0-beta5</version>
</dependency>
```

最新版本可以在这里找到[。](https://web.archive.org/web/20220628151134/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22com.google.auto.factory%22%20AND%20a%3A%22auto-factory%22)

## 3。快速入门

现在让我们快速看一下`AutoFactory`能做什么，并创建一个简单的`Phone`类。

因此，当我们用`@AutoFactory`注释`Phone`类，用`@Provided`注释其构造函数参数时，我们得到:

```java
@AutoFactory
public class Phone {

    private final Camera camera;

    private final String otherParts;

    PhoneAssembler(@Provided Camera camera, String otherParts) {
        this.camera = camera;
        this.otherParts = otherParts;
    }

    //...

}
```

我们只用了两个标注: [`@AutoFactory`](https://web.archive.org/web/20220628151134/https://github.com/google/auto/blob/master/factory/src/main/java/com/google/auto/factory/AutoFactory.java) 和 [`@Provided`](https://web.archive.org/web/20220628151134/https://github.com/google/auto/blob/master/factory/src/main/java/com/google/auto/factory/Provided.java) 。当我们需要为我们的类生成一个工厂时，我们可以用`@AutoFactory,` 来注释它，而`@Provided`适用于这个类的构造函数参数，这意味着注释的参数应该由一个注入的 [`Provider`](https://web.archive.org/web/20220628151134/https://docs.oracle.com/javaee/6/api/javax/inject/Provider.html) 来提供。

在上面的代码片段中，我们期望`Camera`由任何相机制造商提供，而`AutoFactory`将帮助生成以下代码:

```java
@Generated("com.google.auto.factory.processor.AutoFactoryProcessor")
public final class PhoneFactory {

    private final Provider<Camera> cameraProvider;

    @Inject
    PhoneAssemblerFactory(Provider<Camera> cameraProvider) {
        this.cameraProvider = checkNotNull(cameraProvider, 1);
    }

    PhoneAssembler create(String otherParts) {
      return new PhoneAssembler(
        checkNotNull(cameraProvider.get(), 1),
        checkNotNull(otherParts, 2));
    }

    // ...

}
```

现在我们有了一个由`AutoFactory`在编译时自动生成的`PhoneFactory`，我们可以用它来生成 phone 实例:

```java
PhoneFactory phoneFactory = new PhoneFactory(
  () -> new Camera("Unknown", "XXX"));
Phone simplePhone = phoneFactory.create("other parts");
```

`@AutoFactory`注释也可以应用于构造函数:

```java
public class ClassicPhone {

    private final String dialpad;
    private final String ringer;
    private String otherParts;

    @AutoFactory
    public ClassicPhone(
      @Provided String dialpad, @Provided String ringer) {
        this.dialpad = dialpad;
        this.ringer = ringer;
    }

    @AutoFactory
    public ClassicPhone(String otherParts) {
        this("defaultDialPad", "defaultRinger");
        this.otherParts = otherParts;
    }

    //...

}
```

在上面的代码片段中，我们对两个构造函数都应用了`@AutoFactory`。`AutoFactory`将简单地为我们相应地生成两种创建方法:

```java
@Generated(value = "com.google.auto.factory.processor.AutoFactoryProcessor")
public final class ClassicPhoneFactory {
    private final Provider<String> java_lang_StringProvider;

    @Inject
    public ClassicPhoneFactory(Provider<String> java_lang_StringProvider) {
        this.java_lang_StringProvider =
          checkNotNull(java_lang_StringProvider, 1);
    }

    public ClassicPhone create() {
        return new ClassicPhone(
          checkNotNull(java_lang_StringProvider.get(), 1),
          checkNotNull(java_lang_StringProvider.get(), 2));
    }

    public ClassicPhone create(String otherParts) {
        return new ClassicPhone(checkNotNull(otherParts, 1));
    }

    //...

}
```

**`AutoFactory`也支持用`@Provided`标注的参数，但只针对 [JSR-330](https://web.archive.org/web/20220628151134/https://jcp.org/en/jsr/detail?id=330) 标注。**

例如，如果我们希望`cameraProvider`是“Sony ”,我们可以将`Phone`类改为:

```java
@AutoFactory
public class Phone {

    PhoneAssembler(
      @Provided @Named("Sony") Camera camera, String otherParts) {
        this.camera = camera;
        this.otherParts = otherParts;
    }

    //...

}
```

AutoFactory 将保留 [`@Named`](https://web.archive.org/web/20220628151134/https://docs.oracle.com/javaee/6/api/javax/inject/Named.html) `[@Qualifier](https://web.archive.org/web/20220628151134/https://docs.oracle.com/javaee/6/api/javax/inject/Qualifier.html)`以便我们可以利用它，例如，在使用依赖注入框架时:

```java
@Generated("com.google.auto.factory.processor.AutoFactoryProcessor")
public final class PhoneFactory {

    private final Provider<Camera> cameraProvider;

    @Inject
    PhoneAssemblerFactory(@Named("Sony") Provider<Camera> cameraProvider) {
      this.cameraProvider = checkNotNull(cameraProvider, 1);
    }

    //...

}
```

## 4。定制代码生成

我们可以使用几个属性和`@AutoFactory`注释来定制生成的代码。

### 4.1。自定义类名

生成的工厂类的名称可以用 [`className`](https://web.archive.org/web/20220628151134/https://github.com/google/auto/blob/master/factory/src/main/java/com/google/auto/factory/AutoFactory.java#L48) 设置:

```java
@AutoFactory(className = "SamsungFactory")
public class SmartPhone {

    //...

}
```

有了上面的配置，我们将创建一个名为`SamsungFactory`的类:

```java
@Generated("com.google.auto.factory.processor.AutoFactoryProcessor")
public final class SamsungFactory {

    //...

}
```

### 4.2。`non-final`工厂

注意，默认情况下，生成的工厂类被标记为 final，因此我们可以通过将 [`allowSubclasses`](https://web.archive.org/web/20220628151134/https://github.com/google/auto/blob/master/factory/src/main/java/com/google/auto/factory/AutoFactory.java#L64) 属性设置为`false:`来改变这种行为

```java
@AutoFactory(
  className = "SamsungFactory", 
  allowSubclasses = true)
public class SmartPhone {

    //...

}
```

现在我们有:

```java
@Generated("com.google.auto.factory.processor.AutoFactoryProcessor")
public class SamsungFactory {

    //...

}
```

### 4.3。更多功能

另外，我们可以使用 ["](https://web.archive.org/web/20220628151134/https://github.com/google/auto/blob/master/factory/src/main/java/com/google/auto/factory/AutoFactory.java#L53) `[implementing”](https://web.archive.org/web/20220628151134/https://github.com/google/auto/blob/master/factory/src/main/java/com/google/auto/factory/AutoFactory.java#L53) parameter.`为生成的工厂指定要实现的接口列表

这里我们需要 *SamsungFactory* 来生产具有可定制存储的智能手机:

```java
public interface CustomStorage {
    SmartPhone customROMInGB(int romSize);
}
```

**注意，接口中的方法应该返回基类`SmartPhone`的实例。**

**然后，`AutoFactory`需要基类**中的相关构造函数来生成实现了上述接口的工厂类:

```java
@AutoFactory(
  className = "SamsungFactory",
  allowSubclasses = true,
  implementing = CustomStorage.class)
public class SmartPhone {

    public SmartPhone(int romSize){
        //...
    }

    //...

}
```

因此，`AutoFactory`将生成以下代码:

```java
@Generated("com.google.auto.factory.processor.AutoFactoryProcessor")
public class SamsungFactory implements CustomStorage {

    //...

    public SmartPhone create(int romSize) {
        return new SmartPhone(romSize);
    }

    @Override
    public SmartPhone customROMInGB(int romSize) {
        return create(romSize);
    }
}
```

### 4.4。工厂扩建

由于`AutoFactory`可以生成接口实现，自然期望它也能够扩展类，这确实是可能的:

```java
public abstract class AbstractFactory {
    abstract CustomPhone newInstance(String brand);
}

@AutoFactory(extending = AbstractFactory.class)
public class CustomPhone {

    private final String brand;

    public CustomPhone(String brand) {
        this.brand = brand;
    }
}
```

在这里，我们使用 [`extending`](https://web.archive.org/web/20220628151134/https://github.com/google/auto/blob/master/factory/src/main/java/com/google/auto/factory/AutoFactory.java#L58) 扩展了`AbstractFactory`类。另外，我们应该**注意，基础抽象类(`AbstractFactory`)中的每个抽象方法都应该在具体类(`CustomPhone` )** 中有一个对应的构造函数。

最后，我们可以看到以下生成的代码:

```java
@Generated("com.google.auto.factory.processor.AutoFactoryProcessor")
public final class CustomPhoneFactory extends AbstractFactory {

    @Inject
    public CustomPhoneFactory() {
    }

    public CustomPhone create(String brand) {
        return new CustomPhone(checkNotNull(brand, 1));
    }

    @Override
    public CustomPhone newInstance(String brand) {
        return create(brand);
    }

    //...

}
```

我们可以看到,`AutoFactory`足够聪明，可以利用构造函数来实现相应的抽象方法——像`AutoFactory`中这样的强大功能肯定会为我们节省大量时间和代码。

## 5。`AutoFactory`同`Guice`

正如我们在本文前面提到的，`AutoFactory`支持 [JSR-330 注释](https://web.archive.org/web/20220628151134/https://docs.oracle.com/javaee/6/api/javax/inject/package-summary.html)，因此我们可以将现有的依赖注入框架与它集成。

首先，我们把 [`Guice`](https://web.archive.org/web/20220628151134/https://github.com/google/guice) 加到`pom.xml`上:

```java
<dependency>
    <groupId>com.google.inject</groupId>
    <artifactId>guice</artifactId>
    <version>4.2.0</version>
</dependency>
```

最新版本的`Guice`可以在[这里](https://web.archive.org/web/20220628151134/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22com.google.inject%22%20AND%20a%3A%22guice%22)找到。

现在，我们将展示`AutoFactory`与`Guice`的集成有多好。

因为我们期望“Sony”成为摄像机提供者，所以我们需要给`PhoneFactory`的构造函数注入一个`SonyCameraProvider`:

```java
@Generated("com.google.auto.factory.processor.AutoFactoryProcessor")
public final class PhoneFactory {

    private final Provider<Camera> cameraProvider;

    @Inject
    public PhoneFactory(@Named("Sony") Provider<Camera> cameraProvider) {
        this.cameraProvider = checkNotNull(cameraProvider, 1);
    }

    //...

}
```

最后，我们将在一个`Guice`模块中进行绑定:

```java
public class SonyCameraModule extends AbstractModule {

    private static int SONY_CAMERA_SERIAL = 1;

    @Named("Sony")
    @Provides
    Camera cameraProvider() {
        return new Camera(
          "Sony", String.format("%03d", SONY_CAMERA_SERIAL++));
    }

}
```

我们在`SonyCameraModule`中设置标注了`@Named(“Sony”)`的摄像机提供者来匹配`PhoneFactory`的构造器参数。

现在我们可以看到`Guice`正在为我们生成的工厂管理依赖注入:

```java
Injector injector = Guice.createInjector(new SonyCameraModule());
PhoneFactory injectedFactory = injector.getInstance(PhoneFactory.class);
Phone xperia = injectedFactory.create("Xperia");
```

## 6。引擎盖下

**`AutoFactory`提供的所有注释都在编译阶段**进行处理，我们在文章中已经详细解释过:[源代码级的注释处理是如何工作的。](/web/20220628151134/https://www.baeldung.com/java-annotation-processing-builder)

## 7。结论

在本文中，我们介绍了如何使用 AutoFactory，以及如何将其与`Guice –` 集成。编写工厂可能会重复且容易出错——像`AutoFactory`和 [`AutoValue`](/web/20220628151134/https://www.baeldung.com/introduction-to-autovalue) 这样的代码生成工具可以为我们节省大量时间，并使我们摆脱微妙的错误。

和往常一样，代码示例的完整实现可以在 Github 上找到[。](https://web.archive.org/web/20220628151134/https://github.com/eugenp/tutorials/tree/master/code-generation)