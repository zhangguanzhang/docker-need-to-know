---
description: 把外部文件复制到构建的容器里，因为commit所以会到镜像里，也可以说把文件复制到镜像里
---

# COPY

用法

```text
COPY <src>  <dest>  
COPY ["<src>",... "<dest>"]  
COPY home* /home
```

复制本地的文件到容器中的目录，目录不存在则会自动创建，源可以是多个。在低版本的docker里如果源是绝对路径例如/root/data/nginx的话会把整个系统的根上传到docker daemon，会发现上传的内容等同于根的已用容量，例如下面

```text
$ cat Dockerfile
FROM alpine
COPY /root/data/nginx.tar.gz /root/home
$ docker build -t test .
Sending build context to Docker daemon  7.8GB
```

主要是因为上下文的概念，认为上下文的根是client的/，所以会把客户端的/上传到docker daemon，现在新版本是强制相对路径了，如果是绝对路径会报错。相对路径相对于build最后的`.`这个上下文路径为相对路径。

另外COPY还能指定uid：gid，如果容器的rootfs里没有文件`/etc/passwd`和`/etc/group`文件只能使用数字不能使用组名

```text
COPY --chown=55:mygroup files* /somedir/
COPY --chown=bin files* /somedir/
COPY --chown=1 files* /somedir/
COPY --chown=10:11 files* /somedir/
```

 `COPY`接受一个标志`--from=<name|index>`，该标志可用于将源位置设置为`FROM .. AS <name> 主要用于多阶段构建，后面会举个例子来讲解多阶段构建，多阶段构建是17.05之后才出现的功能`

