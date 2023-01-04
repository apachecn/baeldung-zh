# 用 SnakeYAML 解析 YAML

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-snake-yaml>

## 1。概述

在本教程中，我们将学习如何使用 [SnakeYAML](https://web.archive.org/web/20221116165758/https://bitbucket.org/asomov/snakeyaml/overview) 库来**将 Java 对象序列化为 YAML 文档，反之亦然**。

## 2。项目设置

为了在我们的项目中使用 SnakeYAML，我们将添加以下 Maven 依赖项(最新版本可以在这里找到):

```java
<dependency>
    <groupId>org.yaml</groupId>
    <artifactId>snakeyaml</artifactId>
    <version>1.21</version>            
</dependency>
```

## 3。入口点

`Yaml`类是 API 的入口点:

```java
Yaml yaml = new Yaml();
```

由于实现不是线程安全的，不同的线程必须有自己的`Yaml`实例。

## 4。加载 YAML 文件

该库支持从`String`或`InputStream`加载文档。这里的大部分代码样本都是基于对`InputStream`的解析。

让我们首先定义一个简单的 YAML 文档，并将该文件命名为`customer.yaml`:

```java
firstName: "John"
lastName: "Doe"
age: 20
```

### 4.1。基本用法

现在我们将使用`Yaml` 类解析上面的 YAML 文档:

```java
Yaml yaml = new Yaml();
InputStream inputStream = this.getClass()
  .getClassLoader()
  .getResourceAsStream("customer.yaml");
Map<String, Object> obj = yaml.load(inputStream);
System.out.println(obj);
```

上述代码生成以下输出:

```java
{firstName=John, lastName=Doe, age=20}
```

默认情况下，`load()`方法返回一个`Map`实例。每次查询`Map`对象都需要我们预先知道属性键名，并且遍历嵌套属性也不容易。

### 4.2。自定义类型

库**还提供了一种将文档作为自定义类**加载的方法。这个选项允许在内存中轻松遍历数据。

让我们定义一个`Customer`类，并尝试再次加载文档:

```java
public class Customer {

    private String firstName;
    private String lastName;
    private int age;

    // getters and setters
}
```

假设要反序列化的 YAML 文档是一个已知类型，我们可以在文档中指定一个显式的全局`tag`。

让我们更新文档并将其存储在一个新文件`customer_with_type.yaml:`

```java
!!com.baeldung.snakeyaml.Customer
firstName: "John"
lastName: "Doe"
age: 20
```

请注意文档中的第一行，它保存了加载时要使用的类的信息。

现在我们将更新上面使用的代码，并传递新的文件名作为输入:

```java
Yaml yaml = new Yaml();
InputStream inputStream = this.getClass()
 .getClassLoader()
 .getResourceAsStream("yaml/customer_with_type.yaml");
Customer customer = yaml.load(inputStream); 
```

`load()` 方法现在返回一个`Customer` 类型`. ` **的实例。这种方法的缺点是该类型必须导出为一个库，以便在需要的地方使用**。

但是，我们可以使用显式的本地标签，因为我们不需要导出库。

**加载自定义类型的另一种方式是使用`Constructor`类**。这样，我们可以为要解析的 YAML 文档指定根类型。让我们创建一个以`Customer`类型为根类型的`Constructor`实例，并将其传递给`Yaml`实例。

现在加载`customer.yaml, `时，我们将得到`Customer`对象:

```java
Yaml yaml = new Yaml(new Constructor(Customer.class));
```

### 4.3。隐式类型

如果给定的属性没有定义类型，库会自动将值转换为隐式类型。

例如:

```java
1.0 -> Float
42 -> Integer
2009-03-30 -> Date
```

让我们使用一个测试用例来测试这个隐式类型转换:

```java
@Test
public void whenLoadYAML_thenLoadCorrectImplicitTypes() {
   Yaml yaml = new Yaml();
   Map<Object, Object> document = yaml.load("3.0: 2018-07-22");

   assertNotNull(document);
   assertEquals(1, document.size());
   assertTrue(document.containsKey(3.0d));   
}
```

### 4.4。嵌套对象和集合

给定一个顶级类型，库自动检测嵌套对象的类型，除非它们是接口或抽象类，并将文档反序列化为相关的嵌套类型。

让我们将`Contact`和`Address `细节添加到`customer.yaml,`中，并将新文件保存为`customer_with_contact_details_and_address.yaml. `

现在我们将解析新的 YAML 文档:

```java
firstName: "John"
lastName: "Doe"
age: 31
contactDetails:
   - type: "mobile"
     number: 123456789
   - type: "landline"
     number: 456786868
homeAddress:
   line: "Xyz, DEF Street"
   city: "City Y"
   state: "State Y"
   zip: 345657 
```

`Customer`类也应该反映这些变化。下面是更新后的类:

```java
public class Customer {
    private String firstName;
    private String lastName;
    private int age;
    private List<Contact> contactDetails;
    private Address homeAddress;    
    // getters and setters
} 
```

让我们看看`Contact`和`Address`类是什么样子的:

```java
public class Contact {
    private String type;
    private int number;
    // getters and setters
}
```

```java
public class Address {
    private String line;
    private String city;
    private String state;
    private Integer zip;
    // getters and setters
}
```

现在我们将使用给定的测试用例来测试`Yaml` # `load()` :

```java
@Test
public void 
  whenLoadYAMLDocumentWithTopLevelClass_thenLoadCorrectJavaObjectWithNestedObjects() {

    Yaml yaml = new Yaml(new Constructor(Customer.class));
    InputStream inputStream = this.getClass()
      .getClassLoader()
      .getResourceAsStream("yaml/customer_with_contact_details_and_address.yaml");
    Customer customer = yaml.load(inputStream);

    assertNotNull(customer);
    assertEquals("John", customer.getFirstName());
    assertEquals("Doe", customer.getLastName());
    assertEquals(31, customer.getAge());
    assertNotNull(customer.getContactDetails());
    assertEquals(2, customer.getContactDetails().size());

    assertEquals("mobile", customer.getContactDetails()
      .get(0)
      .getType());
    assertEquals(123456789, customer.getContactDetails()
      .get(0)
      .getNumber());
    assertEquals("landline", customer.getContactDetails()
      .get(1)
      .getType());
    assertEquals(456786868, customer.getContactDetails()
      .get(1)
      .getNumber());
    assertNotNull(customer.getHomeAddress());
    assertEquals("Xyz, DEF Street", customer.getHomeAddress()
      .getLine());
}
```

### 4.5。类型安全集合

当给定 Java 类的一个或多个属性是类型安全(泛型)集合时，指定`TypeDescription` 以便识别正确的参数化类型是很重要的。

让我们取一个有不止一个`Contact`的`Customer`，并尝试加载它:

```java
firstName: "John"
lastName: "Doe"
age: 31
contactDetails:
   - { type: "mobile", number: 123456789}
   - { type: "landline", number: 123456789}
```

为了加载这个文档，**我们可以为顶层类**上的给定属性指定`TypeDescription `:

```java
Constructor constructor = new Constructor(Customer.class);
TypeDescription customTypeDescription = new TypeDescription(Customer.class);
customTypeDescription.addPropertyParameters("contactDetails", Contact.class);
constructor.addTypeDescription(customTypeDescription);
Yaml yaml = new Yaml(constructor);
```

### 4.6。加载多个文档

可能有这样的情况，在一个单独的`File`中有几个 YAML 文档，我们想要解析所有的文档。`Yaml` 类提供了一个`loadAll()` 方法来完成这种类型的解析。

默认情况下，该方法返回一个`Iterable<Object>`的实例，其中每个对象都是`Map<String, Object>. `类型。如果需要自定义类型，那么我们可以使用上面讨论的`Constructor`实例`. `

考虑单个文件中的以下文档:

```java
---
firstName: "John"
lastName: "Doe"
age: 20
---
firstName: "Jack"
lastName: "Jones"
age: 25
```

我们可以使用下面的代码示例所示的`loadAll()`方法解析上面的代码:

```java
@Test
public void whenLoadMultipleYAMLDocuments_thenLoadCorrectJavaObjects() {
    Yaml yaml = new Yaml(new Constructor(Customer.class));
    InputStream inputStream = this.getClass()
      .getClassLoader()
      .getResourceAsStream("yaml/customers.yaml");

    int count = 0;
    for (Object object : yaml.loadAll(inputStream)) {
        count++;
        assertTrue(object instanceof Customer);
    }
    assertEquals(2,count);
}
```

## 5。倾销 YAML 文件

这个库还提供了一个方法来把一个给定的 Java 对象转储到一个 YAML 文档中。输出可以是一个`String`或者一个指定的文件/流。

### 5.1。基本用法

我们从一个简单的例子开始，将 `Map<String, Object>`的实例转储到 YAML 文档(`String`):

```java
@Test
public void whenDumpMap_thenGenerateCorrectYAML() {
    Map<String, Object> data = new LinkedHashMap<String, Object>();
    data.put("name", "Silenthand Olleander");
    data.put("race", "Human");
    data.put("traits", new String[] { "ONE_HAND", "ONE_EYE" });
    Yaml yaml = new Yaml();
    StringWriter writer = new StringWriter();
    yaml.dump(data, writer);
    String expectedYaml = "name: Silenthand Olleander\nrace: Human\ntraits: [ONE_HAND, ONE_EYE]\n";

    assertEquals(expectedYaml, writer.toString());
}
```

上面的代码产生了下面的输出(注意，使用一个`LinkedHashMap`的实例保留了输出数据的顺序):

```java
name: Silenthand Olleander
race: Human
traits: [ONE_HAND, ONE_EYE]
```

### 5.2。自定义 Java 对象

我们也可以选择**将定制的 Java 类型转储到输出流**。但是，这会将全局显式`tag`添加到输出文档中:

```java
@Test
public void whenDumpACustomType_thenGenerateCorrectYAML() {
    Customer customer = new Customer();
    customer.setAge(45);
    customer.setFirstName("Greg");
    customer.setLastName("McDowell");
    Yaml yaml = new Yaml();
    StringWriter writer = new StringWriter();
    yaml.dump(customer, writer);        
    String expectedYaml = "!!com.baeldung.snakeyaml.Customer {age: 45, contactDetails: null, firstName: Greg,\n  homeAddress: null, lastName: McDowell}\n";

    assertEquals(expectedYaml, writer.toString());
}
```

使用上面的方法，我们仍然在 YAML 文档中转储标签信息。

这意味着我们必须将我们的类导出为一个库，供任何反序列化它的用户使用。为了避免输出文件中的标签名，我们可以使用库提供的`dumpAs()`方法。

因此，在上面的代码中，我们可以调整以下内容来删除标签:

```java
yaml.dumpAs(customer, Tag.MAP, null);
```

## 6。结论

本文展示了 SnakeYAML 库的用法，将 Java 对象序列化为 YAML 对象，反之亦然。

所有的例子都可以在 GitHub 项目中找到——这是一个基于 Maven 的项目，所以它应该很容易导入和运行。