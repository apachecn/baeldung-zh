# JiBX 简介

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/jibx>

## 1。概述

JiBX 是一个将 XML 数据绑定到 Java 对象的工具。与 JAXB 等其他常用工具相比，它提供了可靠的性能。

与其他 Java-XML 工具相比，JiBX 也非常灵活，它使用绑定定义将 Java 结构从 XML 表示中分离出来，这样就可以独立地改变每一种结构。

在本文中，我们将探索 JiBX 提供的将 XML 绑定到 Java 对象的不同方式。

## 2。JiBX 的组件

### 2.1。绑定定义文件

绑定定义文档指定了 Java 对象如何与 XML 相互转换。

JiBX 绑定编译器将一个或多个绑定定义以及实际的类文件作为输入。它通过将绑定定义添加到类文件中，将它编译成 Java 字节码。一旦类文件用编译后的绑定定义代码进行了增强，它们就可以使用 JiBX 运行时了。

### 2.2。工具

我们将使用三种主要工具:

*   **`BindGen`**–从 Java 代码中生成绑定和匹配的模式定义
*   **`CodeGen`**–从 XML 模式创建 Java 代码和绑定定义
*   **`JiBX2Wsdl`**–从现有的 Java 代码中创建绑定定义和匹配的 WSDL 以及模式定义

## 3。Maven 配置

### 3.1。依赖性

我们需要在`pom.xml`中添加 jibx-run 依赖关系:

```java
<dependency>
    <groupId>org.jibx</groupId>
    <artifactId>jibx-run</artifactId>
    <version>1.3.1</version>
</dependency>
```

这个依赖关系的最新版本可以在[这里](https://web.archive.org/web/20220625221809/https://search.maven.org/classic/#search%7Cga%7C1%7Cg%3A%22org.jibx%22%20AND%20a%3A%22jibx-run%22)找到。

### 3.2。插件

为了在 JiBX 中执行不同的步骤，比如代码生成或绑定生成，我们需要在`pom.xml`中配置`maven-jibx-plugin`。

对于我们需要从 Java 代码开始并生成绑定和模式定义的情况，让我们配置插件:

```java
<plugin>
    <groupId>org.jibx</groupId>
    <artifactId>maven-jibx-plugin</artifactId>
    ...
    <configuration>
        <directory>src/main/resources</directory>
        <includes>
            <includes>*-binding.xml</includes>
        </includes>
        <excludes>
            <exclude>template-binding.xml</exclude>
        </excludes>
        <verbose>true</verbose>
    </configuration>
    <executions>
        <execution>
            <phase>process-classes</phase>
            <goals>
                <goal>bind</goal>
            </goals>
        </execution>
    </executions>
</plugin>
```

当我们有了一个模式并生成了 Java 代码和绑定定义时，`maven-jibx-plugin`被配置了关于模式文件路径和源代码目录路径的信息:

```java
<plugin>
    <groupId>org.jibx</groupId>
    <artifactId>maven-jibx-plugin</artifactId>
    ...
    <executions>
        <execution>
        <id>generate-java-code-from-schema</id>
        <goals>
             <goal>schema-codegen</goal>
        </goals>
            <configuration>
                <directory>src/main/jibx</directory>
                <includes>
                    <include>customer-schema.xsd</include>
                </includes>
                <verbose>true</verbose>
            </configuration>
            </execution>
            <execution>
                <id>compile-binding</id>
                <goals>
                    <goal>bind</goal>
                </goals>
            <configuration>
                <directory>target/generated-sources</directory>
                <load>true</load>
                <validate>true</validate>
                <verify>true</verify>
            </configuration>
        </execution>
    </executions>
</plugin>
```

## 4。绑定定义

绑定定义是 JiBX 的核心部分。基本绑定文件指定了 XML 和 Java 对象字段之间的映射:

```java
<binding>
    <mapping name="customer" class="com.baeldung.xml.jibx.Customer">
        ...
        <value name="city" field="city" />
    </mapping>
</binding>
```

### 4.1。结构映射

结构映射使 XML 结构看起来类似于对象结构:

```java
<binding>
    <mapping name="customer" class="com.baeldung.xml.jibx.Customer">
    ...
    <structure name="person" field="person">
        ...
        <value name="last-name" field="lastName" />
    </structure>
    ...    
    </mapping>
</binding>
```

该结构的相应类将是:

```java
public class Customer {

    private Person person;
    ...

    // standard getters and setters

}

public class Person {

    private String lastName;
    ...

    // standard getters and setters

} 
```

### 4.2。`Collection`和`Array`映射

JiBX 绑定提供了一种处理对象集合的简单方法:

```java
<mapping class="com.baeldung.xml.jibx.Order" name="Order">
    <collection get-method="getAddressList" 
      set-method="setAddressList" usage="optional" 
      createtype="java.util.ArrayList">

        <structure type="com.baeldung.xml.jibx.Order$Address" 
          name="Address">
            <value style="element" name="Name" 
              get-method="getName" set-method="setName"/>
              ...
        </structure>
     ...
</mapping>
```

让我们看看相应的映射 Java 对象:

```java
public class Order {
    List<Address> addressList = new ArrayList<>();
    ...

    // getters and setters here
}

public static class Address {
    private String name;

    ...
    // standard getters and setter

}
```

### 4.3。高级映射

到目前为止，我们已经看到了一个基本的映射定义。JiBX 映射提供了不同风格的映射，比如抽象映射和映射继承。

让我们看看如何定义一个抽象映射:

```java
<binding>
    <mapping name="customer" 
      class="com.baeldung.xml.jibx.Customer">

        <structure name="person" field="person">
            ...
            <value name="name" field="name" />
        </structure>
        <structure name="home-phone" field="homePhone" />
        <structure name="office-phone" field="officePhone" />
        <value name="city" field="city" />
    </mapping>

    <mapping name="phone" 
      class="com.baeldung.xml.jibx.Phone" abstract="true">
        <value name="number" field="number"/>
    </mapping>
</binding>
```

让我们看看它是如何绑定到 Java 对象的:

```java
public class Customer {
    private Person person;
    ...
    private Phone homePhone;
    private Phone officePhone;

    // standard getters and setters

}
```

这里我们在`Customer`类中指定了多个`Phone`字段。这个`Phone` 本身又是一个 POJO:

```java
public class Phone {

    private String number;

    // standard getters and setters
}
```

除了常规映射，我们还可以定义扩展。每个扩展映射都引用一些基本映射。在封送处理时，实际的对象类型决定应用哪个 XML 映射。

让我们看看扩展是如何工作的:

```java
<binding>
    <mapping class="com.baeldung.xml.jibx.Identity" 
      abstract="true">
        <value name="customer-id" field="customerId"/>
    </mapping>
    ...  
    <mapping name="person" 
      class="com.baeldung.xml.jibx.Person" 
      extends="com.baeldung.xml.jibx.Identity">
        <structure map-as="com.baeldung.xml.jibx.Identity"/>
        ...
    </mapping>
    ...
</binding>
```

让我们看看相应的 Java 对象:

```java
public class Identity {

    private long customerId;

    // standard getters and setters
}
```

## 5。结论

在这篇简短的文章中，我们探索了使用 JiBX 在 XML 和 Java 对象之间进行转换的不同方法。我们还看到了如何使用绑定定义来处理不同的表示。

GitHub 上的[提供了这篇文章的完整代码。](https://web.archive.org/web/20220625221809/https://github.com/eugenp/tutorials/tree/master/xml)