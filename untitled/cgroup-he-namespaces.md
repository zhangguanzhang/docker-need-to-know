---
description: docker的隔离是利用的cgroup和namespace，这里仅仅是简单讲解下cgroup和namespace
---

# cgroup和namespaces

## cgroups: cpu, 内存, io,网络的分配和限制

首先测试机器用的centos7.6已经安装过docker，htop看是2核2g

![](../.gitbook/assets/image%20%2862%29.png)

打开一个终端运行下面脚本

```text
cat<<EOF >cpu.sh
x=0
while [ True ];do
    x=$x+1
done;
EOF
bash cpu.sh
```

再开一个终端使用htop发现吃满cpu的就是它

![](../.gitbook/assets/image%20%2847%29.png)

细心点会发现它在运行时候不一定一直被调度到同一核上

![](../.gitbook/assets/image%20%2816%29.png)

ctrl+c停掉脚本后安装cgroup配置工具

```text
yum install -y libcgroup-tools libcgroup
```

要注意的是如果已经安装过docker，docker会创建一个cgroup的挂载目录/sys/fs/cgroup/ ,我们创建的cgroup组也会在里面

```text
$ mount -t cgroup
cgroup on /sys/fs/cgroup/systemd type cgroup (rw,nosuid,nodev,noexec,relatime,xattr,release_agent=/usr/lib/systemd/systemd-cgroups-agent,name=systemd)
cgroup on /sys/fs/cgroup/perf_event type cgroup (rw,nosuid,nodev,noexec,relatime,perf_event)
cgroup on /sys/fs/cgroup/rdma type cgroup (rw,nosuid,nodev,noexec,relatime,rdma)
cgroup on /sys/fs/cgroup/cpu,cpuacct type cgroup (rw,nosuid,nodev,noexec,relatime,cpu,cpuacct)
cgroup on /sys/fs/cgroup/hugetlb type cgroup (rw,nosuid,nodev,noexec,relatime,hugetlb)
cgroup on /sys/fs/cgroup/memory type cgroup (rw,nosuid,nodev,noexec,relatime,memory)
cgroup on /sys/fs/cgroup/net_cls,net_prio type cgroup (rw,nosuid,nodev,noexec,relatime,net_cls,net_prio)
cgroup on /sys/fs/cgroup/blkio type cgroup (rw,nosuid,nodev,noexec,relatime,blkio)
cgroup on /sys/fs/cgroup/freezer type cgroup (rw,nosuid,nodev,noexec,relatime,freezer)
cgroup on /sys/fs/cgroup/cpuset type cgroup (rw,nosuid,nodev,noexec,relatime,cpuset)
cgroup on /sys/fs/cgroup/pids type cgroup (rw,nosuid,nodev,noexec,relatime,pids)
cgroup on /sys/fs/cgroup/devices type cgroup (rw,nosuid,nodev,noexec,relatime,devices)
```

创建一个cgroup分组

```text
cgcreate -g cpu,cpuset:my1
```

设置限制使用cpu 1

```text
cgset -r cpuset.cpus=1 my1
cgset -r cpuset.mems=0 my1
```

设置限制cpu最多用到50%

```text
cgset -r cpu.cfs_period_us=100000  my1
cgset -r cpu.cfs_quota_us=50000  my1
```

使用上面创建的cg控制组运行脚本

```text
cgexec  -g cpuset,cpu:/my1 bash cpu.sh
```

![](../.gitbook/assets/image%20%2828%29.png)

我们发现cpu被限制在第二个核心上一直50%\(ps:htop显示的index是从1开始，cgset是从0开始\)

我们可以看看cgroup的path下

