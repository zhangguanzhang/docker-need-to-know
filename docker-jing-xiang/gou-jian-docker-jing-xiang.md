# 构建Docker镜像

构建镜像只有两种方式，docker build 和 docker commit。实际上docker build是调用的docker commit。不推荐手动去run的容器去docker commmit成镜像。所以主要讲docker build和dockerfile

使用docker build 指定Dockerfile来完成一个新镜像的构建。命令格式一般为：

```text
docker build [option] [-t <image>:<tag>]  <path>
```

其中path指向的文件称为context（上下文），context包含build docker镜像过程中需要的Dockerfile以及其他的资源文件。执行build命令后执行流程如下

**Docker client端：**

1）解析命令行参数，完成对相关信息的设置，Docker client向Docker server发送POST/build的HTTP请求，包含了所需的上下文文件

**Docker server端：**

1）创建一个临时目录，并将context指定的文件系统解压到该目录下

2）读取并解析Dockerfile

3）根据解析出的Dockerfile遍历其中的所有指令，并分发到不同的模块（parser）去执行

4）parser为Dockerfile的每一个指令创建一个对应的临时容器，在临时容器中执行当前指令，然后通过commit使用此镜像生成一个镜像层

5）Dockerfile中所有的指令对应的层的集合，就是此次build后的结果。如果指定了tag参数，便给镜像打上对应的tag。最后一次commit生成的镜像ID就会作为最终的镜像ID返回。

主要是用户编写dockerfile来描述构建镜像的过程，接下来讲解下Dockerfile的字段和注意事项

