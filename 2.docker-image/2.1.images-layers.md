# 镜像是分层的

镜像为啥分层？说这个问题不如想想来源，镜像分层是解决啥？

虽然镜像解决了打包，但是实际应用中我们的应用都是基于同一个rootfs来打包和迭代的，难道是每个rootfs都会多份吗？

为此docker利用了存储驱动AUFS，devicemapper，overlay，overlay2的存储技术实现了分层。初期是AUFS，到现在的overlay2驱动，不推荐devicemapper坑很多。例如一个nginx:alpine和python:alpine镜像可以从分层角度这样去理解

![](../.gitbook/assets/image%20%2870%29.png)

实际上只有不同的才占据存储空间，相同的是引用关系。抽象看镜像是一个实体，实际上是/var/lib/docker目录里的分层文件+一些json和db文件把层联系起来组成了镜像。存储路径是`/var/lib/docker/存储驱动类型/`

![](../.gitbook/assets/image%20%2812%29.png)

