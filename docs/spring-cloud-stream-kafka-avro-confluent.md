# 使用 Kafka、Apache Avro 和合流模式注册表的 Spring Cloud Stream 指南

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-cloud-stream-kafka-avro-confluent>

## 1.介绍

Apache Kafka 是一个消息平台。有了它，我们可以在大规模的不同应用程序之间交换数据。

Spring Cloud Stream 是一个构建消息驱动应用的框架。它可以简化 Kafka 与我们服务的整合。

按照惯例，Kafka 与 Avro 消息格式一起使用，由模式注册表支持。在本教程中，我们将使用合流模式注册表。我们将尝试 Spring 与融合模式注册中心的集成实现，以及融合本地库。

## 2.融合模式注册表

Kafka 将所有数据表示为字节，因此常见的是**使用外部模式，并根据该模式序列化和反序列化为字节**。与其为每条消息提供一个模式副本(这是一个昂贵的开销),还不如将模式保存在注册表中，只为每条消息提供一个 id。

融合模式注册表提供了一种简单的方法来存储、检索和管理模式。它公开了几个有用的 RESTful APIs。

模式是按主题存储的，默认情况下，注册中心会在允许针对主题上传新模式之前进行兼容性检查。

每个生产者都知道自己生产的模式，每个消费者应该能够消费任何格式的数据，或者应该有自己喜欢的特定模式。当发送消息时，生产者咨询注册中心以建立正确的 ID 来使用。消费者使用注册中心获取发送者的模式。

当消费者知道发送者的模式和它自己想要的消息格式时，Avro 库可以将数据转换成消费者想要的格式。

## 3.阿帕奇 Avro

