# exec与entrypoint使用脚本

其实现在很多有状态的官方镜像的ENTRYPOINT都是使用了一个脚本。例如redis

```text
COPY docker-entrypoint.sh /usr/local/bin/
ENTRYPOINT ["docker-entrypoint.sh"]

EXPOSE 6379
CMD ["redis-server"]
```

```text
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

第一个if的逻辑是如果`docker run -some_option redis镜像名` 后面接-x 或者--xx或者xxx.conf，就把脚本收到的$@改成

```text
redis-server 脚本收到的$@
```

例如我们可用修改容器里的redis端口。

```text
docker run -d -p 7379:7379 redis --port 7379
```

如果我们传入的command不是`-`开头的也不是`.conf`结尾的字符，例如是date，则会跑到最后的逻辑执行我们的date命令不会启动redis-server

第二个if这里，如果满足第一个if或者直接默认的cmd下而且容器里用户uid是0，则把属主不是redis的文件改成redis用户，然后切成redis用户去启动redis-server。

我们可以看到entrypoint能在业务进程启动前做很多事情。而且优秀的镜像都离不开entrypoint脚本，能够根据用户传入的变量和command来切换启动的场景和配置。

前面说了，主进程一定要是业务进程，这里怎么是个脚本呢，那业务进程不就不是pid为1了吗？ 这里用了exec来退位让贤，最终redis-server还是pid为1的。可以简单几个命令讲解下exec的作用。

写个test.sh脚本，在脚本里用pstree -p，运行脚本bash test.sh查看进程层次

![](../../.gitbook/assets/image%20%282%29.png)

发现pstree是在我们脚本bash\(17003\)的子进程

然后在脚本最后面加一行exec pstree -p看看输出

![](../../.gitbook/assets/image%20%2823%29.png)

我们发现bash进程运行的时候pid是17030，然后第二个pstree上升到了17030这一层次了，假设pid为a的命令或者二进制exec执行了命令b，那b就接替了a的pid。

如果说我们entrypoint或者cmd使用脚本，那么我们一定要在脚本最后启动业务进程的时候前面加个exec让脚本退位让贤，让业务进程接管pid为1的角色，业务进程退出容器就退出。

最后环境变量写配置文件涉及到修改，还有一些判断是否初次启动的，有下面一些工具或者套路

* xmlstarlet 处理 xml
* pip 安装 shyaml 处理yaml，当然还是推荐[https://github.com/mikefarah/yq](https://github.com/mikefarah/yq) 的二进制文件处理 yaml 文件不依赖python环境
* jq 读取 json
* envsubst &lt; xxx.template &gt; xxx.conf 直接把模板文件里的shell变量从env渲染
* jsonnet 生成 yaml 或者 json
* nodejs 的 npm 安装 json 可以修改 json 文件
* 处理 excel 或者 csv 使用 in2csv，csvkit 提供了 in2csv，csvcut，csvjoin，csvgrep
* \`touch -d "@0"\`写在构建的最后一个RUN里把时间戳设置为1970-1-1，然后用 stat 命令判断是否初次启动
* ```text
  if [ "$(stat -c "%Y" "${CONF_INSTALL}/conf/server.xml")" -eq "0" ]; then
  ```
* 另外 entrypoint 脚本 COPY 进去的时候注意可执行权限，如果 Windows 上传到 Linux 构建会因为 entrypoint 脚本没带权限无法运行

