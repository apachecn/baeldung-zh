# Hibernate 多对多注释教程

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/hibernate-many-to-many>

## 1。简介

在这个快速教程中，我们将快速看一下如何在 Hibernate 中使用`@ManyToMany`注释来指定这种类型的关系。

## 2。典型的例子

让我们从一个简单的实体关系图开始——它显示了两个实体`employee` 和`project:`之间的多对多关联

[![New 300x59](img/95dc22ed67f1f0a9b48e3cb50dbdafc3.png)](/web/20220908192458/https://www.baeldung.com/wp-content/uploads/2017/09/New.png)

在这个场景中，任何给定的`employee` 都可以被分配给多个项目，而一个`project` 可能有多个员工为其工作，这导致了两者之间的多对多关联。

我们有一个以`employee_id`为主键的`employee` 表和一个以`project_id` 为主键的`project`表。这里需要一个连接表`employee_project` 来连接双方。

## 3。数据库设置

假设我们已经创建了一个名为`spring_hibernate_many_to_many.`的数据库

我们还需要创建`employee`和`project`表以及将`employee_id`和`project_id`作为外键的`employee_project`连接表:

```java
CREATE TABLE `employee` (
  `employee_id` int(11) NOT NULL AUTO_INCREMENT,
  `first_name` varchar(50) DEFAULT NULL,
  `last_name` varchar(50) DEFAULT NULL,
  PRIMARY KEY (`employee_id`)
) ENGINE=InnoDB AUTO_INCREMENT=17 DEFAULT CHARSET=utf8;

CREATE TABLE `project` (
  `project_id` int(11) NOT NULL AUTO_INCREMENT,
  `title` varchar(50) DEFAULT NULL,
  PRIMARY KEY (`project_id`)
) ENGINE=InnoDB AUTO_INCREMENT=18 DEFAULT CHARSET=utf8;

CREATE TABLE `employee_project` (
  `employee_id` int(11) NOT NULL,
  `project_id` int(11) NOT NULL,
  PRIMARY KEY (`employee_id`,`project_id`),
  KEY `project_id` (`project_id`),
  CONSTRAINT `employee_project_ibfk_1` 
   FOREIGN KEY (`employee_id`) REFERENCES `employee` (`employee_id`),
  CONSTRAINT `employee_project_ibfk_2` 
   FOREIGN KEY (`project_id`) REFERENCES `project` (`project_id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8; 
```

随着数据库的建立，下一步将是准备 Maven 依赖项和 Hibernate 配置。关于这方面的信息，请参考关于[用春天冬眠指南 4](/web/20220908192458/https://www.baeldung.com/hibernate-4-spring)的文章

## 4。模型类

模型类`Employee` 和`Project` 需要用 JPA 注释创建:

```java
@Entity
@Table(name = "Employee")
public class Employee { 
    // ...

    @ManyToMany(cascade = { CascadeType.ALL })
    @JoinTable(
        name = "Employee_Project", 
        joinColumns = { @JoinColumn(name = "employee_id") }, 
        inverseJoinColumns = { @JoinColumn(name = "project_id") }
    )
    Set<Project> projects = new HashSet<>();

    // standard constructor/getters/setters
}
```

```java
@Entity
@Table(name = "Project")
public class Project {    
    // ...  

    @ManyToMany(mappedBy = "projects")
    private Set<Employee> employees = new HashSet<>();

    // standard constructors/getters/setters   
}
```

正如我们所看到的，**`Employee`类和`Project` 类相互引用，这意味着它们之间的关联是双向的。**

为了映射多对多的关联，我们使用了`@ManyToMany`、 `@JoinTable`和`@JoinColumn`注释。让我们仔细看看它们。

两个类中都使用了`@ManyToMany`注释来创建实体之间的多对多关系。

**这种关联有两个方面，即拥有方和相反方。**在我们的例子中，拥有方是`Employee` ，所以通过使用`Employee` 类中的`@JoinTable`注释在拥有方指定连接表。`@JoinTable`用于定义连接/链接表。在这种情况下，它就是`Employee_Project.`

`@JoinColumn` 注释用于指定主表的连接/链接列。在这里，连接列是`employee_id` ，而`project_id` 是反向连接列，因为`Project` 位于关系的反向。

在`Project` 类中，`mappedBy` 属性被用在`@ManyToMany` 注释中，表示`employees` 集合被所有者端的`projects` 集合映射。

## 5。执行

为了查看多对多注释的运行情况，我们可以编写以下 JUnit 测试:

```java
public class HibernateManyToManyAnnotationMainIntegrationTest {
	private static SessionFactory sessionFactory;
	private Session session;

	//...

	@Test
        public void givenSession_whenRead_thenReturnsMtoMdata() {
	    prepareData();
       	    @SuppressWarnings("unchecked")
	    List<Employee> employeeList = session.createQuery("FROM Employee").list();
            @SuppressWarnings("unchecked")
	    List<Project> projectList = session.createQuery("FROM Project").list();
            assertNotNull(employeeList);
            assertNotNull(projectList);
            assertEquals(2, employeeList.size());
            assertEquals(2, projectList.size());

            for(Employee employee : employeeList) {
               assertNotNull(employee.getProjects());
               assertEquals(2, employee.getProjects().size());
            }
            for(Project project : projectList) {
               assertNotNull(project.getEmployees());
               assertEquals(2, project.getEmployees().size());
            }
        }

	private void prepareData() {
	    String[] employeeData = { "Peter Oven", "Allan Norman" };
	    String[] projectData = { "IT Project", "Networking Project" };
	    Set<Project> projects = new HashSet<Project>();

	    for (String proj : projectData) {
		projects.add(new Project(proj));
	    }

	    for (String emp : employeeData) {
		Employee employee = new Employee(emp.split(" ")[0], emp.split(" ")[1]);
		employee.setProjects(projects);

	        for (Project proj : projects) {
		    proj.getEmployees().add(employee);
		}

		session.persist(employee);
	    }
	}

	//...
}
```

我们可以看到在数据库中创建的两个实体之间的多对多关系:`employee`、`project`和`employee_project`表，其中的示例数据表示这种关系。

## 6。结论

在本教程中，我们看到了如何使用 Hibernate 的多对多注释创建映射，与创建 XML 映射文件相比，这是一种更方便的对应方式。

本教程的源代码可以在 GitHub 上找到[。](https://web.archive.org/web/20220908192458/https://github.com/eugenp/tutorials/tree/master/persistence-modules/hibernate-mapping-2)