# 参考 pom.xml 中的环境变量

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/maven-env-variables>

## 1.概观

在这个快速教程中，我们将看到如何从 [Maven](/web/20220726094420/https://www.baeldung.com/maven) 的`pom.xml`中读取环境变量来定制构建过程。

## 2.环境变量

**要引用来自`pom.xml`的环境变量，我们可以使用`${env.VARIABLE_NAME} `语法**。

例如，让我们在构建过程中使用它来[具体化 Java 版本](/web/20220726094420/https://www.baeldung.com/maven-java-version):

```
<build>
    <plugins>
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-compiler-plugin</artifactId>
            <version>3.8.1</version>
            <configuration>
                <source>${env.JAVA_VERSION}</source>
                <target>${env.JAVA_VERSION}</target>
            </configuration>
        </plugin>
    </plugins>
</build>
```

我们应该记住通过环境变量传递 Java 版本信息。如果我们做不到这一点，那么我们将无法建立这个项目。

要针对这样的构建文件运行 Maven 目标或阶段，我们应该首先导出环境变量。例如:

```
$ export JAVA_VERSION=9
$ mvn clean package
```

在 Windows 上，我们应该使用`“` `set VAR=value” `语法来导出环境变量。

**为了在`JAVA_VERSION `环境变量丢失时提供默认值，我们可以使用一个 Maven 概要文件**:

```
<profiles>
    <profile>
        <id>default-java</id>
        <activation>
            <property>
                <name>!env.JAVA_VERSION</name>
            </property>
        </activation>
        <build>
            <plugins>
                <plugin>
                    <groupId>org.apache.maven.plugins</groupId>
                    <artifactId>maven-compiler-plugin</artifactId>
                    <version>3.8.1</version>
                    <configuration>
                        <source>1.8</source>
                        <target>1.8</target>
                    </configuration>
                </plugin>
            </plugins>
        </build>
    </profile>
</profiles>
```

如上所示，我们正在创建一个概要文件，并且只有当`JAVA_VERSION `环境变量缺失时才激活它。如果发生这种情况，那么这个新的插件定义将覆盖现有的插件定义。

## 3.结论

在这个简短的教程中，我们看到了如何通过向`pom.xml`传递环境变量来定制构建过程。