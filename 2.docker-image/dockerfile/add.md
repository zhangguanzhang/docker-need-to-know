---
description: 类似于COPY，但是支持下载文件和自动解压
---

# ADD

 和COPY一样，但是源可以是一个url会自动下载，另外源是压缩包的话会自动解压，但是实际中不会使用它，因为前面讲`RUN`的时候说的层概念。例如下面是一个ADD用的多的举例

```text
ADD https://xxxxx/name-$version.tar.gz /home/test/
RUN cd /home/test && \
    编译安装... \
    rm -rf /home/test
```

ADD下载源码包，然后RUN里编译安装完删除源码包。实际上后面的层起来的容器虽说读取不到源码包了，但是实际上文件还是在镜像里，参照我之前的RUN里那个test.html的例子。

一般避免多余的层和容量都是RUN里去下载源码包，处理完后删掉源码包，参照nginx的dockerfile的第一个RUN。 [https://github.com/nginxinc/docker-nginx/blob/7d7c67f2eaa6b2b32c718ba9d93f152870513c7c/mainline/alpine/Dockerfile\#L7](https://github.com/nginxinc/docker-nginx/blob/7d7c67f2eaa6b2b32c718ba9d93f152870513c7c/mainline/alpine/Dockerfile#L7)

总结就是不推荐使用ADD

另外要注意alpine的tar的一个坑，例如

```text
RUN apk add curl && \
    curl -sL https:/xxxx/xxx.tar.gz | tar -zxvf - 
```

这样在alpine的基础镜像下某些情况下会有几率报错`tar: invalid magic`,  推荐下载后解压，用完了再删掉压缩包和相关文件。或者安装tar命令

[https://tutel.me/c/unix/questions/302192/how+to+solve+tar+invalid+magic+error+on+linux+alpine](https://tutel.me/c/unix/questions/302192/how+to+solve+tar+invalid+magic+error+on+linux+alpine)

