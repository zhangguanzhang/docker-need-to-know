# bridge

不指定容器网络的时候默认就是bridge，这里我懒得写了，直接去别人博客看吧

{% embed url="https://www.cnblogs.com/CloudMan6/p/7066851.html" %}

当Docker进程启动时，会在主机上创建一个名为docker0的虚拟网桥，此主机上启动的Docker容器会连接到这个虚拟网桥上。虚拟网桥的工作方式和物理交换机类似，这样主机上的所有容器就通过交换机连在了一个二层网络中。

从docker0子网中分配一个IP给容器使用，并设置docker0的IP地址为容器的默认网关。在主机上创建一对虚拟网卡veth pair设备，Docker将veth pair设备的一端放在新创建的容器中，并命名为eth0（容器的网卡），另一端放在主机中，以vethxxx这样类似的名字命名，并将这个网络设备加入到docker0网桥中。可以通过brctl show命令查看。

bridge模式是docker的默认网络模式，不写--net参数，就是bridge模式。使用docker run -p时，docker实际是在iptables做了DNAT规则，实现端口转发功能。可以使用iptables -t nat -vnL查看。

bridge模式如下图所示：

![](../.gitbook/assets/image%20%2842%29.png)

### 容器访问外网：

我们用之前的容器ping下114

![](../.gitbook/assets/image%20%2846%29.png)

我们可以通过nf\_contrack查看到nat记录表

![](../.gitbook/assets/image%20%288%29.png)

实际上容器的ip段被docker添加了snat规则

![](../.gitbook/assets/image%20%2841%29.png)

这条规则的意思是说，源ip是172.17.0.0/16的包，在经过路由后不是从docker0网桥出去的（容器访问外网会走默认路由，那路由过后也就是我们机器的网卡ens33）做snat，出去的ip为出去网卡（ens33）的ip

![](../.gitbook/assets/image%20%2831%29.png)

流程为下图，引用下cloudman的图（他宿主机网卡名为enp0s3，网卡ip和我不一样）

![](../.gitbook/assets/image%20%2857%29.png)

### 外面访问容器：

Linux的端口要被外部访问那这台机器上一定有进程bind这个端口，我们开个-p 80:80的nginx看看是哪个进程bind住80端口的

![](../.gitbook/assets/image%20%2822%29.png)

我们发现bind端口的是docker-proxy，查看它的进程参数也一目了然。也就是说实际上不映射端口我们也可以在宿主机上访问容器ip和端口即可访问他们的服务，也就是下面的图，引用下别人的图

![](../.gitbook/assets/image%20%2840%29.png)

最后也可以自定义另一个网络就不说了，详细去看 [https://www.cnblogs.com/CloudMan6/p/7077198.html](https://www.cnblogs.com/CloudMan6/p/7077198.html)

