# CMD

设置镜像运行出来的容器的缺省命令

有两种写法，写多个和FROM一个已经有CMD的镜像的话，以最后一个为准

```text
CMD ["executable", "param1", "param2"] 
CMD command param1 param2 
```

前者是exec格式也是推荐格式，后者是/bin/sh格式，exec和CMD还有ENTRYPOINT这三者之间联系非常紧密，后面单独将相关的知识点。这里先用一个例子讲/bin/sh格式啥意思

![](../../.gitbook/assets/image%20%2813%29.png)

我们发现pid为1的是一个/bin/sh的进程，而我们的进程在容器里在后面。容器是单独一个pid namespaces的。这里懒得去做个图了，借用下别人的图

![](../../.gitbook/assets/image%20%281%29.png)

默认下所有进程在一个顶级的pid namespaces里，pid namespaces像一个树一样。从根到最后可以多级串。容器的pid namespaces实际上是在宿主机上能看到的，也就是下面，我们可以看到容器在宿主机上的进程，由于子namespaces无法看到父级的namespaces，所以容器里第一个进程\(也就是cmd\)认为自己是pid为1，容器里其余进程都是它的子进程。

![](../../.gitbook/assets/image%20%285%29.png)



![](../../.gitbook/assets/image%20%2825%29.png)

先看下docker run命令格式

```text
$ docker run --help
Usage:	docker run [OPTIONS] IMAGE [COMMAND] [ARG...]
```

