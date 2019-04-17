# USER

两种写法

```text
USER <user>[:<group>] or
USER <UID>[:<GID>]
```

指定后面指令和镜像起来的容器的运行用户。可用可不用。

不用的情况建议给容器的最终进程指定用户去运行，例如nginx官方添加了一个不登陆的nginx用户，配置文件里指定使用这个用户运行nginx。

