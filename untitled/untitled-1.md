# 为什么docker搞了个镜像的东西

 前面说到了cgroup和namespaces可以隔离，实际上不用docker也可以做到沙箱的概念，例如centos7或者ubuntu下systemd为pid1的发行版上使用systemd可以为进程设置隔离和资源限制，这里不多说，有兴趣可以去看相关资料 [https://www.imooc.com/article/72502](https://www.imooc.com/article/72502) [http://0pointer.de/blog/projects/resources.html](http://0pointer.de/blog/projects/resources.html)

在以前很火的Cloud Foundry的Pass项目就是打包和分发,用户利用cli推送应用（包含了启动执行的文件和启动脚本），runner的机器上有一个agent拉取到用户的推送然后run起来。但是每个机器上都会跑不同用户的应用，底层利用cgroups和namespaces隔离成一个沙盒,这就是所谓的容器。

而 Docker 项目，实际上跟 Cloud Foundry 的容器并没有太大不同，所以在它发布后不久，Cloud Foundry 的首席产品经理 James Bayer 就在社区里做了一次详细对比，告诉用户 Docker 实际上只是一个同样使用 Cgroups 和 Namespace 实现的“沙盒”而已，没有什么特别的黑科技，也不需要特别关注。

然而，短短几个月，Docker 项目就迅速崛起了。主要原因就是docker镜像。

一旦用上了 PaaS，用户就必须为每种语言、每种框架，甚至每个版本的应用维护一个打好的包。这个打包过程，没有任何章法可循，更麻烦的是，明明在本地运行得好好的应用，却需要做很多修改和配置工作才能在 PaaS 里运行起来。而这些修改和配置，并没有什么经验可以借鉴，基本上得靠不断试错，直到你摸清楚了本地应用和远端 PaaS 匹配的“脾气”才能够搞定。

在以前的传统运输行业也有类似的问题，货主与承运方都会担心因货物类型的不同而导致损失，比如几个铁桶错误地压在了一堆香蕉上。另一方面，运输过程中需要使用不同的交通工具也让整个过程痛苦不堪：货物先装上车运到码头，卸货，然后装上船，到岸后又卸下船，再装上火车，到达目的地，最后卸货。一半以上的时间花费在装、卸货上，而且搬上搬下还容易损坏货物。

![](../.gitbook/assets/image%20%2852%29.png)

在集装箱的发明后问题得到解决

![](../.gitbook/assets/image%20%2848%29.png)

Docker 将集装箱思想运用到软件打包上，为代码提供了一个基于容器的标准化运输系统。Docker 可以将任何应用及其依赖打包成一个轻量级、可移植、自包含的容器。容器可以运行在几乎所有的操作系统上。

![](../.gitbook/assets/image%20%2828%29.png)

docker的logo上面就是一堆集装箱

![](../.gitbook/assets/image%20%2860%29.png)

针对hello-world的输出信息我们可以得到docker的架构图

![](../.gitbook/assets/image%20%2818%29.png)

在系统层面对比容器，操作系统分为内核+rootfs，rootfs分为很多，常见的centos，ubuntu，alpine，debian

![](../.gitbook/assets/image%20%2856%29.png)

