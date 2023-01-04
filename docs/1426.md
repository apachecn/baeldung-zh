# 一个使用 Servlets 和 JSP 的 MVC 例子

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/mvc-servlet-jsp>

## 1。概述

在这篇简短的文章中，我们将使用基本的 Servlets 和 JSP 创建一个实现模型视图控制器(MVC)设计模式的小型 web 应用程序。

在我们继续实现之前，我们将稍微探讨一下 MVC 是如何工作的，以及它的关键特性。

## 2。MVC 简介

模型-视图-控制器(MVC)是软件工程中使用的一种模式，用于将应用程序逻辑与用户界面分离开来。顾名思义，MVC 模式有三层。

**模型定义应用的业务层，控制器管理应用的流程，视图定义应用的表示层。**

尽管 MVC 模式并不特定于 web 应用程序，但它非常适合这种类型的应用程序。在 Java 上下文中，模型由简单的 Java 类组成，控制器由 servlets 组成，视图由 JSP 页面组成。

以下是该模式的一些关键特征:

*   它将表示层与业务层分离开来
*   控制器执行调用模型和发送数据到视图的动作
*   该模型甚至不知道它被某个 web 应用程序或桌面应用程序使用

让我们看看每一层。

### 2.1。模型层

这是数据层，包含系统的业务逻辑，也表示应用程序的状态。

它独立于表示层，控制器从模型层获取数据并将其发送到视图层。

### 2.2。控制器层

控制器层充当视图和模型之间的接口。它从视图层接收请求并处理它们，包括必要的验证。

请求被进一步发送到模型层进行数据处理，一旦它们被处理，数据被发送回控制器，然后显示在视图上。

### 2.3。视图层

这一层表示应用程序的输出，通常是某种形式的 UI。表示层用于显示控制器获取的模型数据。

## 3。带有 Servlets 和 JSP 的 MVC

为了实现基于 MVC 设计模式的 web 应用程序，我们将创建`Student`和`StudentService`类——它们将充当我们的模型层。

S `tudentServlet` 类将充当控制器，对于表示层，我们将创建`student-record.jsp` 页面。

现在，让我们一个一个地写这些层，从`Student`类开始:

```
public class Student {
    private int id;
    private String firstName;
    private String lastName;

    // constructors, getters and setters goes here
} 
```

现在让我们编写我们的`StudentService`，它将处理我们的业务逻辑:

```
public class StudentService {

    public Optional<Student> getStudent(int id) {
        switch (id) {
            case 1:
                return Optional.of(new Student(1, "John", "Doe"));
            case 2:
                return Optional.of(new Student(2, "Jane", "Goodall"));
            case 3:
                return Optional.of(new Student(3, "Max", "Born"));
            default:
                return Optional.empty();
        }
    }
}
```

现在让我们创建我们的控制器类`StudentServlet`:

```
@WebServlet(
  name = "StudentServlet", 
  urlPatterns = "/student-record")
public class StudentServlet extends HttpServlet {

    private StudentService studentService = new StudentService();

    private void processRequest(
      HttpServletRequest request, HttpServletResponse response) 
      throws ServletException, IOException {

        String studentID = request.getParameter("id");
        if (studentID != null) {
            int id = Integer.parseInt(studentID);
            studentService.getStudent(id)
              .ifPresent(s -> request.setAttribute("studentRecord", s));
        }

        RequestDispatcher dispatcher = request.getRequestDispatcher(
          "/WEB-INF/jsp/student-record.jsp");
        dispatcher.forward(request, response);
    }

    @Override
    protected void doGet(
      HttpServletRequest request, HttpServletResponse response) 
      throws ServletException, IOException {

        processRequest(request, response);
    }

    @Override
    protected void doPost(
      HttpServletRequest request, HttpServletResponse response) 
      throws ServletException, IOException {

        processRequest(request, response);
    }
}
```

这个 servlet 是我们的 web 应用程序的控制器。

首先，它从请求中读取一个参数`id` 。如果提交了`id`，则从业务层获取一个`Student`对象。

一旦从模型中检索到必要的数据，它就使用`setAttribute()`方法将这些数据放入请求中。

最后，控制器`forwards` 将请求和响应对象传递给一个 JSP，即应用程序的视图。

接下来，让我们编写我们的表示层`student-record.jsp`:

```
<html>
    <head>
        <title>Student Record</title>
    </head>
    <body>
    <% 
        if (request.getAttribute("studentRecord") != null) {
            Student student = (Student) request.getAttribute("studentRecord");
    %>

    <h1>Student Record</h1>
    <div>ID: <%= student.getId()%></div>
    <div>First Name: <%= student.getFirstName()%></div>
    <div>Last Name: <%= student.getLastName()%></div>

    <% 
        } else { 
    %>

    <h1>No student record found.</h1>

    <% } %>	
    </body>
</html>
```

当然，JSP 是应用程序的视图；它从控制器接收所有需要的信息，不需要直接与业务层交互。

## 4。结论

在本教程中，我们已经了解了 MVC，即模型视图控制器架构，我们重点关注如何实现一个简单的例子。

像往常一样，这里展示的代码可以在 GitHub 上找到[。](https://web.archive.org/web/20221023203132/https://github.com/eugenp/tutorials/tree/master/web-modules/javax-servlets)