**[Apache Avro](/web/20220628152326/https://www.baeldung.com/java-apache-avro) 是一个数据序列化系统**。

它使用 JSON 结构来定义模式，提供字节和结构化数据之间的序列化。

Avro 的一个优势是它支持把用一个版本的模式编写的消息发展成兼容的替代模式定义的格式。

Avro 工具集还能够生成表示这些模式的数据结构的类，这使得序列化进出 POJOs 变得很容易。

## 4.设置项目

要使用带有 [Spring Cloud Stream](https://web.archive.org/web/20220628152326/https://search.maven.org/search?q=g:org.springframework.cloud%20AND%20a:spring-cloud-dependencies&core=gav) 的模式注册表，我们需要 [Spring Cloud Kafka Binder](https://web.archive.org/web/20220628152326/https://search.maven.org/search?q=a:spring-cloud-stream-binder-kafka) 和[模式注册表](https://web.archive.org/web/20220628152326/https://search.maven.org/search?q=g:org.springframework.cloud%20AND%20a:spring-cloud-stream-schema) Maven 依赖项:

```java
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-stream-binder-kafka</artifactId>
</dependency>

<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-stream-schema</artifactId>
</dependency>
```

对于[汇合器的串行器](https://web.archive.org/web/20220628152326/https://docs.confluent.io/1.0/installation.html?highlight=maven#installation-maven)，我们需要:

```java
<dependency>
    <groupId>io.confluent</groupId>
    <artifactId>kafka-avro-serializer</artifactId>
    <version>4.0.0</version>
</dependency>
```

汇流的串行器在它们的报告中:

```java
<repositories>
    <repository>
        <id>confluent</id>
        <url>https://packages.confluent.io/maven/</url>
    </repository>
</repositories>
```

同样，让我们使用一个 [Maven 插件](https://web.archive.org/web/20220628152326/https://search.maven.org/search?q=g:org.apache.avro%20AND%20a:avro-maven-plugin&core=gav)来生成 Avro 类:

```java
<build>
    <plugins>
        <plugin>
            <groupId>org.apache.avro</groupId>
            <artifactId>avro-maven-plugin</artifactId>
            <version>1.8.2</version>
            <executions>
                <execution>
                    <id>schemas</id>
                    <phase>generate-sources</phase>
                    <goals>
                        <goal>schema</goal>
                        <goal>protocol</goal>
                        <goal>idl-protocol</goal>
                    </goals>
                    <configuration>                        
                        <sourceDirectory>${project.basedir}/src/main/resources/</sourceDirectory>
                        <outputDirectory>${project.basedir}/src/main/java/</outputDirectory>
                    </configuration>
                </execution>
            </executions>
        </plugin>
    </plugins>
</build>
```

为了测试，我们可以使用一个现有的 Kafka 和模式注册表，或者使用一个 [dockerized 合流和 Kafka。](https://web.archive.org/web/20220628152326/https://hub.docker.com/r/confluent/kafka)

## 5.春云流

现在我们已经建立了我们的项目，让我们接下来使用 [Spring Cloud Stream](/web/20220628152326/https://www.baeldung.com/spring-cloud-stream) 编写一个生产者。它会发布某个主题的员工详细信息。

然后，我们将创建一个消费者，它将从主题中读取事件，并将它们写在一个日志语句中。

### 5.1\. (计划或理论的)纲要

首先，让我们为雇员详细信息定义一个模式。我们可以将其命名为`employee-schema.avsc`。

我们可以将模式文件保存在`src/main/resources:`中

```java
{
    "type": "record",
    "name": "Employee",
    "namespace": "com.baeldung.schema",
    "fields": [
    {
        "name": "id",
        "type": "int"
    },
    {
        "name": "firstName",
        "type": "string"
    },
    {
        "name": "lastName",
        "type": "string"
    }]
}
```

创建上述模式后，我们需要构建项目。然后，Apache Avro 代码生成器将在包`com.baeldung.schema`下创建一个名为`Employee`的 POJO。

### 5.2.生产者

春云流提供了`Processor` 接口。这为我们提供了一个输出和输入通道。

让我们用它来制作一个向`employee-details` 卡夫卡主题`:`发送`Employee`对象的生成器

```java
@Autowired
private Processor processor;

public void produceEmployeeDetails(int empId, String firstName, String lastName) {

    // creating employee details
    Employee employee = new Employee();
    employee.setId(empId);
    employee.setFirstName(firstName);
    employee.setLastName(lastName);

    Message<Employee> message = MessageBuilder.withPayload(employee)
                .build();

    processor.output()
        .send(message);
}
```

### 5.2.消费者

现在，让我们写给我们的消费者:

```java
@StreamListener(Processor.INPUT)
public void consumeEmployeeDetails(Employee employeeDetails) {
    logger.info("Let's process employee details: {}", employeeDetails);
}
```

该消费者将阅读发布在`employee-details`主题上的事件。让我们将它的输出指向日志，看看它做了什么。

### 5.3.卡夫卡绑定

到目前为止，我们只处理了我们的`Processor`对象的`input`和`output`通道。这些频道需要配置正确的目的地。

让我们使用`application.yml`来提供 Kafka 绑定:

```java
spring:
  cloud:
    stream: 
      bindings:
        input:
          destination: employee-details
          content-type: application/*+avro
        output:
          destination: employee-details
          content-type: application/*+avro
```

我们应该注意到，在这种情况下，`destination `意味着卡夫卡式的话题。它被称为`destination`可能有点令人困惑，因为在这种情况下它是输入源，但它是消费者和生产者的一致术语。

### 5.4.入口点

现在我们有了生产者和消费者，让我们公开一个 API 来从用户那里获取输入，并将其传递给生产者:

```java
@Autowired
private AvroProducer avroProducer;

@PostMapping("/employees/{id}/{firstName}/{lastName}")
public String producerAvroMessage(@PathVariable int id, @PathVariable String firstName, 
  @PathVariable String lastName) {
    avroProducer.produceEmployeeDetails(id, firstName, lastName);
    return "Sent employee details to consumer";
}
```

### 5.5.启用融合模式注册表和绑定

最后，为了让我们的应用程序同时应用 Kafka 和 schema 注册表绑定，我们需要在我们的一个配置类上添加`@EnableBinding`和`@EnableSchemaRegistryClient`:

```java
@SpringBootApplication
@EnableBinding(Processor.class)
// The @EnableSchemaRegistryClient annotation needs to be uncommented to use the Spring native method.
// @EnableSchemaRegistryClient
public class AvroKafkaApplication {

    public static void main(String[] args) {
        SpringApplication.run(AvroKafkaApplication.class, args);
    }

}
```

我们应该提供一个`ConfluentSchemaRegistryClient` bean:

```java
@Value("${spring.cloud.stream.kafka.binder.producer-properties.schema.registry.url}")
private String endPoint;

@Bean
public SchemaRegistryClient schemaRegistryClient() {
    ConfluentSchemaRegistryClient client = new ConfluentSchemaRegistryClient();
    client.setEndpoint(endPoint);
    return client;
}
```

`endPoint`是合流模式注册中心的 URL。

### 5.6.测试我们的服务

让我们用 POST 请求来测试服务:

```java
curl -X POST localhost:8080/employees/1001/Harry/Potter
```

日志告诉我们这是有效的:

```java
2019-06-11 18:45:45.343  INFO 17036 --- [container-0-C-1] com.baeldung.consumer.AvroConsumer       : Let's process employee details: {"id": 1001, "firstName": "Harry", "lastName": "Potter"}
```

### 5.7.加工过程中发生了什么？

让我们试着理解我们的示例应用程序到底发生了什么:

1.  制片人使用`Employee`对象构建了卡夫卡信息
2.  生产者向模式注册中心注册了雇员模式以获得模式版本 ID，这要么创建一个新的 ID，要么为该确切的模式重用现有的 ID
3.  Avro 使用模式序列化了`Employee` 对象
4.  Spring Cloud 将 schema-id 放在消息头中
5.  该消息在该主题上发布
6.  当消息到达消费者时，它从消息头读取 schema-id
7.  消费者使用 schema-id 从注册表中获取`Employee`模式
8.  消费者找到了一个可以表示该对象的本地类，并将消息反序列化到该类中

## 6.使用原生 Kafka 库的序列化/反序列化

Spring Boot 提供了一些开箱即用的消息转换器。**默认情况下，Spring Boot 使用`Content-Type`报头来选择合适的消息转换器。**

在我们的例子中，`Content-Type`是`application/*+avro,` ，因此它使用`AvroSchemaMessageConverter `来读写 Avro 格式。但是，合流推荐**使用`KafkaAvroSerializer` 和`KafkaAvroDeserializer` 进行消息转换**。

虽然 Spring 自己的格式工作得很好，但它在分区方面有一些缺点，并且它不能与合流标准互操作，而我们的 Kafka 实例上的一些非 Spring 服务可能需要这样。

让我们更新我们的`application.yml`以使用汇流转换器:

```java
spring:
  cloud:
    stream:
      default: 
        producer: 
          useNativeEncoding: true
        consumer:  
          useNativeEncoding: true     
      bindings:
        input:
          destination: employee-details
          content-type: application/*+avro
        output:
          destination: employee-details
          content-type: application/*+avro
      kafka:
         binder:        
           producer-properties:
             key.serializer: io.confluent.kafka.serializers.KafkaAvroSerializer
             value.serializer: io.confluent.kafka.serializers.KafkaAvroSerializer
             schema.registry.url: http://localhost:8081 
           consumer-properties:
             key.deserializer: io.confluent.kafka.serializers.KafkaAvroDeserializer
             value.deserializer: io.confluent.kafka.serializers.KafkaAvroDeserializer
             schema.registry.url: http://localhost:8081
             specific.avro.reader: true 
```

我们已经启用了`useNativeEncoding`。它强制 Spring Cloud Stream 将序列化委托给提供的类。

我们还应该知道如何使用`kafka.binder.producer-properties`和`kafka.binder.consumer-properties.`在 Spring Cloud 中为 Kafka 提供本地设置属性

## 7.消费者群体和分区

**消费者组是属于同一个应用程序**的一组消费者。来自同一消费者组的消费者共享相同的组名。

让我们更新`application.yml`以添加一个消费者组名称:

```java
spring:
  cloud:
    stream:
      // ...     
      bindings:
        input:
          destination: employee-details
          content-type: application/*+avro
          group: group-1
      // ...
```

所有的消费者在他们之间平均分配主题分区。不同分区中的消息将被并行处理。

**在一个消费者组中，一次读取消息的消费者的最大数量等于分区的数量。**因此，我们可以配置分区和消费者的数量，以获得所需的并行性。一般来说，在我们服务的所有副本中，我们应该拥有比消费者总数更多的分区。

### 7.1.分区键

在处理我们的消息时，它们的处理顺序可能很重要。当我们的消息被并行处理时，处理的顺序将很难控制。

Kafka 提供了这样的规则:在给定的分区中，消息总是按照它们到达的顺序进行处理。因此，当某些消息以正确的顺序处理很重要时，我们确保它们到达彼此相同的分区。

我们可以在向主题发送消息时提供分区键。**具有相同分区键的消息将总是进入相同的分区**。如果分区键不存在，消息将以循环方式进行分区。

我们试着用一个例子来理解这一点。假设我们正在接收一个雇员的多条消息，并且我们希望按顺序处理一个雇员的所有消息。部门名称和员工 id 可以唯一地标识员工。

因此，让我们用员工 id 和部门名称来定义分区键:

```java
{
    "type": "record",
    "name": "EmployeeKey",
    "namespace": "com.baeldung.schema",
    "fields": [
     {
        "name": "id",
        "type": "int"
    },
    {
        "name": "departmentName",
        "type": "string"
    }]
}
```

在构建项目之后，`EmployeeKey` POJO 将在包`com.baeldung.schema`下生成。

让我们更新我们的生成器，使用`EmployeeKey` 作为分区键:

```java
public void produceEmployeeDetails(int empId, String firstName, String lastName) {

    // creating employee details
    Employee employee = new Employee();
    employee.setId(empId);
    // ...

    // creating partition key for kafka topic
    EmployeeKey employeeKey = new EmployeeKey();
    employeeKey.setId(empId);
    employeeKey.setDepartmentName("IT");

    Message<Employee> message = MessageBuilder.withPayload(employee)
        .setHeader(KafkaHeaders.MESSAGE_KEY, employeeKey)
        .build();

    processor.output()
        .send(message);
}
```

这里，我们将分区键放在消息头中。

现在，同一个分区将收到具有相同员工 id 和部门名称的消息。

### 7.2。消费者并发

Spring Cloud Stream 允许我们在`application.yml`中为消费者设置并发性:

```java
spring:
  cloud:
    stream:
      // ... 
      bindings:
        input:
          destination: employee-details
          content-type: application/*+avro
          group: group-1
          concurrency: 3
```

现在，我们的消费者将同时阅读来自该主题的三条消息。换句话说，Spring 将产生三个不同的线程来独立消费。

## 8.结论

在本文中，我们用 Avro 模式和合流模式注册中心集成了针对 **Apache Kafka 的生产者和消费者。**

我们在单个应用程序中实现了这一点，但是生产者和消费者可以部署在不同的应用程序中，并且可以拥有他们自己的模式版本，通过注册中心保持同步。

我们看了如何使用 **Spring 的 Avro 和 Schema Registry client 的实现**，然后我们看了如何切换到**的融合标准实现**的序列化和反序列化，以实现互操作性。

最后，我们看了如何划分我们的主题，并确保我们有正确的消息键来支持我们的消息的安全并行处理。

本文使用的完整代码可以在 GitHub 上找到。