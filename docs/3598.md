# 春季的 MaxUploadSizeExceededException

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-maxuploadsizeexceeded>

## 1。概述

在 Spring 框架中，当应用程序试图上传一个大小超过配置中指定的某个阈值的文件时，会抛出一个`MaxUploadSizeExceededException`。

在本教程中，我们将看看如何指定最大上传大小。然后我们将展示一个简单的文件上传控制器，并讨论处理这个异常的不同方法。

## 2。设置最大上传大小

默认情况下，可以上传的文件大小没有限制。为了设置最大上传大小，您必须声明一个类型为`MultipartResolver`的 bean。

让我们看一个将文件大小限制为 5 MB 的示例:

```
@Bean
public MultipartResolver multipartResolver() {
    CommonsMultipartResolver multipartResolver
      = new CommonsMultipartResolver();
    multipartResolver.setMaxUploadSize(5242880);
    return multipartResolver;
}
```

## 3。文件上传控制器

接下来，让我们定义一个控制器方法来处理文件的上传和保存到服务器:

```
@RequestMapping(value = "/uploadFile", method = RequestMethod.POST)
public ModelAndView uploadFile(MultipartFile file) throws IOException {

    ModelAndView modelAndView = new ModelAndView("file");
    InputStream in = file.getInputStream();
    File currDir = new File(".");
    String path = currDir.getAbsolutePath();
    FileOutputStream f = new FileOutputStream(
      path.substring(0, path.length()-1)+ file.getOriginalFilename());
    int ch = 0;
    while ((ch = in.read()) != -1) {
        f.write(ch);
    }

    f.flush();
    f.close();

    modelAndView.getModel().put("message", "File uploaded successfully!");
    return modelAndView;
}
```

如果用户试图上传大于 5 MB 的文件，应用程序将抛出类型为`MaxUploadSizeExceededException`的异常。

## 4。处理`MaxUploadSizeExceededException`

为了处理这个异常，我们可以让我们的控制器实现接口`HandlerExceptionResolver`，或者我们可以创建一个`@ControllerAdvice`注释类。

### 4.1。实施`HandlerExceptionResolver`

`HandlerExceptionResolver`接口声明了一个名为 `resolveException()`的方法，其中可以处理不同类型的异常。

让我们重写`resolveException()`方法，以便在捕获的异常是类型`MaxUploadSizeExceededException`的情况下显示一条消息:

```
@Override
public ModelAndView resolveException(
  HttpServletRequest request,
  HttpServletResponse response, 
  Object object,
  Exception exc) {   

    ModelAndView modelAndView = new ModelAndView("file");
    if (exc instanceof MaxUploadSizeExceededException) {
        modelAndView.getModel().put("message", "File size exceeds limit!");
    }
    return modelAndView;
}
```

### 4.2。创建控制器建议拦截器

通过拦截器而不是在控制器本身中处理异常有几个优点。一个是我们可以对多个控制器应用相同的异常处理逻辑。

另一个是我们可以创建一个方法，只针对我们想要处理的异常，允许框架委托异常处理，而不必使用`instanceof`来检查抛出了什么类型的异常:

```
@ControllerAdvice
public class FileUploadExceptionAdvice {

    @ExceptionHandler(MaxUploadSizeExceededException.class)
    public ModelAndView handleMaxSizeException(
      MaxUploadSizeExceededException exc, 
      HttpServletRequest request,
      HttpServletResponse response) {

        ModelAndView modelAndView = new ModelAndView("file");
        modelAndView.getModel().put("message", "File too large!");
        return modelAndView;
    }
}
```

## 5。Tomcat 配置

如果您正在部署 Tomcat server 版本 7 及更高版本，那么您可能需要设置或更改一个名为`maxSwallowSize`的配置属性。

该属性指定当 Tomcat 知道服务器将忽略该文件时，它将从客户端“吞下”上载的最大字节数。

该属性的默认值是 2097152 (2 MB)。如果保持不变或者设置低于我们在`MultipartResolver`中设置的 5 MB 限制，Tomcat 将拒绝任何上传超过 2 MB 的文件的尝试，并且我们的自定义异常处理将永远不会被调用。

为了使请求成功并显示来自应用程序的错误消息，您需要将`maxSwallowSize`属性设置为负值。这指示 Tomcat 忽略所有失败的上传，而不管文件大小。

这是在`TOMCAT_HOME/conf/server.xml`文件中完成的:

```
<Connector port="8080" protocol="HTTP/1.1"
  connectionTimeout="20000"
  redirectPort="8443" 
  maxSwallowSize = "-1"/>
```

## 6。结论

在本文中，我们展示了如何在 Spring 中配置最大文件上传大小，以及如何处理当客户端试图上传超过这个大小限制的文件时产生的`MaxUploadSizeExceededException`。

本文的完整源代码可以在 [GitHub 项目](https://web.archive.org/web/20220629005111/https://github.com/eugenp/tutorials/tree/master/spring-web-modules/spring-mvc-forms-jsp)中找到。