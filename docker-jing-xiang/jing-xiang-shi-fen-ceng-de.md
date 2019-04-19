# 镜像是分层的

镜像为啥分层？说这个问题不如想想来源，镜像分层是解决啥？

虽然镜像解决了打包，但是实际应用中我们的应用都是基于同一个rootfs来打包和迭代的，难道是每个rootfs都会多份吗？为此docker利用了存储驱动AUFS，devicemapper，overlay，overlay2的存储技术实现了分层。初期时AUFS，到现在的overlay2驱动，不推荐devicemapper坑很多。例如一个nginx:alpine和python:alpine镜像可以从分层角度这样去理解

![](../.gitbook/assets/image%20%2866%29.png)

实际上只有不同的才占据存储空间，相同的是引用关系。默认存储路径是`/var/lib/docker/存储驱动类型/`

![](../.gitbook/assets/image%20%2812%29.png)

