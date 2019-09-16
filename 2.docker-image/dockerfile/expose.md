# EXPOSE

用法

```text
EXPOSE <port> [<port>/<protocol>...]
```

例子

```text
EXPOSE 80/tcp
EXPOSE 80/udp
EXPOSE 80 443
```

声明需要暴露的端口（缺省tcp），仅仅是声明并没有说写了它才能映射端口，对容器网络不熟悉的话后面会讲容器网络的。我们可以看到nginx官方镜像的Dockerfile里有写80

```text
EXPOSE 80
```

我们假设简单的run起来让外部访问的话可以这样

```text
docker run -d -p 80:80 nginx:alpine
```

这条命令是使用nginx:alpine镜像运行一个容器，把宿主机的80映射到容器的80端口上，我们可以访问宿主机ip:80就可以看到默认nginx的index页面，如果说是云主机80可能需要备案，可以改成81:80。可以自己把nginx官方dockerfile的EXPOSE删掉发现还可以映射的。

EXPOSE作用是告诉使用者应该把容器的哪个端口暴漏出去。另一个作用给`docker run -P`用的

```text
docker run -P nginx:alpine
```

会映射宿主机上随机没被bind的端口到EXPOSE的端口，例如  random\_port:80

