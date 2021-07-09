# WORKDIR

声明后续指令的工作目录，目录不存在则创建，可以理解为`mkdir -p <dir> && cd <dir>`

```text
WORKDIR /path/to/workdir
```

 可以在a中多次使用`Dockerfile`。如果提供了相对路径，则它将相对于前一条`WORKDIR`指令的路径 。例如：

```text
WORKDIR /a
WORKDIR b
WORKDIR c
RUN pwd
```

 最终`pwd`命令的输出`Dockerfile`将是 `/a/b/c`是

