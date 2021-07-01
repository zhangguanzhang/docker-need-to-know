# 使用 buildx 构建多平台 Docker 镜像

 `Docker 19.03` 引入了一个新的实验性插件，该插件使得跨平台构建 Docker 镜像比以往更加容易了。

 要想使用 `buildx`，首先要确保 Docker 版本不低于 `19.03`，同时还要通过设置环境变量 `DOCKER_CLI_EXPERIMENTAL` 来启用。可以通过下面的命令来为当前终端启用 buildx 插件：

```text
export DOCKER_CLI_EXPERIMENTAL=enabled
```

验证是否开启：

```text
$ docker buildx version
github.com/docker/buildx v0.3.0-5-g5b97415-tp-docker 5b974158f94f67d2bc072a8853fb280cbc600841
```

#### 启用 binfmt\_misc <a id="&#x542F;&#x7528;-binfmt_misc"></a>

 如果你使用的是 Docker 桌面版（MacOS 和 Windows），默认已经启用了 `binfmt_misc`，可以跳过这一步。

如果你使用的是 Linux，需要手动启用 `binfmt_misc`。大多数 Linux 发行版都很容易启用，不过还有一个更容易的办法，直接运行一个特权容器，容器里面写好了设置脚本：

```text
docker run --rm --privileged docker/binfmt:a7996909642ee92942dcd6cff44b9b95f08dad64
# 上面这个很久没人维护了，可以试试下面的
docker run --rm --privileged multiarch/qemu-user-static --reset -p yes

```

建议将 Linux 内核版本升级到 4.x 以上，特别是 CentOS 用户，你可能会遇到错误。

验证是 binfmt\_misc 否开启：

```text
$ ls -al /proc/sys/fs/binfmt_misc/
total 0
drwxr-xr-x 2 root root 0 Nov 10 10:24 .
dr-xr-xr-x 1 root root 0 Oct  8 14:16 ..
-rw-r--r-- 1 root root 0 Nov 10 10:25 qemu-aarch64
-rw-r--r-- 1 root root 0 Nov 10 10:25 qemu-arm
-rw-r--r-- 1 root root 0 Nov 10 10:25 qemu-ppc64le
-rw-r--r-- 1 root root 0 Nov 10 10:25 qemu-riscv64
-rw-r--r-- 1 root root 0 Nov 10 10:25 qemu-s390x
--w------- 1 root root 0 Nov 10 10:24 register
-rw-r--r-- 1 root root 0 Nov 10 10:24 status
```

验证是否启用了相应的处理器：

```text
$ cat /proc/sys/fs/binfmt_misc/qemu-aarch64
enabled
interpreter /usr/bin/qemu-aarch64
flags: OCF
offset 0
magic 7f454c460201010000000000000000000200b7
mask ffffffffffffff00fffffffffffffffffeffff
```

先创建一个新的构建器：

```text
$ docker buildx create --use --name test --platform linux/amd64,linux/arm64
```

查看构建器:

```text
$ docker buildx ls
NAME/NODE DRIVER/ENDPOINT             STATUS   PLATFORMS
test *    docker-container                     
  test0   unix:///var/run/docker.sock inactive linux/amd64, linux/arm64
default   docker                               
  default default                     running  linux/amd64, linux/386
```

启动构建器：

```text
$ docker buildx inspect test --bootstrap
[+] Building 12.4s (1/1) FINISHED                                                                                                                                                                                                           
 => [internal] booting buildkit                                                                                                                                                                                                       12.4s
 => => pulling image moby/buildkit:buildx-stable-1                                                                                                                                                                                    11.4s
 => => creating container buildx_buildkit_test0                                                                                                                                                                                        1.0s
Name:   test
Driver: docker-container

Nodes:
Name:      test0
Endpoint:  unix:///var/run/docker.sock
Status:    running
Platforms: linux/amd64, linux/arm64
```

查看当前使用的构建器及构建器支持的 CPU 架构，可以看到支持很多 CPU 架构：

```text
🐳 → docker buildx ls

NAME/NODE    DRIVER/ENDPOINT             STATUS  PLATFORMS
mybuilder *  docker-container
  mybuilder0 unix:///var/run/docker.sock running linux/amd64, linux/arm64, linux/ppc64le, linux/s390x, linux/386, linux/arm/v7, linux/arm/v6
default      docker
  default    default                     running linux/amd64, linux/386
```

#### 构建多平台镜像 <a id="&#x6784;&#x5EFA;&#x591A;&#x5E73;&#x53F0;&#x955C;&#x50CF;"></a>

`docker buildx build` 时候的 `--platform`后面架构参数必须小于等于构建器创建时的`--platform`，`--push`是推送，必须提前登录

```text
docker buildx build -t zhangguanzhang/keepalived:v2.0.20 . \
  --push \
  --platform linux/amd64,linux/arm64
```

如果想将构建好的镜像保存在本地，可以将 `type` 指定为 `docker`，但必须分别为不同的 CPU 架构构建不同的镜像，不能合并成一个镜像，即：

```text
$ docker buildx build -t zhangguanzhang/keepalived:v2.0.20 --platform=linux/arm -o type=docker .
$ docker buildx build -t zhangguanzhang/keepalived:v2.0.20 --platform=linux/arm64 -o type=docker .
$ docker buildx build -t zhangguanzhang/keepalived:v2.0.20 --platform=linux/amd64 -o type=docker .
```

## 参考

* [https://fuckcloudnative.io/posts/multiarch-docker-with-buildx/](https://fuckcloudnative.io/posts/multiarch-docker-with-buildx/)

