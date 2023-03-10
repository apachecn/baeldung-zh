# 在 Dockerfile 文件副本中保留子目录结构

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/ops/dockerfile-copy-same-subdirectory-structure>

## 1.概观

在本文中，我们将学习将一个目录复制到 Docker 映像中，并保留子目录结构。

## 2.将本地目录复制到映像

让我们创建以下文件树:

```java
|   Dockerfile
|   
\---folder1
    +---subfolder1
    |       file1.txt
    |       
    \---subfolder2
            file2.txt
```

这可以通过运行以下命令来完成:

```java
$ mkdir folder1
$ cd folder1
$ mkdir subfolder1
$ cd subfolder1
$ touch file1.txt
$ cd ..
$ mkdir subfolder2
$ cd subfolder2
$ touch file2.txt
$ cd ../.. 
```

我们现在开始我们的`[Dockerfile](/web/20221005092324/https://www.baeldung.com/ops/docker-compose#2-building-an-image)`:

`$ touch Dockerfile`

然后，让我们插入以下内容:

```java
FROM ubuntu:latest
COPY folder1/ /workdir/
RUN ls --recursive /workdir/
```

让我们逐行理解内容:

*   第一行声明我们使用最新的 ubuntu 映像作为基础映像
*   **第二行将`folder1`目录的内容复制到图像的`workdir`目录中。**如果`workdir`还不存在，它将被创建
*   第三行执行图像外壳中的 [`ls`](/web/20221005092324/https://www.baeldung.com/linux/symlinks-in-listing-all-files#using-ls) 命令，递归地列出`workdir`文件夹中所有子目录的内容

我们现在可以建立我们的 Docker 形象了:

```java
$ docker build .
#4 [1/3] FROM docker.io/library/ubuntu:latest
#6 [2/3] COPY folder1/ /workdir/
#7 [3/3] RUN ls --recursive /workdir/
#7 0.324 /workdir/:
#7 0.324 subfolder1
#7 0.324 subfolder2
#7 0.324
#7 0.324 /workdir/subfolder1:
#7 0.324 file1.txt
#7 0.324
#7 0.324 /workdir/subfolder2:
#7 0.324 file2.txt
```

不出所料，递归打印`workdir`的内容突出显示它包含了`folder1`的所有子目录和文件。

## 3.将本地目录合并到映像中

现在，让我们稍微更新一下文件树，使其符合以下内容:

```java
|   Dockerfile
|   
+---folder1
|   +---subfolder1
|   |       file1.txt
|   |       
|   \---subfolder2
|           file2.txt
|           
\---folder2
        file3.txt
```

为了进行修改，我们将运行以下命令:

```java
$ mkdir folder2
$ cd folder2
$ touch file3.txt
$ cd ..
```

**我们现在想要将`folder2`的内容合并到图像的`workdir`中。**让我们完成我们的`Dockerfile`:

```java
FROM ubuntu:latest
COPY folder1/ /workdir/
RUN ls --recursive /workdir/
COPY folder2/ /workdir/
RUN ls --recursive /workdir/
```

第二个`COPY`指令不会删除之前添加的文件。让我们建立自己的形象来检查这种行为:

```java
$ docker build .
#4 [1/5] FROM docker.io/library/ubuntu:latest
#6 [2/5] COPY folder1/ /workdir/
#7 [3/5] RUN ls --recursive /workdir/
#8 [4/5] COPY folder2/ /workdir/
#9 [5/5] RUN ls --recursive /workdir/
#9 0.398 /workdir/:
#9 0.398 file3.txt
#9 0.398 subfolder1
#9 0.398 subfolder2
#9 0.398
#9 0.398 /workdir/subfolder1:
#9 0.398 file1.txt
#9 0.398
#9 0.398 /workdir/subfolder2:
#9 0.398 file2.txt
```

如日志所示，`folder1`和`folder2`的所有子目录确实被复制到了`workdir`。

另外，我们选择这个例子是为了演示合并行为。如果我们想要的只是将`folder1`和`folder2`的内容同时复制到图像的`workdir`目录中，我们可以利用 **`COPY`可以接受多个来源**这一事实:

```java
FROM ubuntu:latest
COPY folder1/ folder2/ /workdir/
RUN ls --recursive /workdir/
```

## 4.结论

在本教程中，我们看到了如何将本地目录复制到 Docker 映像，同时保持它们的子目录结构。

和往常一样，代码可以在 GitHub 上找到[。](https://web.archive.org/web/20221005092324/https://github.com/eugenp/tutorials/tree/master/docker-modules/docker-images)