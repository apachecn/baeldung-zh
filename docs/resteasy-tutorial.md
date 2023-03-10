# RESTEasy 指南

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/resteasy-tutorial>

## 1。简介

**JAX-RS** (用于 RESTful Web 服务的 Java API)是一组 Java API，为创建**REST API**提供支持。该框架很好地利用了注释来简化这些 API 的开发和部署。

在本教程中，我们将使用 RESTEasy，JBoss 提供的 JAX-RS 规范的可移植实现，以创建一个简单的 RESTful Web 服务。

## 2。项目设置

我们将考虑两种可能的情况:

*   独立安装——适用于所有应用服务器
*   JBoss AS 设置–仅考虑在 JBoss AS 中部署

### 2.1。独立设置

让我们从使用**JBoss wildly 10**的独立设置开始。

JBoss WildFly 10 附带了 rest easy 3 . 0 . 11 版本，但正如你将看到的，我们将用新的 3.0.14 版本配置`pom.xml`。

多亏了`**resteasy-servlet-initializer**`，RESTEasy 通过`ServletContainerInitializer`集成接口提供了与独立`Servlet 3.0`容器的集成。

让我们看看 **`pom.xml`** `:`

```java
<properties>
    <resteasy.version>3.0.14.Final</resteasy.version>
</properties>
<dependencies>
    <dependency>
        <groupId>org.jboss.resteasy</groupId>
        <artifactId>resteasy-servlet-initializer</artifactId>
        <version>${resteasy.version}</version>
    </dependency>
    <dependency>
        <groupId>org.jboss.resteasy</groupId>
        <artifactId>resteasy-client</artifactId>
        <version>${resteasy.version}</version>
    </dependency>
</dependencies> 
```

`**jboss-deployment-structure.xml**`

在 JBoss 中，作为 WAR、JAR 或 EAR 部署的所有东西都是一个模块。这些模块被称为`dynamic modules`。

除了这些，`$JBOSS_HOME/modules`中还有一些静态的 `modules` 。由于 JBoss 有 rest easy`static modules`——对于独立部署，`jboss-deployment-structure.xml`是强制性的，以便排除其中一些。

这样，我们的`WAR`中包含的所有类和`JAR`文件都将被加载:

```java
<jboss-deployment-structure>
    <deployment>
        <exclude-subsystems>
            <subsystem name="resteasy" />
        </exclude-subsystems>
        <exclusions>
            <module name="javaee.api" />
            <module name="javax.ws.rs.api"/>
            <module name="org.jboss.resteasy.resteasy-jaxrs" />
        </exclusions>
        <local-last value="true" />
    </deployment>
</jboss-deployment-structure>
```

### 2.2。JBoss as 设置

如果您打算用 JBoss version 6 或更高版本运行 RESTEasy，您可以选择采用已经捆绑在应用服务器中的库，从而简化 pom:

```java
<dependencies>
    <dependency>
        <groupId>org.jboss.resteasy</groupId>
        <artifactId>resteasy-jaxrs</artifactId>
        <version>${resteasy.version}</version>
    </dependency>
<dependencies> 
```

注意不再需要`jboss-deployment-structure.xml`。

## 3。服务器端代码

### 3.1。Servlet 版本 3

现在让我们快速浏览一下这个简单项目的 web.xml:

```java
<?xml version="1.0" encoding="UTF-8"?>
<web-app version="3.0" 
   xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
   xsi:schemaLocation="http://java.sun.com/xml/ns/javaee 
     http://java.sun.com/xml/ns/javaee/web-app_3_0.xsd">

   <display-name>RestEasy Example</display-name>

   <context-param>
      <param-name>resteasy.servlet.mapping.prefix</param-name>
      <param-value>/rest</param-value>
   </context-param>

</web-app>
```

**`resteasy.servlet.mapping.prefix`** 只有在你想给 API 应用预设一个相对路径的时候才需要。

此时，需要注意的是，我们没有在 `**web.xml**`中声明任何`Servlet` ，因为 **`resteasy`** **`servlet-initializer`** 已经在 **`pom.xml`** 中被添加为依赖项。原因是–rest easy 提供了实现`javax.server.ServletContainerInitializer`的`org.jboss.resteasy.plugins.servlet.ResteasyServletInitializer`类。

是一个初始化器，它在任何 servlet 上下文准备好之前执行——你可以使用这个初始化器为你的应用定义 servlet、过滤器或监听器。

### 3.2。应用程序类

`javax.ws.rs.core.Application`类是一个标准的 JAX-RS 类，您可以实现它来提供关于您的部署的信息:

```java
@ApplicationPath("/rest")
public class RestEasyServices extends Application {

    private Set<Object> singletons = new HashSet<Object>();

    public RestEasyServices() {
        singletons.add(new MovieCrudService());
    }

    @Override
    public Set<Object> getSingletons() {
        return singletons;
    }
}
```

正如你所看到的——这只是一个列出了所有 JAX-RS 根资源和提供者的类，并且使用了`@ApplicationPath`注释。

如果为 by 类和 singletons 返回任何空集，将扫描 WAR 以查找 JAX-RS 注释资源和提供者类。

### 3.3。服务实现类

最后，让我们在这里看一个实际的 API 定义:

```java
@Path("/movies")
public class MovieCrudService {

    private Map<String, Movie> inventory = new HashMap<String, Movie>();

    @GET
    @Path("/getinfo")
    @Produces({ MediaType.APPLICATION_JSON, MediaType.APPLICATION_XML })
    public Movie movieByImdbId(@QueryParam("imdbId") String imdbId) {
        if (inventory.containsKey(imdbId)) {
            return inventory.get(imdbId);
        } else {
            return null;
        }
    }

    @POST
    @Path("/addmovie")
    @Consumes({ MediaType.APPLICATION_JSON, MediaType.APPLICATION_XML })
    public Response addMovie(Movie movie) {
        if (null != inventory.get(movie.getImdbId())) {
            return Response
              .status(Response.Status.NOT_MODIFIED)
              .entity("Movie is Already in the database.").build();
        }

        inventory.put(movie.getImdbId(), movie);
        return Response.status(Response.Status.CREATED).build();
    }
}
```

## 4。结论

在这个快速教程中，我们介绍了 RESTEasy，并用它构建了一个超级简单的 API。

本文中使用的示例在 GitHub 中作为[示例项目提供。](https://web.archive.org/web/20221208143837/https://github.com/eugenp/tutorials/tree/master/web-modules/resteasy)