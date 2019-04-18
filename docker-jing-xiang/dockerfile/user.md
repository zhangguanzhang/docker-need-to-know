# USER

两种写法

```text
USER <user>[:<group>] or
USER <UID>[:<GID>]
```

USER 指令和 WORKDIR 相似，都是改变环境状态并影响以后的层。WORKDIR 是改变工作目录，USER 则是改变之后层的执行 RUN, CMD 以及 ENTRYPOINT 这类命令的身份。

当然，和 WORKDIR 一样，USER 只是帮助你切换到指定用户而已，这个用户必须是事先建立好的，否则无法切换。可用可不用。

不用的情况建议给容器的最终进程指定用户去运行，例如nginx官方添加了一个不登陆的nginx用户，配置文件里指定使用这个用户运行nginx。

