---
description: 用户可以docker build时使用--build-arg <varname>=<value> 将该变量传递给构建器替换默认变量
---

# ARG

格式有两种

```text
ARG key1
ARG key2
----------------
ARG key=value
ARG key2=value2
...
```

一般来讲第二种用得多，表明build的时候不传入变量设置默认值，无值也就是第一种下用户在`docker build`的时候必须传入值，否则就报错。例如我们可以把 nginx 官方 dockerfile 的第一个 ENV 改成 ARG，我们想构建哪个版本直接build的时候传入变量就行了。例如下面这样`VERSION=2.4.7`

```text
ARG VERSION=2.4.7
RUN .... && \
    curl -sS https://downloads.mariadb.com/MariaDB/mariadb_repo_setup \
        | bash -s -- -mariadb-maxscale-version $VERSION
```

当然 ARG 是唯一一个可以用于 FROM 前面的指令，例如下面这样我们可以通过命令行传递参数来改变FROM的 base 镜像

```text
ARG jdk=1.8xxxx
FROM openjdk:$jdk
```

如果你在 FROM 前面使用 arg 指令会有个坑，在 FROM 前面的 ARG 指令的值，会在过了这个 FROM 后面都是空值的

```text
$ cat Dockerfile //Dockerfle的内容
ARG version=1
FROM alpine
ARG var=2
RUN echo ${version}-${var} >/test
$ docker build -t test . //构建镜像
Sending build context to Docker daemon  2.048kB
Step 1/4 : ARG version=1
Step 2/4 : FROM alpine
 ---> a24bb4013296
Step 3/4 : ARG var=2
 ---> Running in ed406679adf9
Removing intermediate container ed406679adf9
 ---> 7811965e12b7
Step 4/4 : RUN echo ${version}-${var} >/test
 ---> Running in ac3d3c203ce7
Removing intermediate container ac3d3c203ce7
 ---> 0fefcb646a01
Successfully built 0fefcb646a01
Successfully tagged test:latest
$ docker run --rm test cat /test //运行容器查看文件内容
-2 //这里不是预期的1-2结果
```

Docker其实也预定了一些ARG方便我们构建的时候使用代理

* HTTP\_PROXY 
* http\_proxy 
* HTTPS\_PROXY 
* https\_proxy 
* FTP\_PROXY 
* ftp\_proxy 
* NO\_PROXY 
* no\_proxy

