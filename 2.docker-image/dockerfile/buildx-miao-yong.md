# buildx 妙用

### 直接构建成果到宿主机上

下面是我最近折腾的 fio 编译，编译给 arm64 使用：

```
FROM ubuntu as build
WORKDIR /opt
ARG VER=fio-3.29
#ARG DEBIAN_FRONTEND=noninteractive
RUN if [  -e /etc/apt/sources.list ];then sed -ri 's/[a-zA-Z0-9.]+(debian.org|ubuntu.com)/mirrors.aliyun.com/g' /etc/apt/sources.list; fi && \
    export DEBIAN_FRONTEND=noninteractive && \
    apt-get update && \
    apt-get install -y git gcc make cmake libaio1 libaio-dev zlib1g zlib1g-dev && \
    git clone https://github.com/axboe/fio.git && \
    cd fio  && \
    git checkout ${VER} && \
    ./configure --build-static && \
    make && make install  && \
    cp `which fio` /fio

FROM scratch AS bin
COPY --from=build /fio /fio
```

编译命令：

```
docker buildx build --platform linux/arm64  --target bin   --output .   .
```

最后一阶段用 scratch 然后 `--target` 和 `--output`&#x20;

```
$ ll
total 8308
-rw-r--r-- 1 root root     657 Jan 24 03:19 Dockerfile
-rwxr-xr-x 1 root root 8502760 Jan 24 03:30 fio
$ ldd fio
	not a dynamic executable
$ file fio
fio: ELF 64-bit LSB executable, ARM aarch64, version 1 (GNU/Linux), statically linked, BuildID[sha1]=6a644c0696153f99d2d527948e0400720742a002, for GNU/Linux 3.7.0, not stripped
```
