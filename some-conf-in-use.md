# 8.一些使用和配置问题

## docker pull x509:certificate signed by unknown authority <a href="#articlecontentid" id="articlecontentid"></a>

```
domain=xxx
port=443

mkdir -p /etc/docker/certs.d/${domain}
openssl s_client -showcerts -connect ${domain}:${port} < /dev/null | \
  sed -ne '/-BEGIN CERTIFICATE-/,/-END CERTIFICATE-/p' > /etc/docker/certs.d/${domain}/ca.crt
  
```

### 根据进程pid或者端口查容器名字

```
$ netstat -nlpt | grep 10080
tcp        0      0 0.0.0.0:10080           0.0.0.0:*               LISTEN      11751/nginx: master
# 列出 pid 11751的进程树
$ pstree -sp 11751
systemd(1)───dockerd(10383)───containerd(10480)───containerd-shim(11018)───bash(11079)───nginx(11751)───nginx(23483)
# 列出containerd-shim(11018)的启动参数
$ ps aux | grep 1101[8]
root     11018  0.0  0.0 108716  7252 ?        Sl   Jun16   0:48 containerd-shim -namespace moby -workdir \
    /data/kube/docker/containerd/daemon/io.containerd.runtime.v1.linux/moby/fc667670478635e6e71f74e5435f4998e5510ae2705ea44210863ff1720fa942 \
     -address /var/run/docker/containerd/containerd.sock -containerd-binary /data/kube/bin/containerd -runtime-root /var/run/docker/runtime-runc
# 拿 moby/后几个字符串来查，不要全部字符串
$ docker ps -a | grep fc66
fc6676704786     treg.xxxx.xxx.cn/xxxxxx/ftp_nginx:xxx   "/bin/ba…"   2 weeks ago     Up 13 days           ftp_nginx
```

上面比如有可疑进程，确定进程不是宿主机的 path，可以上面的步骤 pstree -sp 开始查到容器的名字。

### 判断是否在容器内

`/proc/1/sched`的第一行会显示真实的 pid，所以可以靠这个来判断是否是在容器内

```
if [ `head -n1 /proc/1/sched | cut -d \( -f2 | cut -d, -f1` -eq 1 ];then
    echo not in container
else
    echo in container
fi
```

### 一次复制多个文件到容器里

```
tar -c $(ls | grep -v "^(ui\|ui-v2\|website\|bin\|pkg\|.git)") \
    | docker cp - ${container_id}:/consul
```

### 查看 veth 的 peer

这里我以 k8s 的做示例，docker 的也一样，k8s 现在 cni-plugins 标准，k8s 的容器都是在 `cni-plugins` 下桥接在 `cni0` 上的

```
# 取容器 pid
$ docker inspect d30 | grep -m1 -i pid
            "Pid": 9079,

$ nsenter --net --target 9079 ip a s 
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
3: eth0@if210: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1450 qdisc noqueue state UP group default 
    link/ether 06:3e:42:00:91:33 brd ff:ff:ff:ff:ff:ff link-netnsid 0
    inet 172.27.0.74/24 brd 172.27.0.255 scope global eth0
       valid_lft forever preferred_lft forever
```

&#x20;注意看 `if` 后面的数字，宿主机上查看下是哪个 `veth`

```
$ ip link | grep -E '^210'
210: vetha1ca1d55@if3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1450 qdisc noqueue master cni0 state UP mode DEFAULT group default
```

使用 `brctl` 看下 `cni0` 下是有这个的:

```
$ brctl show cni0 | grep vetha1ca1d55
							vetha1ca1d55
```

