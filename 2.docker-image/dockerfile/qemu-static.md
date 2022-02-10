# 不升级内核使用qemu-static构建其他架构镜像



​docker buildx 需要内核支持，如果机器不好升级内核，可以尝试下面的方法：

1. 先注册 binfmt，此步是必须的前提

```
docker run --rm --privileged multiarch/qemu-user-static:register --reset
```

&#x20;2\. 使用 qemu-static 运行 arm64 的镜像：

```
$ wget https://github.com/multiarch/qemu-user-static/releases/download/v6.1.0-8/qemu-aarch64-static.tar.gz
$ tar zxf qemu-aarch64-static.tar.gz
$ docker run --rm -t arm64v8/ubuntu uname -m
standard_init_linux.go:211: exec user process caused "no such file or directory"

$ docker run --rm -t -v $PWD/qemu-aarch64-static:/usr/bin/qemu-aarch64-static arm64v8/ubuntu uname -m
aarch64
```

构建镜像可以：

```
FROM arm64v8/ubuntu
ADD qemu-aarch64-static.tar.gz /usr/bin
RUN xxx
```

但是上面依赖本地文件，实际上官方提供了一个单独的镜像，我们可以：

```
FROM multiarch/qemu-user-static:x86_64-aarch64 as qemu
FROM arm64v8/ubuntu
COPY --from=qemu /usr/bin/qemu-aarch64-static /usr/bin/
RUN xxxx
```

上面就是不升级内核情况下在 x86\_64 上构建 arm64 的镜像步骤了

## 参考

&#x20;\- [https://github.com/multiarch/qemu-user-static](https://github.com/multiarch/qemu-user-static)
