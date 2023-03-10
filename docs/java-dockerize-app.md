# 将 Java 应用程序归档

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-dockerize-app>

## 1.概观

在本文中，我们将展示如何**对接一个基于 Java runnable jar 的应用程序**。请务必阅读使用 Docker 的[好处。](https://web.archive.org/web/20220919052435/https://apiumhub.com/tech-blog-barcelona/top-benefits-using-docker/)

## 2。构建可运行的 Jar

我们将使用 Maven 来构建一个可运行的 jar。

因此，我们的应用程序有一个简单的类`HelloWorld.java`，带有一个`main`方法:

```java
public class HelloWorld {
    public static void main(String[] args){
        System.out.println("Welcome to our application");
    }
}
```

我们使用`maven-jar-plugin`来生成一个可运行的 jar:

```java
<plugin>
   <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-jar-plugin</artifactId>
     <version>${maven-jar-plugin.version}</version>             
      <configuration>
          <archive>
             <manifest>
              <mainClass>com.baeldung.HelloWorld</mainClass>
              </manifest>
           </archive>
       </configuration>
</plugin>
```

## 3.编写 Dockerfile 文件

让我们在`Dockerfile`中编写 Dockerize 我们的可运行 jar 的步骤。`Dockerfile`驻留在`build context`的根目录下:

```java
FROM openjdk:11
MAINTAINER baeldung.com
COPY target/docker-java-jar-0.0.1-SNAPSHOT.jar app.jar
ENTRYPOINT ["java","-jar","/app.jar"]
```

这里，在第一行中，我们从官方存储库中导入 OpenJDK Java version 11 映像作为我们的基础映像。随后的线**将在我们前进**时在这个基础图像上创建额外的层。

在第二行中，我们指定了图像的维护者，在本例中是`baeldung.com`。这一步不会创建任何附加层。

在第三行中，我们通过将生成的 jar`docker-java-jar-0.0.1-SNAPSHOT.jar`从`build context`的`target`文件夹复制到名为`app.jar`的容器的`root`文件夹中来创建一个新层。

在最后一行，**,我们用统一的命令指定主应用程序，该命令为这个图像**执行。在这种情况下，我们使用`java -jar`命令告诉容器运行`app.jar`。此外，这一行没有引入任何额外的层。

## 4.构建和测试映像

现在我们有了`Dockerfile`，让我们使用 Maven 来构建和打包我们的可运行 jar:

```java
mvn package
```

之后，让我们建立我们的 Docker 形象:

```java
docker image build -t docker-java-jar:latest .
```

这里，我们使用`-t`标志来指定一个**名称和标签，格式为`<name>:<tag>`**。在这种情况下，`docker-java-jar`是我们的图像名称，标签是`latest`。的“.”象征着我们的`Dockerfile`居住的道路。在本例中，它只是当前目录。

注意:我们可以用相同的名称和不同的标签构建不同的 Docker 图像。

最后，让我们从命令行运行 Docker 映像:

```java
docker run docker-java-jar:latest
```

上面的命令运行我们的 Docker 图像，通过名称和标签以`<name>:<tag>`格式识别它。

## 4.结论

在本文中，我们已经看到了对可运行的 Java jar 进行 Dockerizing 的步骤。本文中使用的代码示例可以从 GitHub 上的[处获得。](https://web.archive.org/web/20220919052435/https://github.com/eugenp/tutorials/tree/master/docker-modules/docker-java-jar)