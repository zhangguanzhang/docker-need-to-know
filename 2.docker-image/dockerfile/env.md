# ENV

写法有两种，后者支持写多个，一般多个的话也是使用后者居多

```
ENV key value
-------
ENV key=value key2=value2
ENV key=value \
    key2=value2  \
    key3=value3   \
    key4=value4
```

设置一个环境变量，可以被 dockerfile 里后续的指令使用，也在容器运行过程中保持，支持的指令为:

```
ADD COPY ENV EXPOSE FROM LABEL STOPSIGNAL USER VOLUME WORKDIR ONBUILD
```

可以在 dockerhub 上发现各种官方镜像的 Dockerfile 的步骤都是固定了，新版本发布直接改下 ENV 后构建下即可

```
ENV NGINX_VERSION 1.15.11
RUN .... \
    && curl -fSL https://nginx.org/download/nginx-$NGINX_VERSION.tar.gz -o nginx.tar.gz \
    ...
    && cd /usr/src/nginx-$NGINX_VERSION \
	&& ./configure $CONFIG --with-debug \
	...
```

很多应用镜像启动都是先启动一个脚本，拼接一堆参数最终传递给应用的主进程当作参数，最常见的就是 tomcat，或者说很多的应用基于 tomcat。下面是之前我修改一个镜像 Dockerfile 摸索出的启动脚本的运行过程

```
+ '[' -r /opt/atlassian/jira/bin/setenv.sh ']'
+ . /opt/atlassian/jira/bin/setenv.sh


$ cat /opt/atlassian/jira/bin/setenv.sh
...
JAVA_OPTS="-Xms${JVM_MINIMUM_MEMORY} -Xmx${JVM_MAXIMUM_MEMORY} ${JVM_CODE_CACHE_ARGS} ${JAVA_OPTS} ${JVM_REQUIRED_ARGS} ${DISABLE_NOTIFICATIONS} ${JVM_SUPPORT_RECOMMENDED_ARGS} ${JVM_EXTRA_ARGS} ${JIRA_HOME_MINUSD} ${START_JIRA_JAVA_OPTS}"
...
export JAVA_OPTS
...
exec java $JAVA_OPTS
```

其中有一行：

```
JAVA_OPTS="... ${JAVA_OPTS} ..."
```

我们知道了他拼接了自己，所以我们想给java最终的参数添加的固定参数的话我们可以在构建镜像声明 JAVA\_OPTS，例如添加时区我们应该在Dockerfile里设置

```
ENV  JAVA_OPTS='-Duser.timezone=GMT+08'
```

docker run 有选可以指定 env，ENV 指令不一定是给 Dockerfile 用的，有时候是给容器启动时候用的，我们可以在 docker run 的时候指定 env 或者覆盖 env 达到不需要修改镜像，例如常见的后端需要连接一个 mysql，可以在后端代码 `os.getEnv("mysql_address")`，我们启动的时候指定 mysql\_address 变量为真实的 mysql 地址即可。

### --env-file

命令行或者 api 的 env-file 这个有个坑，文件是按照`=`去 split 的，就是说值不能带引号，引号也会传到值里去，否则会错误

```
$ cat .env 
FOO="bar"
$ docker run --rm --env-file .env $IMG sh -c 'echo $FOO'
"bar"

$ cat .env2
FOO=bar
$ docker run --rm --env-file .env $IMG sh -c 'echo $FOO'
bar
```

### 命令规范

`-e` `--env-file`   `docker-compose`  `k8s` 之类的，无论哪种方式设置环境变量，环境变量命名遵循着以下规则：

```
1. 第一个不能是数字
2. 由数字，26个大小写字母和下划线_组成
```

少数人可能用带横线的命名（部分os上是可以读取到的），但是部分 os 上无法读取到，所以要遵循命令
