# 在 Java 中访问 Maven 属性

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-accessing-maven-properties>

## 1。概述

在这个简短的教程中，我们将看看如何在 Java 应用程序中使用 Maven 的`pom.xml`中定义的变量。

## 2。插件配置

在整个例子中，我们将使用 [Maven 属性插件](https://web.archive.org/web/20220628052152/https://www.mojohaus.org/properties-maven-plugin/)。

**这个插件将绑定到`generate-resources`阶段，并在编译期间创建一个包含在`pom.xml`中定义的变量的文件。然后，我们可以在运行时读取该文件来获取值。**

让我们从在我们的项目中包含插件开始:

```java
<plugin>
    <groupId>org.codehaus.mojo</groupId> 
    <artifactId>properties-maven-plugin</artifactId> 
    <version>1.0.0</version> 
    <executions> 
        <execution> 
            <phase>generate-resources</phase> 
            <goals> 
                <goal>write-project-properties</goal> 
            </goals> 
            <configuration> 
                <outputFile>${project.build.outputDirectory}/properties-from-pom.properties</outputFile> 
            </configuration> 
        </execution> 
    </executions> 
</plugin> 
```

接下来，我们将继续为变量赋值。**此外，因为我们在`pom.xml`中定义它们，我们也可以使用 [Maven 占位符](https://web.archive.org/web/20220628052152/https://github.com/cko/predefined_maven_properties/blob/master/README.md):**

```java
<properties> 
    <name>${project.name}</name> 
    <my.awesome.property>property-from-pom</my.awesome.property> 
</properties>
```

## 3。读取属性

现在是时候从配置中访问我们的属性了。让我们创建一个简单的实用程序类，从类路径上的文件中读取属性:

```java
public class PropertiesReader {
    private Properties properties;

    public PropertiesReader(String propertyFileName) throws IOException {
        InputStream is = getClass().getClassLoader()
            .getResourceAsStream(propertyFileName);
        this.properties = new Properties();
        this.properties.load(is);
    }

    public String getProperty(String propertyName) {
        return this.properties.getProperty(propertyName);
    }
}
```

接下来，我们简单地编写一个小的测试用例来读取我们的值:

```java
PropertiesReader reader = new PropertiesReader("properties-from-pom.properties"); 
String property = reader.getProperty("my.awesome.property");
Assert.assertEquals("property-from-pom", property);
```

## 4。结论

在本文中，我们经历了使用 Maven 属性插件读取在`pom.xml`中定义的值的过程。

和往常一样，GitHub 上的所有代码[都是可用的。](https://web.archive.org/web/20220628052152/https://github.com/eugenp/tutorials/tree/master/maven-modules/maven-properties)