# ENTRYPOINT

和CMD用法也一样两种格式，唯一要注意的就是区别

`CMD` 和`ENTRYPOINT`只有一个或者两者都有都可以，容器最终运行的命令为

**&lt;ENTRYPOINT&gt; &lt;CMD&gt;**

例如我们做个测试

![](../../.gitbook/assets/image%20%2814%29.png)

alpine 的 root 目录是没有文件的，所以`ls /root`没有输出，我们用选项去覆盖住 entrypoint 可以看到输出了 date。注意一点是覆盖 entrypoint 的时候镜像的 CMD 会被忽略，我们真要调试的时候需要加command 的话，可以在docker run的镜像后面加 command 和 arg。

上面例子可以很形象的证明了是这个关系，最终运行的是**&lt;ENTRYPOINT&gt; &lt;CMD&gt;**，同时不光在docker run的时候覆盖掉 CMD，也可以覆盖掉默认的 entrypoint。很多时候我们可以主进程 bash 或者 sh 进去手动启动看看。老版本接触不多，不确定老版本有没有`--entrypoint`的选项。

最后如果是/bin/sh 的 entrypoint 会忽略掉 CMD 和 `docker run` 的 command 参数。

不要在 `ENTRYPOINT`里用变量，要么你的命令放镜像`PATH`里，要么放根目录：

```text
COPY myapp /usr/local/bin/myapp
ENTRYPOINT ["myapp"]
# or this
COPY myapp /myapp
ENTRYPOINT ["/myapp"]
```