```text
$ ll /sys/fs/cgroup/{,cpu\,cpuacct/my1}
/sys/fs/cgroup/:
total 0
dr-xr-xr-x 4 root root  0 Apr 15 18:45 blkio
lrwxrwxrwx 1 root root 11 Apr 15 18:14 cpu -> cpu,cpuacct
lrwxrwxrwx 1 root root 11 Apr 15 18:14 cpuacct -> cpu,cpuacct
dr-xr-xr-x 5 root root  0 Apr 15 18:45 cpu,cpuacct
dr-xr-xr-x 3 root root  0 Apr 15 18:45 cpuset
dr-xr-xr-x 4 root root  0 Apr 15 18:45 devices
dr-xr-xr-x 2 root root  0 Apr 15 18:45 freezer
dr-xr-xr-x 2 root root  0 Apr 15 18:45 hugetlb
dr-xr-xr-x 4 root root  0 Apr 15 18:45 memory
lrwxrwxrwx 1 root root 16 Apr 15 18:14 net_cls -> net_cls,net_prio
dr-xr-xr-x 2 root root  0 Apr 15 18:45 net_cls,net_prio
lrwxrwxrwx 1 root root 16 Apr 15 18:14 net_prio -> net_cls,net_prio
dr-xr-xr-x 2 root root  0 Apr 15 18:45 perf_event
dr-xr-xr-x 4 root root  0 Apr 15 18:45 pids
dr-xr-xr-x 2 root root  0 Apr 15 18:45 rdma
dr-xr-xr-x 4 root root  0 Apr 15 18:45 systemd

/sys/fs/cgroup/cpu,cpuacct/my1:
total 0
-rw-rw-r-- 1 root root 0 Apr 15 18:31 cgroup.clone_children
-rw-rw-r-- 1 root root 0 Apr 15 18:31 cgroup.procs
-r--r--r-- 1 root root 0 Apr 15 18:31 cpuacct.stat
-rw-rw-r-- 1 root root 0 Apr 15 18:31 cpuacct.usage
-r--r--r-- 1 root root 0 Apr 15 18:31 cpuacct.usage_all
-r--r--r-- 1 root root 0 Apr 15 18:31 cpuacct.usage_percpu
-r--r--r-- 1 root root 0 Apr 15 18:31 cpuacct.usage_percpu_sys
-r--r--r-- 1 root root 0 Apr 15 18:31 cpuacct.usage_percpu_user
-r--r--r-- 1 root root 0 Apr 15 18:31 cpuacct.usage_sys
-r--r--r-- 1 root root 0 Apr 15 18:31 cpuacct.usage_user
-rw-rw-r-- 1 root root 0 Apr 15 18:31 cpu.cfs_period_us
-rw-rw-r-- 1 root root 0 Apr 15 18:31 cpu.cfs_quota_us
-rw-rw-r-- 1 root root 0 Apr 15 18:31 cpu.rt_period_us
-rw-rw-r-- 1 root root 0 Apr 15 18:31 cpu.rt_runtime_us
-rw-rw-r-- 1 root root 0 Apr 15 18:31 cpu.shares
-r--r--r-- 1 root root 0 Apr 15 18:31 cpu.stat
-rw-rw-r-- 1 root root 0 Apr 15 18:31 notify_on_release
-rw-rw-r-- 1 root root 0 Apr 15 18:34 tasks
```

实际上不一定得用命令创建cgroup组，我们手动按照目录结构创建也可以

清理我们创建的cgroup组

```text
cgdelete cpu,cpuset:my1
```

这里只是简单介绍下，cgroup的控制能力不单单是上面这个例子这么点，单独对cgroup有兴趣的可以网上找文章看看

docker run 命令资源限制的一些文章

{% embed url="https://blog.csdn.net/u010472499/article/details/52994517" %}

{% embed url="https://blog.csdn.net/weixin\_43397326/article/details/83587431" %}

### namespaces

* Mount \(mnt Linux 2.4.19\): 类似chroot,但是更安全
* UTS \(Linux 2.6.19\):允许每个容器有自己的hostname和dns domain
* IPC \(Interprocess Communication Linux 2.6.19\):实现进程间通信\(IPC\)资源、System V IPC对象、POSIX消息队列的隔离
* PID \(Process ID Linux 2.6.24\): 进程之间可视的隔离,第一个进程pid为1,其他进程会被附加到它下面,pid为1的进程决定容器的生存周期
* NET \(Network Linux 2.6.24-2.6.29\):每个命名空间都有一组私有IP地址，自己的路由表，套接字列表，连接跟踪表，防火墙和其他与网络相关的资源
* UID\(User ID Linux 2.6.23-3.8\): 让进程在ns内部拥有root权限但是在外只是普通权限

