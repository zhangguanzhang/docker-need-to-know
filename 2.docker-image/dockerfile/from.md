# FROM

用法：

FROM &lt;baseimage&gt;   或者   FROM &lt;baseimage&gt;:&lt;tag&gt;

指定从哪个镜像为基础迭代，如果本地没有镜像则会从仓库拉取，通常是第一行，镜像默认tag为latest，生产中不能使用latest标签。而scratch是空镜像，是所有rootfs和一些单独可执行文件做镜像的根源，关于scratch后续会说

例如centos的Dockerfile是下面

```text
FROM scratch
ADD centos-7-x86_64-docker.tar.xz /

LABEL org.label-schema.schema-version="1.0" \
    org.label-schema.name="CentOS Base Image" \
    org.label-schema.vendor="CentOS" \
    org.label-schema.license="GPLv2" \
    org.label-schema.build-date="20190305"

CMD ["/bin/bash"]
```

而hello-world为

```text
FROM scratch
COPY hello /
CMD ["/hello"]
```

```text
docker images | grep hello
hello-world         latest              fce289e99eb9        3 months ago        1.84kB
```

hello-world镜像非常小，后续会讲它

nginx:alpine镜像的dockerfile 链接为[https://github.com/nginxinc/docker-nginx/blob/7d7c67f2eaa6b2b32c718ba9d93f152870513c7c/mainline/alpine/Dockerfile](https://github.com/nginxinc/docker-nginx/blob/7d7c67f2eaa6b2b32c718ba9d93f152870513c7c/mainline/alpine/Dockerfile)，后续大多会以它为例子讲解，因为不会去重复造轮子。

nginx:alpine既满足运行的最小环境下大小又很小，主要归功于`FROM alpine` ，现在alpine这个系统和rootfs得益于docker发展非常快，也有很多应用镜像都有alpine版本

```text
FROM alpine:3.9

LABEL maintainer="NGINX Docker Maintainers <docker-maint@nginx.com>"

ENV NGINX_VERSION 1.15.11

RUN ...省略步骤，步骤是下载源码，安装编译需要的依赖，编译安装完删掉源码包和编译的依赖保留运编译出来的nginx二进制和需要的所有so文件

COPY nginx.conf /etc/nginx/nginx.conf
COPY nginx.vh.default.conf /etc/nginx/conf.d/default.conf

EXPOSE 80

STOPSIGNAL SIGTERM

CMD ["nginx", "-g", "daemon off;"]
```

