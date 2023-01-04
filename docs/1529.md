# 如何包含 Docker 构建上下文之外的文件

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/ops/docker-include-files-outside-build-context>

## 1.概观

一般来说，Docker [`build`](https://web.archive.org/web/20221115155727/https://docs.docker.com/engine/reference/commandline/build/) 命令限制了我们可以在 Docker 映像中使用的文件的来源。我们指定一个构建上下文，它是必须从中找到`Dockerfile`及其所有依赖文件的根。

然而，有时候，我们可能希望将文件系统一部分的`Dockerfile`用于另一部分的文件。

在这个简短的教程中，我们将看看几种克服这种限制的方法，以及它们的优缺点。

## 2.码头建筑及其背景

### 2.1.传统码头建筑

让我们从一个例子开始——一个简单的`nginx`应用程序，它有一个 HTML 文件和一个`Dockerfile`。目录结构是:

```
projects
├ <some other projects>...
└── sample-site
       ├── html
       │    └── index.html
       └── Dockerfile
```

我们还有一个小`Dockerfile`:

```
FROM nginx:latest
COPY html/* /etc/nginx/html/
```

现在，要为这个应用程序构建一个映像，让我们运行:

```
$ docker build -t sample-site:latest .
```

这里，构建上下文通过“.”设置为当前目录论点。

将`Dockerfile`保存在项目根目录是一种常见的做法。默认情况下，该命令期望`Dockerfile`出现在那里。我们希望包含在图像中的所有文件应该存在于该上下文中的某个位置。

### 2.2.更复杂的构建场景

有时候传统的方法可能对我们不起作用。

例如，基于不同的环境，我们可能有不同的`Docker`或`docker-compose`文件。或者，我们可能将添加容器作为独立于开发的活动。

在这些情况下，将与 Docker 相关的文件移动到一个单独的目录是有意义的。类似地，在某些情况下，我们可能会将图像的配置文件保存在项目根目录之外。

不幸的是，Docker 阻止我们从文件系统的任意部分添加文件，因为这可能会打开一个安全漏洞。但是，有一些变通方法:

*   构建一个更大的环境来包含所有需要的东西
*   使用位于上下文之外的文件创建基础映像，然后扩展基础映像
*   复制所有必要的文件来创建一个临时上下文，并从中构建映像

让我们一个一个地检查一下。

## 3.在更大的背景下构建

假设我们想要将`Dockerfile`移动到一个名为`docker`的单独目录中。我们还想用一个定制的配置文件覆盖标准的`nginx`配置，该文件位于项目根目录`sample-site`之外的`config`目录中。新的目录结构是:

```
projects
├ <some other projects>...
├── sample-site
│      ├── html
│      │    └── index.html
│      └── docker
│           └── Dockerfile
└── config
      └── nginx.conf 
```

在这种情况下，先前的`Dockerfile`和`docker build`命令都不再起作用。为了让它再次工作，**我们必须用一个更大的上下文——T2 目录**来构建它。

### 3.1.改变`Dockerfile`

既然我们的环境已经改变，我们需要改变我们的`Dockerfile`:

```
FROM nginx:latest
COPY sample-site/html/* /etc/nginx/html/
COPY config/nginx.conf /etc/nginx/nginx.conf
```

我们已经根据新的上下文修改了`html`目录的路径。我们还包括了`nginx.conf`的位置。

### 3.2.建立形象

现在，让我们转到`projects`目录并运行命令来构建映像:

```
$ cd projects
$ docker build -f sample-site/docker/Dockerfile -t sample-site:latest .
```

这里，我们再次使用“.”作为上下文，当我们从`projects`目录运行命令时。这将`Dockerfile`和`nginx.conf`带入当前的构建上下文中。由于`Dockerfile`不在上下文目录的根目录下，我们使用`-f`选项提供它的路径。

这种方法的问题是 Docker 客户机向 Docker 守护进程发送一份构建上下文的副本——整个`projects`目录。该目录可能包含许多其他不相关的文件和目录。因此，这可能需要 Docker 扫描大量资源，这会导致构建过程缓慢。

## 4.使用外部文件创建基础映像

另一种方法是**用外部文件创建一个基础映像，然后再**扩展它。我们将重用与上一个示例相同的结构:

```
projects
├ <some other projects>...
├── sample-site
│      ├── html
│      │    └── index.html
│      └── docker
│           └── Dockerfile
└── config
       └── nginx.conf
```

### 4.1.写`Dockerfile`为基数

首先，让我们用配置写一个`Dockerfile`:

```
FROM nginx:latest
COPY nginx.conf /etc/nginx/nginx.conf
```

我们将文件放在`projects/config`目录中。

### 4.2.建立基础

下一步是运行`projects/config`中的构建命令来创建基础映像:

```
$ docker build -t sample-site-base:latest .
```

现在，我们有一个名为`sample-site-base:latest`的`Docker`映像，包含`nginx`服务器和配置文件。

### 4.3.为孩子写`Dockerfile`

接下来，让我们编写`sample-site`的`Dockerfile`来扩展*样地库*:

```
FROM sample-site-base:latest
COPY html/* /etc/nginx/html/
```

### 4.4.打造孩子

最后，让我们运行`projects/sample-site`中的命令来构建我们的应用程序的映像:

```
$ docker build -f docker/Dockerfile -t sample-site:latest .
```

这里，我们将 Docker 构建分成两个独立的阶段，每个阶段都与项目中不同的目录树相关。

这种方法在其子图像中重用了`Dockerfile`的公共部分。这种结构相对容易维护。如果基础映像有任何变化，我们需要重新构建子映像，同样的变化会反映在所有子映像中。

我们应该注意到**这种方法增加了我们最终 Docker 图像的层数**。

## 5.创建临时上下文

我们最后的选择是**创造一个量身定制的临时环境来建立形象**。这使用了一些围绕我们的`docker build`命令的脚本来将必要的文件拉到 Docker 友好的目录结构中。

### 5.1.创建一个临时目录

首先，让我们创建一个临时目录，并复制所有必需的资产:

```
$ mkdir tmp-context
$ cp -R ../html tmp-context/
$ cp -R ../../config tmp-context/
```

这将是我们的构建环境。

### 5.2.创建`Dockerfile`

现在，让我们相对于这个上下文来写`Dockerfile`:

```
FROM nginx:latest
COPY html/* /etc/nginx/html/
COPY config/nginx.conf /etc/nginx/nginx.conf
```

我们已经把需要的文件放在了`tmp-context`中，所以我们不需要在这里提到任何外部路径。

### 5.3.建立形象

让我们运行命令来构建映像:

```
$ cd tmp-context
$ docker build -t sample-site:latest .
```

该命令将`tmp-context`作为构建上下文。它会在目录中找到它需要的所有东西，从而毫无问题地构建映像。

### 5.4.打扫

最后，让我们清理临时目录:

```
$ rm -rf tmp-context
```

这是从任何地方添加文件到 Docker 映像的最干净的方法。在这里，我们可以完全控制如何创建我们的环境。我们可以用上面所有的命令创建一个 bash 脚本来简化整个过程。

但是，我们应该理解有些文件可能很大。它们可能需要很长时间才能被复制到上下文中，这在每个构建中都可能发生。

## 6.结论

在本文中，我们看到了 Docker 通常如何从与`Dockerfile`相同目录中的文件构建图像。我们研究了一些从通常的构建环境之外添加文件的解决方案。

我们还探讨了每个解决方案的优点和缺点。

与往常一样，与本文相关的示例代码可以在 GitHub 上的[处获得。](https://web.archive.org/web/20221115155727/https://github.com/eugenp/tutorials/tree/master/docker-modules/docker-images)