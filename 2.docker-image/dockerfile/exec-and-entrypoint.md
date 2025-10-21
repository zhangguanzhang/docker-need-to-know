# exec与entrypoint使用脚本

其实现在很多有状态的官方镜像的ENTRYPOINT都是使用了一个脚本。例如redis

```
COPY docker-entrypoint.sh /usr/local/bin/
ENTRYPOINT ["docker-entrypoint.sh"]

EXPOSE 6379
CMD ["redis-server"]
```

```
#!/bin/sh
set -e

# first arg is `-f` or `--some-option`
# or first arg is `something.conf`
if [ "${1#-}" != "$1" ] || [ "${1%.conf}" != "$1" ]; then
	set -- redis-server "$@"
fi

# allow the container to be started with `--user`
if [ "$1" = 'redis-server' -a "$(id -u)" = '0' ]; then
	find . \! -user redis -exec chown redis '{}' +
	exec gosu redis "$0" "$@"
fi

exec "$@"
```

最终运行的是`docker-entrypoint.sh  redis-server`

第一个if的逻辑是如果`docker run -some_option redis镜像名` 后面接 `-x ` 或者 `--xx` 或者 `xxx.conf`，就把脚本收到的 `$@` 改成

```
redis-server 脚本收到的$@
```

例如我们可用修改容器里的redis端口。

```
docker run -d -p 7379:7379 redis --port 7379
```

如果我们传入的 command 不是`-`开头的也不是`.conf`结尾的字符，例如是date，则会跑到最后的逻辑执行我们的 date 命令不会启动 redis-server

第二个 if 这里，如果满足第一个if或者直接默认的 cmd 下而且容器里用户 uid 是 0，则把属主不是 redis 的文件改成 redis 用户，然后切成 redis 用户去启动 redis-server。

我们可以看到 entrypoint 能在业务进程启动前做很多事情。而且优秀的镜像都离不开 entrypoint 脚本，能够根据用户传入的变量和 command 来切换启动的场景和配置。

前面说了，主进程一定要是业务进程，这里怎么是个脚本呢，那业务进程不就不是 pid 为1了吗？ 这里用了exec 来退位让贤，最终 redis-server 还是 pid 为1的。可以简单几个命令讲解下exec的作用。

写个test.sh脚本，在脚本里用pstree -p，运行脚本bash test.sh查看进程层次

![](<../../.gitbook/assets/image (2).png>)

发现 pstree 是在我们脚本 bash(17003) 的子进程

然后在脚本最后面加一行 exec pstree -p 看看输出

![](<../../.gitbook/assets/image (23).png>)

我们发现 bash 进程运行的时候pid是17030，然后第二个 pstree 上升到了 17030 这一层次了，假设 pid 为 a 的命令或者二进制 exec 执行了命令 b，那 b 就接替了 a 的 pid。

如果说我们 entrypoint 或者 cmd 使用脚本，那么我们一定要在脚本最后启动业务进程的时候前面加个exec 让脚本退位让贤，让业务进程接管pid为1的角色，业务进程退出容器就退出。

最后环境变量写配置文件涉及到修改，还有一些判断是否初次启动的，有下面一些工具或者套路

* xmlstarlet 处理 xml
* pip 安装 shyaml 处理yaml，当然还是推荐[https://github.com/mikefarah/yq](https://github.com/mikefarah/yq) 的二进制文件处理 yaml 文件不依赖python环境
* jq 读取 json，yq golang版本更强大的 jq ，支持修改和读取 json&#x20;
* gettext 的envsubst 可以通过 envsubst < xxx.template > xxx.conf 直接把模板文件里的shell变量从env渲染，也有个 golang 版本的，但是我没使用过
* jsonnet 生成 yaml 或者 json
* nodejs 的 npm 安装 json 可以修改 json 文件
* 处理 excel 或者 csv 使用 in2csv，csvkit 提供了 in2csv，csvcut，csvjoin，csvgrep
* \`touch -d "@0"\`写在构建的最后一个RUN里把时间戳设置为1970-1-1，然后用 stat 命令判断是否初次启动
* ```
  if [ "$(stat -c "%Y" "${CONF_INSTALL}/conf/server.xml")" -eq "0" ]; then
  ```
* 另外 entrypoint 脚本 COPY 进去的时候注意可执行权限，如果 Windows 上传到 Linux 构建会因为 entrypoint 脚本没带权限无法运行

# 过滤出所有的容器内 pid1 是脚本的容器

20250521 ，发现新业务组上线后有部分容器内部 pid1 的是 shell 脚本而不能转发信号导致无法优雅下线和自动重启，人工排查是不可能的，打算写脚本过滤，发现 `docker top` 展示的并不是按照进程运行时间顺序展示，而是按照 pid 数字排序，而 Linux 里 pid 数字分配是系统分配的，默认从小递增，到 max 后又回来，此刻 docker top 第一行就不是容器内 pid 为1的了，所以使用 docker inspect 取容器内 pid 为1。

```shell
function t(){
    docker ps --format '{{.ID}}' | while read ctr_id;do
        ctr_pid1=`docker inspect ${ctr_id} --format '{{ .State.Pid }}'`
        # restarting pid will be 0
        if [ "$ctr_pid1" -eq 0 ];then
            continue
        fi
        cmd=`ps  -p $ctr_pid1 -o cmd=`
        if [ -n "$cmd" ] && echo "$cmd" | grep -Eq '(sh|bash)$' ;then
            docker ps -a --filter id=${ctr_id} --format '{{.Names}}'
        fi
    done
}

t
```

起测试容器可以：

```
$ cat > test.sh << EOF
#!/bin/bash
sleep 10000
EOF
$ chmod a+x test.sh
$ docker run --rm -d -ti --entrypoint /w/test.sh -v $PWD:/w debian:11
$ docker ps -a
CONTAINER ID   IMAGE          COMMAND        CREATED          STATUS          PORTS     NAMES
a147179ba730   9df6d6105df2   "/w/test.sh"   11 minutes ago   Up 11 minutes             admiring_shtern
```

