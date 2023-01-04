# Kaniko 简介

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/ops/kaniko>

## 1.介绍

在本教程中，我们将看看如何使用 [Kaniko](https://web.archive.org/web/20221104125813/https://github.com/GoogleContainerTools/kaniko) 构建容器图像。

## 2\. Kaniko

Kaniko 是一个从`Dockerfile`构建容器图像的工具。与 Docker 不同，Kaniko 不需要 Docker 守护进程。

由于不依赖于守护进程，这可以在任何用户没有 root 访问权限的环境中运行，比如 Kubernetes 集群。

Kaniko 使用运行在容器内部的`executor image` : `gcr.io/kaniko-project/executor` 在用户空间中完全执行`Dockerfile`中的每个命令；例如，一个库伯内特斯豆荚。它按顺序执行`Dockerfile`中的每个命令，并在每个命令后拍摄文件系统的快照。

如果文件系统有变化，执行器将文件系统变化的快照作为“diff”层，并更新映像元数据。

部署和运行 Kaniko 有不同的方式:

*   不可思议的群集
*   gVisor
*   谷歌云构建

在本教程中，我们将使用 Kubernetes 集群部署 Kaniko。

## 3.安装 Minikube

我们将使用 Minikube 在本地部署 Kubernetes。它可以作为独立的二进制文件下载:

```
$ curl -Lo minikube https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64 &&
  chmod +x minikube
```

然后，我们可以将 Minikube 可执行文件添加到以下路径:

```
$ sudo mkdir -p /usr/local/bin/
$ sudo install minikube /usr/local/bin/
```

接下来，让我们确保 Docker 守护进程正在运行:

```
$ docker version
```

此外，这个命令将告诉我们安装的客户机和服务器版本。

现在，我们可以创建我们的 Kubernetes 集群:

```
$ minikube start --driver=docker
```

一旦 start 命令成功执行，我们将看到一条消息:

```
Done! kubectl is now configured to use "minikube"
For best results, install kubectl: https://kubernetes.io/docs/tasks/tools/install-kubectl/
```

运行`minikube status`命令后，我们应该看到`kubelet`状态为“`Running`”:

```
m01
host: Running
kubelet: Running
apiserver: Running
kubeconfig: Configured
```

接下来，我们需要设置`kubectl`二进制文件来运行 Kubernetes 命令。让我们下载二进制文件并使其可执行:

```
$ curl -LO https://storage.googleapis.com/kubernetes-release/release/`curl -s \ 
  https://storage.googleapis.com/kubernetes-release/release/stable.txt`/bin/linux/amd64/kubectl &&
  chmod +x ./kubectl
```

现在让我们将它移到路径:

```
$ sudo mv ./kubectl /usr/local/bin/kubectl
```

我们可以使用以下方法验证版本:

```
$ kubectl version
```

## 4.使用 Kaniko 构建图像

现在我们已经准备好了 Kubernetes 集群，让我们开始使用 Kaniko 构建一个映像。

首先，我们需要创建一个本地目录，它将作为构建上下文安装在 Kaniko 容器中。

为此，我们需要 SSH 到 Kubernetes 集群并创建目录:

```
$ minikube ssh
$ mkdir kaniko && cd kaniko
```

接下来，让我们创建一个 Dockerfile，它提取 Ubuntu 图像并回显一个字符串"`hello”`:

```
$ echo 'FROM ubuntu' >> dockerfile
$ echo 'ENTRYPOINT ["/bin/bash", "-c", "echo hello"]' >> dockerfile
```

如果我们现在运行`cat dockerfile` ，我们应该会看到:

```
FROM ubuntu
ENTRYPOINT ["/bin/bash", "-c", "echo hello"]
```

最后，我们将运行`pwd`命令，获取本地目录的路径，稍后需要在持久卷中指定该路径。

其输出应该类似于:

```
/home/docker/kaniko
```

最后，我们可以中止 SSH 会话:

```
$ exit
```

### 4.1.Kaniko 执行者图像参数

在进一步创建 Kubernetes 配置文件之前，让我们看一下 Kaniko executor 映像需要的一些参数:

*   docker File(`–dockerfile`)–包含构建映像所需的所有命令的文件
*   构建上下文(`–context`)–这类似于 Docker 的构建上下文，它指的是 Kaniko 用来构建映像的目录。到目前为止， **Kaniko 支持谷歌云存储(GCS)、亚马逊 S3、Azure blob 存储、Git 存储库和本地目录。在本教程中，我们将使用之前配置的本地目录。**
*   destination(`–destination`)–这是指 Docker 注册中心或任何类似的存储库，我们将图像推送到这些存储库。这个参数是强制性的。如果我们不想推送图像，我们可以通过使用`–no-push`标志来覆盖该行为。

我们还可以看到可以传递给的[附加标志。](https://web.archive.org/web/20221104125813/https://github.com/GoogleContainerTools/kaniko#additional-flags)

### 4.2.设置配置文件

现在让我们开始创建在 Kubernetes 集群中运行 Kaniko 所需的配置文件。

首先，让我们创建持久卷，它提供了之前在集群中创建的卷挂载路径。让我们把这个文件叫做`volume.yaml`:

```
apiVersion: v1
kind: PersistentVolume
metadata:
  name: dockerfile
  labels:
    type: local
spec:
  capacity:
    storage: 10Gi
  accessModes:
    - ReadWriteOnce
  storageClassName: local-storage
  hostPath:
    path: /home/docker/kaniko # replace this with the output of pwd command from before, if it is different
```

接下来，让我们为这个持久卷创建一个持久卷声明。我们将创建一个文件`volume-claim.yaml`,包含:

```
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: dockerfile-claim
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 8Gi
  storageClassName: local-storage
```

最后，让我们创建包含执行者图像的 pod 描述符。这个描述符引用了上面指定的卷挂载，它又指向我们之前创建的`Dockerfile`。

我们将该文件称为`pom.yaml`:

```
apiVersion: v1
kind: Pod
metadata:
  name: kaniko
spec:
  containers:
  - name: kaniko
    image: gcr.io/kaniko-project/executor:latest
    args: ["--dockerfile=/workspace/dockerfile",
            "--context=dir://workspace",
            "--no-push"] 
    volumeMounts:
      - name: dockerfile-storage
        mountPath: /workspace
  restartPolicy: Never
  volumes:
    - name: dockerfile-storage
      persistentVolumeClaim:
        claimName: dockerfile-claim
```

如前所述，在本教程中，我们只关注使用 Kaniko 的图像创建，而不是发布它。因此，我们在执行器的参数中指定了`no-push`标志。

准备好所有必需的配置文件后，让我们应用它们:

```
$ kubectl create -f volume.yaml
$ kubectl create -f volume-claim.yaml
$ kubectl create -f pod.yaml
```

应用描述符后，我们可以检查 Kaniko pod 是否进入完成状态。我们可以使用`kubectl get po`检查这一点:

```
NAME     READY   STATUS      RESTARTS   AGE
kaniko    0/1   Completed       0        3m
```

现在，我们可以使用`kubectl logs kaniko,`来检查此 pod 的日志，以检查映像创建的状态，它应该会显示以下输出:

```
INFO[0000] Resolved base name ubuntu to ubuntu          
INFO[0000] Resolved base name ubuntu to ubuntu          
INFO[0000] Retrieving image manifest ubuntu             
INFO[0003] Retrieving image manifest ubuntu             
INFO[0006] Built cross stage deps: map[]                
INFO[0006] Retrieving image manifest ubuntu             
INFO[0008] Retrieving image manifest ubuntu             
INFO[0010] Skipping unpacking as no commands require it. 
INFO[0010] Taking snapshot of full filesystem...        
INFO[0013] Resolving paths                              
INFO[0013] ENTRYPOINT ["/bin/bash", "-c", "echo hello"] 
INFO[0013] Skipping push to container registry due to --no-push flag
```

我们可以在输出中看到，容器已经执行了我们放在`Dockerfile`中的步骤。

它开始于拉动基本的 Ubuntu 映像，结束于向入口点添加`echo`命令。由于指定了`no-push`标志，它没有将图像推送到任何存储库。

如前所述，我们还看到在添加入口点之前拍摄了文件系统的快照。

## 5.结论

在本教程中，我们看了 Kaniko 的基本介绍。我们已经看到了如何使用它来构建映像，以及如何使用 Minikube 设置 Kubernetes 集群，并为 Kaniko 提供运行所需的配置。

像往常一样，本文中使用的代码片段可以从 GitHub 上的[处获得。](https://web.archive.org/web/20221104125813/https://github.com/eugenp/tutorials/tree/master/kaniko)