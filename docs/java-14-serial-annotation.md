# Java 14 中的@Serial 注释指南

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-14-serial-annotation>

## 1。简介

在这个快速教程中，我们将看看 Java 14 引入的新的`@Serial`注释。

与`@Override`类似，该注释与串行 lint 标志结合使用，对类的[序列化相关成员执行编译时检查。](/web/20221206171024/https://www.baeldung.com/java-serialization)

**虽然注释已经在版本 25 中可用，[lint 检查尚未发布。](https://web.archive.org/web/20221206171024/https://bugs.openjdk.java.net/browse/JDK-8202056)**

## 2。用途

让我们从用`@Serial`注释七个与序列化相关的方法和字段开始:

```java
public class MySerialClass implements Serializable {

    @Serial
    private static final ObjectStreamField[] serialPersistentFields = null;

    @Serial
    private static final long serialVersionUID = 1;

    @Serial
    private void writeObject(ObjectOutputStream stream) throws IOException {
        // ...
    }

    @Serial
    private void readObject(ObjectInputStream stream) throws IOException, ClassNotFoundException {
        // ...
    }

    @Serial
    private void readObjectNoData() throws ObjectStreamException {
        // ...
    }

    @Serial
    private Object writeReplace() throws ObjectStreamException {
        // ...
        return null;
    }

    @Serial
    private Object readResolve() throws ObjectStreamException {
        // ...
        return null;
    }

}
```

之后，我们需要用串行 lint 标志编译我们的类:

```java
javac -Xlint:serial MySerialClass.java
```

然后，编译器会检查被注释成员的签名和类型，如果它们与预期的不匹配，就会发出警告。

此外，如果应用了`@Serial`，编译器也会抛出一个错误:

*   当一个类没有实现`Serializable`接口时:

```java
public class MyNotSerialClass {
    @Serial 
    private static final long serialVersionUID = 1; // Compilation error
} 
```

*   无效的地方——例如枚举`,` [的任何序列化方法，因为这些方法被忽略](https://web.archive.org/web/20221206171024/https://docs.oracle.com/en/java/javase/11/docs/specs/serialization/serial-arch.html#serialization-of-enum-constants):

```java
public enum MyEnum { 
    @Serial 
    private void readObjectNoData() throws ObjectStreamException {} // Compilation error 
}
```

*   到 [`Externalizable`](/web/20221206171024/https://www.baeldung.com/java-externalizable) 类中的`writeObject()`、`readObject()`、`readObjectNoData()`和`serialPersistentFields`，因为这些类使用不同的序列化方法:

```java
public class MyExternalizableClass implements Externalizable {
    @Serial 
    private void writeObject(ObjectOutputStream stream) throws IOException {} // Compilation error 
}
```

## 3。结论

这篇短文介绍了新的`@Serial`注释用法。

和往常一样，文章中的所有代码都可以在 GitHub 上找到[。](https://web.archive.org/web/20221206171024/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-14)