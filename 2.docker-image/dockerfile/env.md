# ENV

写法有两种，后者支持写多个，一般多个的话也是使用后者居多

```text
ENV key value
-------
ENV key=value key2=value2
ENV key=value \
    key2=value2  \
    key3=value3   \
    key4=value4
```

设置一个环境变量，可以被dockerfile里后续的指令使用，也在容器运行过程中保持，支持的指令为:

```text
ADD COPY ENV EXPOSE FROM LABEL STOPSIGNAL USER VOLUME WORKDIR ONBUILD
```

可以在dockerhub上发现各种官方镜像的Dockerfile的步骤都是固定了，新版本发布直接改下ENV后构建下即可

```text
ENV NGINX_VERSION 1.15.11
RUN .... \
    && curl -fSL https://nginx.org/download/nginx-$NGINX_VERSION.tar.gz -o nginx.tar.gz \
    ...
    && cd /usr/src/nginx-$NGINX_VERSION \
	&& ./configure $CONFIG --with-debug \
	...
```

很多应用镜像启动都是先启动一个脚本，拼接一堆参数最终传递给应用的主进程当作参数，最常见的就是tomcat，或者说很多的应用基于tomcat。下面是之前我修改一个镜像Dockerfile摸索出的启动脚本的运行过程

```text
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

```text
JAVA_OPTS="... ${JAVA_OPTS} ..."
```

我们知道了他拼接了自己，所以我们想给java最终的参数添加的固定参数的话我们可以在构建镜像声明JAVA\_OPTS，例如添加时区我们应该在Dockerfile里设置

```text
ENV  JAVA_OPTS='-Duser.timezone=GMT+08'
```

docker run 有选可以指定env，ENV指令不一样是给Dockerfile用的，有时候是给容器启动时候用的，我们可以在docker run的时候指定env或者覆盖env达到不需要修改镜像，例如常见的后端需要连接一个mysql，可以在后端代码os.getEnv\("mysql\_address"\)，我们启动的时候指定mysql\_address变量为真实的mysql地址即可。

### --env-file

命令行或者api的env-file这个有个坑，文件是按照`=`去split的，就是说值不能带引号，引号也会传到值里去，否则会错误

```text
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

```text
1. 第一个不能是数字
2. 由数字，26个大小写字母和下划线_组成
```

少数人可能用带横线的命名（部分os上是可以读取到的），但是部分 os 上无法读取到，所以要遵循命令

