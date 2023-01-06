  ![如何删除 Docker 镜像和容器](https://chinese.freecodecamp.org/news/content/images/size/w2000/2021/01/photo-1548092372-0d1bd40894a3.jpeg)

## ******Docker rmi******

`docker rmi` 通过镜像的 ID 删除镜像。

要删除镜像，首先需要列出所有镜像以获取镜像的 ID，镜像的名称和其他详细信息。 运行简单的命令 `docker images -a` 或 `docker images`。

之后，明确要删除哪个镜像，然后执行简单命令 `docker rmi <your-image-id>`。然后，列出所有镜像并检查，可以确认镜像是否已删除。

### ******一次删除多张******镜像

当你要一次删除多张镜像时，可以使用一种方法。首先只需列出镜像即可获取镜像的 ID，然后执行简单的命令：

`docker rmi <your-image-id> <your-image-id> ...`

列出镜像的 ID，每个 ID 之间留一个空格。

### ******一次删除所有******镜像

要删除所有镜像，有一个简单的命令可以做到：`docker rmi $(docker images -q)`。

在上面的命令中，有两个命令，第一个在 `$()` 中执行的命令是 shell 语法，返回以该执行的结果。然后，`-q-` 是一个选项，用于返回唯一的 ID。$() 返回镜像 ID 的结果，然后 `docker rmi` 删除所有这些镜像。

#### ******更多信息****：**

-   [Docker CLI docs: rmi](https://docs.docker.com/engine/reference/commandline/rm/)

`docker rm` 根据容器的名称或者 ID 来删除容器。

如果 Docker 容器正在运行，你在删除它们之前需要先停止运行。

-   停止所有容器运行：`docker stop $(docker ps -a -q)`
-   删除所有停止运行的容器：`docker rm $(docker ps -a -q)`

### ******删除多个容器******

你可以通过向命令传递要删除的容器列表来停止和删除多个容器。shell 语法 `$()` 返回括号中执行的任何结果。因此，你可以在其中创建容器列表，以传递给 `stop` 和 `rm` 命令。

### **docker ps -a -q 分解**

-   `docker ps` 列出容器。
-   `-a` 这个选项用于列出所有容器，包括停止运行的。如果没有这个选项，则默认只列出在运行的容器。
-   `-q` 这个选项列出容器的数字 ID，而不是容器的所有信息。

#### ******更多信息****：**

-   [Docker CLI docs: rm](https://docs.docker.com/engine/reference/commandline/rm/)

## **关于 Docker** 镜像**的更多信息：**

-   [Docker](https://www.freecodecamp.org/news/docker-image-guide-how-to-remove-and-delete-docker-images-stop-containers-and-remove-all-volumes/) 镜像[指南](https://www.freecodecamp.org/news/docker-image-guide-how-to-remove-and-delete-docker-images-stop-containers-and-remove-all-volumes/)
-   [Docker](https://www.freecodecamp.org/news/where-are-docker-images-stored-docker-container-paths-explained/) 镜像[存储在哪里](https://www.freecodecamp.org/news/where-are-docker-images-stored-docker-container-paths-explained/)

## **关于 Docker 容器的更多信息：**

-   [如何自动化部署 Docker 容器](https://www.freecodecamp.org/news/automate-docker-container-deployment-via-maven-53a855e26d3e/)
-   [如何修复 Docker 容器](https://www.freecodecamp.org/news/how-to-find-and-fix-docker-container-vulnerabilities-in-2020/)

## **关于 Docker 的更多信息：**

-   [Docker 入门指南](https://www.freecodecamp.org/news/a-beginners-guide-to-docker-how-to-create-your-first-docker-application-cc03de9b639f/)
-   [Docker DevOps 课程（视频）](https://www.freecodecamp.org/news/docker-devops-course/)
-   [Docker 101：从创建到部署](https://www.freecodecamp.org/news/docker-101-creation-to-deployment/)

原文：[How to Remove Images and Containers in Docker](https://www.freecodecamp.org/news/how-to-remove-images-in-docker/)

___

___

在 freeCodeCamp 免费学习编程。 freeCodeCamp 的开源课程已帮助 40,000 多人获得开发者工作。[开始学习](https://www.freecodecamp.org/chinese/learn/)