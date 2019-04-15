# Hello World

运行下docker的hello-world

```text
$ docker run hello-world

Hello from Docker!
This message shows that your installation appears to be working correctly.

To generate this message, Docker took the following steps:
 1. The Docker client contacted the Docker daemon.
 2. The Docker daemon pulled the "hello-world" image from the Docker Hub.
    (amd64)
 3. The Docker daemon created a new container from that image which runs the
    executable that produces the output you are currently reading.
 4. The Docker daemon streamed that output to the Docker client, which sent it
    to your terminal.

To try something more ambitious, you can run an Ubuntu container with:
 $ docker run -it ubuntu bash

Share images, automate workflows, and more with a free Docker ID:
 https://hub.docker.com/

For more examples and ideas, visit:
 https://docs.docker.com/get-started/

```

我们可以看到输出的内容得到几个信息

1. docker是个c/s架构，我们运行的docker命令实际上是发送到docker daemon，daemon端执行的。默认docker daemon上带一个客户端docker命令的，也就是说默认安装docker客户端和daemon端是在一起的（docker命令通过`/var/run/docker.sock`和docker daemon通信）
2. docker daemon从dockerhub上拉取了一个hello-world的镜像
3. docker daemon从镜像创建了容器然后运行输出了上面那一堆信息
4. docker daemon把输出传送到docker客户端送到了当前的终端

这里多了俩个名词，镜像和容器，下面先讲解下docker的镜像

