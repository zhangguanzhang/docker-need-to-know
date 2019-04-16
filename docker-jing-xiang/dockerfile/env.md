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

