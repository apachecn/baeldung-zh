# 设置和运行 MySQL 容器

> 原文::1230【https://web . archive . org/web/202209930061024/https://www . BAE message . com/ops/docker-mysql-container

## 1.概观

容器是 IT 行业最热门的话题，因为它有很多优点。企业正以惊人的速度采用基于容器的解决方案。[根据 451 Research](https://web.archive.org/web/20220928150307/https://451research.com/451-research-says-application-containers-market-will-grow-to-reach-4-3bn-by-2022) 的研究，应用程序容器市场在未来几年将增长四倍。

今天，我们甚至有像 MySQL、 [MongoDB](/web/20220928150307/https://www.baeldung.com/linux/mongodb-as-docker-container) 、 [PostgreSQL](/web/20220928150307/https://www.baeldung.com/ops/postgresql-docker-setup) 这样的数据库，以及更多容器形式的数据库。然而，本文将探索设置和运行 MySQL 容器的选项。首先，我们将备份现有的 MySQL 数据库。接下来，我们将构建一个 YAML 形式的容器配置，并使用`docker-compose`运行它，这是一个用于将一堆应用程序容器放在一起的开源工具包。

事不宜迟，让我们进入它的本质细节。

## 2.构建 MySQL 容器配置

在这一节中，我们将使用`docker-compose`工具构建 MySQL 容器。但是，YAML 也使用 docker 文件中的映像作为当前路径中的基本配置。

### 2.1 .复合坞站

首先，让我们创建带有`version`和`services`标签的 YAML 文件。我们在 YAML 文件的`version`标签下定义文件格式版本。MySQL 服务使用 Dockerfile 中的图像信息，这是我们在上下文中定义的。

此外，**我们还指示工具使用在`.env`** 文件中定义为环境变量的默认参数。最后，`ports`标签将绑定容器和主机端口 3306。让我们看看我们用来启动 MySQL 服务的`docker-compose` YAML 文件的内容:

```
# cat docker-compose.yml
version: '3.3'
services:
### MySQL Container
  mysql:
    build:
      context: /home/tools/bael/dung/B015
      args:
        - MYSQL_DATABASE=${MYSQL_DATABASE}
        - MYSQL_USER=${MYSQL_USER}
        - MYSQL_PASSWORD=${MYSQL_PASSWORD}
        - MYSQL_ROOT_PASSWORD=${MYSQL_ROOT_PASSWORD}
    ports:
      - "${MYSQL_PORT}:3306"
```

### 2.2.Dockerfile 文件创建

在内部，`docker-compose`使用指定路径中的 Dockerfile 构建镜像并为 MySQL 设置环境。我们的 Dockerfile 从 DockerHub 下载图像，并用定义的变量旋转容器:

```
# cat Dockerfile
FROM mysql:latest

MAINTAINER baeldung.com

RUN chown -R mysql:root /var/lib/mysql/

ARG MYSQL_DATABASE
ARG MYSQL_USER
ARG MYSQL_PASSWORD
ARG MYSQL_ROOT_PASSWORD

ENV MYSQL_DATABASE=$MYSQL_DATABASE
ENV MYSQL_USER=$MYSQL_USER
ENV MYSQL_PASSWORD=$MYSQL_PASSWORD
ENV MYSQL_ROOT_PASSWORD=$MYSQL_ROOT_PASSWORD

ADD data.sql /etc/mysql/data.sql

RUN sed -i 's/MYSQL_DATABASE/'$MYSQL_DATABASE'/g' /etc/mysql/data.sql
RUN cp /etc/mysql/data.sql /docker-entrypoint-initdb.d

EXPOSE 3306
```

现在，让我们看一下下面 Dockerfile 文件片段中给出的所有指令:

*   `FROM`–有效的 Dockerfile 以`FROM` 语句开始，它描述了图像名称和`version`标签。在我们的例子中，我们使用带有`latest`标签的`mysql`图像。
*   `MAINTAINER`–将作者信息设置为容器的元数据，通过`docker inspect`可见。
*   `RUN`–在`mysql`图像的顶部执行命令，随后形成一个新层。提交结果映像，并将其用于 docker 文件中定义的后续步骤。
*   `ARG`–在构建期间传递变量。这里，我们传递四个用户变量作为构建参数。
*   `ENV`–我们使用`$`符号来表示 Dockerfile 中的环境变量。在上面的代码片段中，我们使用了四个变量。
*   `ADD`–在构建期间，它会将文件添加到容器中以备将来使用。
*   `EXPOSE`–使服务在 Docker 容器之外可用。

### 2.3.设置环境

此外，我们可以在当前路径中创建一个环境变量文件`.env`。该文件包含合成文件中涉及的所有变量:

```
# cat .env
MYSQL_DATABASE=my_db_name
MYSQL_USER=baeldung
MYSQL_PASSWORD=pass
MYSQL_ROOT_PASSWORD=pass
MYSQL_PORT=3306
```

### 2.4.MySQL 备份文件

为了便于演示，让我们从现有的数据库表中进行备份。这里，我们通过`data.sql`文件将同一个`Customers`表自动导入到 MySQL 容器中。

下面，我们展示了使用`SELECT`查询的表数据，该查询从请求的表中获取数据:

```
mysql> select * from Customers;
+--------------+-----------------+---------------+-----------+------------+---------+
| CustomerName | ContactName     | Address       | City      | PostalCode | Country |
+--------------+-----------------+---------------+-----------+------------+---------+
| Cardinal     | Tom B. Erichsen | Skagen 21     | Stavanger | 4006       | Norway  |
| Wilman Kala  | Matti Karttunen | Keskuskatu 45 | Helsinki  | 21240      | Finland |
+--------------+-----------------+---------------+-----------+------------+---------+
2 rows in set (0.00 sec)
```

作为 MySQL RDBMS 包的一部分，`mysqldump`实用程序用于将数据库中的所有数据备份到一个文本文件中。使用带有内联参数的简单命令，我们可以快速备份 MySQL 表:

*   -u: MySQL 用户名
*   -p: MySQL 密码

```
# mysqldump -u [user name] –p [password] [database_name] > [dumpfilename.sql]

# mysqldump -u root -p my_db_name > data.sql
Enter password:
```

在高级别上，备份文件将删除所选数据库中任何名为`Customers`的表，并将所有备份的数据插入其中:

```
# cat data.sql
-- MySQL dump 10.13  Distrib 8.0.26, for Linux (x86_64)
...
... output truncated ...
...
DROP TABLE IF EXISTS `Customers`;
/*!40101 SET @saved_cs_client     = @@character_set_client */;
/*!50503 SET character_set_client = utf8mb4 */;
CREATE TABLE `Customers` (
  `CustomerName` varchar(255) DEFAULT NULL,
...
... output truncated ...
...
INSERT INTO `Customers` VALUES ('Cardinal','Tom B. Erichsen','Skagen 21','Stavanger','4006','Norway'),('Wilman Kala','Matti Karttunen','Keskuskatu 45','Helsinki','21240','Finland');
/*!40000 ALTER TABLE `Customers` ENABLE KEYS */;
UNLOCK TABLES;
...
... output truncated ...
...
-- Dump completed on 2022-07-28  1:56:09
```

但是，数据库的创建或删除不是在创建的转储文件中管理的。我们将在`data.sql`文件中添加下面的代码片段，如果数据库不存在，它将创建数据库。它通过管理数据库和表来完成这个循环。最后，它还通过`USE`命令使用创建的数据库:

```
--
-- Create a database using `MYSQL_DATABASE` placeholder
--
CREATE DATABASE IF NOT EXISTS `MYSQL_DATABASE`;
USE `MYSQL_DATABASE`;
```

目前，目录结构如下所示:

```
# tree -a
.
├── data.sql
├── docker-compose.yml
├── Dockerfile
└── .env
```

## 3.旋转 MySQL 服务器容器

现在，我们已经准备好通过`docker-compose`旋转一个容器。为了打开 MySQL 容器，我们需要执行`docker-compose up`。

当我们浏览输出行时，**我们可以看到它们在 MySQL 映像**的每一步中都形成了新的层。

随后，它还创建数据库并加载在`data.sql`文件中指定的数据:

```
# docker-compose up
Building mysql
Sending build context to Docker daemon  7.168kB
Step 1/15 : FROM mysql:latest
 ---> c60d96bd2b77
Step 2/15 : MAINTAINER baeldung.com
 ---> Running in a647bd02b91f
Removing intermediate container a647bd02b91f
 ---> fafa500c0fac
Step 3/15 : RUN chown -R mysql:root /var/lib/mysql/
 ---> Running in b37e1d5ba079

...
... output truncated ...
...

Step 14/15 : RUN cp /etc/mysql/data.sql /docker-entrypoint-initdb.d
 ---> Running in 34f1d9807bad
Removing intermediate container 34f1d9807bad
 ---> 927b68a43976
Step 15/15 : EXPOSE 3306
 ---> Running in defb868f4207
Removing intermediate container defb868f4207
 ---> 6c6f435f52a9
Successfully built 6c6f435f52a9
Successfully tagged b015_mysql:latest
Creating b015_mysql_1 ... done
Attaching to b015_mysql_1
mysql_1  | 2022-07-28 00:49:03+00:00 [Note] [Entrypoint]: Entrypoint script for MySQL Server 8.0.26-1debian10 started.

...
... output truncated ...
...

mysql_1  | 2022-07-28 00:49:16+00:00 [Note] [Entrypoint]: Creating database my_db_name
mysql_1  | 2022-07-28 00:49:16+00:00 [Note] [Entrypoint]: Creating user baeldung
mysql_1  | 2022-07-28 00:49:16+00:00 [Note] [Entrypoint]: Giving user baeldung access to schema my_db_name
mysql_1  |
mysql_1  | 2022-07-28 00:49:16+00:00 [Note] [Entrypoint]: /usr/local/bin/docker-entrypoint.sh: running /docker-entrypoint-initdb.d/data.sql
...
... output truncated ...
...
```

**我们可以使用`-d`选项在分离模式下运行容器**:

```
# docker-compose up -d
Building mysql
Sending build context to Docker daemon  7.168kB
Step 1/15 : FROM mysql:latest
 ---> c60d96bd2b77
...
... output truncated ...
...
Step 15/15 : EXPOSE 3306
 ---> Running in 958e1d4af340
Removing intermediate container 958e1d4af340
 ---> c3516657c4c8
Successfully built c3516657c4c8
Successfully tagged b015_mysql:latest
Creating b015_mysql_1 ... done
#
```

## 4.MySQL 客户端就绪

必须安装一个[客户端](/web/20220928150307/https://www.baeldung.com/linux/mysql-client-utilities)才能轻松访问 MySQL 服务器。根据我们的需要，我们可以将客户端安装在主机或任何其他与服务器容器具有 IP 可达性的机器或容器上:

```
$ sudo apt install mysql-client -y
Reading package lists... Done
Building dependency tree
Reading state information... Done
mysql-client is already the newest version (5.7.37-0ubuntu0.18.04.1).
...
... output truncated ...
...
```

现在，让我们提取 MySQL 客户端的安装路径和版本:

```
$ which mysql
/usr/bin/mysql
$ mysql --version
mysql  Ver 14.14 Distrib 5.7.37, for Linux (x86_64) using  EditLine wrapper
```

## 5.服务器客户端通信

我们可以使用客户端应用程序访问部署的 MySQL 服务器。在这一节中，我们将看到如何通过客户端访问 MySQL 服务器。

让我们使用`docker ps`命令查看创建的容器 id 和状态:

```
# docker ps | grep b015_mysql
9ce4da8eb682   b015_mysql                "docker-entrypoint.s…"   21 minutes ago   Up 21 minutes         0.0.0.0:3306->3306/tcp, :::3306->3306/tcp, 33060/tcp                                                                    b015_mysql_1
```

接下来，让我们使用安装的客户机服务获取容器 IP 地址来访问数据库。**如果我们发出`docker inspect`命令，我们将看到 JSON 格式的容器的详细信息。**我们也可以从生成的 JSON 中选择任何字段。这里，我们从`range.NetworkSettings.Networks -> IPAddress`获取 IP 地址:

```
# docker inspect -f '{{range.NetworkSettings.Networks}}{{.IPAddress}}{{end}}' 9ce4da8eb682
172.19.0.2
```

然后，我们可以使用配置的主机和端口信息，通过客户端登录 MySQL 服务器:

```
# mysql -h 172.17.0.2 -P 3306 --protocol=tcp -u root -p
Enter password:
Welcome to the MySQL monitor.  Commands end with ; or \g.
...
... output truncated ...
...
mysql> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| my_db_name         |
| mysql              |
| performance_schema |
| sys                |
+--------------------+
5 rows in set (0.00 sec)
mysql> use my_db_name
...
... output truncated ...
...

Database changed 
```

在这里，我们可以看到数据是从`data.sql`文件中自动恢复的:

```
mysql> select * from Customers;
+--------------+-----------------+---------------+-----------+------------+---------+
| CustomerName | ContactName     | Address       | City      | PostalCode | Country |
+--------------+-----------------+---------------+-----------+------------+---------+
| Cardinal     | Tom B. Erichsen | Skagen 21     | Stavanger | 4006       | Norway  |
| Wilman Kala  | Matti Karttunen | Keskuskatu 45 | Helsinki  | 21240      | Finland |
+--------------+-----------------+---------------+-----------+------------+---------+
2 rows in set (0.00 sec)
```

现在，让我们尝试向现有的数据库表中添加几行。我们将使用一个`INSERT`查询向表中添加数据:

```
mysql> INSERT INTO Customers (CustomerName, ContactName, Address, City, PostalCode, Country) VALUES ('White Clover Markets', 'Karl Jablonski', '305 - 14th Ave. S. Suite 3B', 'Seattle', '98128', 'USA');
Query OK, 1 row affected (0.00 sec)
```

我们还成功地在恢复的表中插入了一个新行。恭喜你。让我们看看结果:

```
mysql> select * from Customers;
+----------------------+-----------------+-----------------------------+-----------+------------+---------+
| CustomerName         | ContactName     | Address                     | City      | PostalCode | Country |
+----------------------+-----------------+-----------------------------+-----------+------------+---------+
| Cardinal             | Tom B. Erichsen | Skagen 21                   | Stavanger | 4006       | Norway  |
| Wilman Kala          | Matti Karttunen | Keskuskatu 45               | Helsinki  | 21240      | Finland |
| White Clover Markets | Karl Jablonski  | 305 - 14th Ave. S. Suite 3B | Seattle   | 98128      | USA     |
+----------------------+-----------------+-----------------------------+-----------+------------+---------+
3 rows in set (0.00 sec)
```

或者，MySQL 服务器容器随 MySQL 客户端安装一起提供。但是，它只能在容器内用于任何测试目的。现在，让我们登录 Docker 容器，并尝试使用默认的 MySQL 客户端访问 MySQL 服务器。

`docker exec`命令帮助使用容器 id 登录到正在运行的容器。选项`-i`保持 STDIN 打开，`-t`将分配伪 TTY，最后，最后的`/bin/bash`让我们进入 BASH 提示符:

```
# docker exec -it 9ce4da8eb682 /bin/bash
[[email protected]](/web/20220928150307/https://www.baeldung.com/cdn-cgi/l/email-protection):/# mysql -h localhost -u root -p
Enter password:
Welcome to the MySQL monitor.  Commands end with ; or \g.
...
... output truncated ...
...
mysql>
```

## 6.结论

总之，我们讨论了使用`docker-compose`启动 MySQL 服务器容器的步骤。它还自动从备份文件中恢复数据库和表。此外，我们还访问了恢复的数据并执行了一些 CRUD 操作。