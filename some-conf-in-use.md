# 8.一些使用和配置问题

## docker pull x509:certificate signed by unknown authority <a id="articleContentId"></a>

```text
domain=xxx
port=443

mkdir -p /etc/docker/certs.d/${domain}
openssl s_client -showcerts -connect ${domain}:${port} < /dev/null | \
  sed -ne '/-BEGIN CERTIFICATE-/,/-END CERTIFICATE-/p' > /etc/docker/certs.d/${domain}/ca.crt
  
```

### 根据端口查容器名字

```text
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

### 判断是否在容器内

`/proc/1/sched`的第一行会显示真实的 pid，所以可以靠这个来判断是否是在容器内

```text
if [ `head -n1 /proc/1/sched | cut -d \( -f2 | cut -d, -f1` -eq 1 ];then
    echo not in container
else
    echo in container
fi
```

### 一次复制多个文件到容器里

```text
tar -c $(ls | grep -v "^(ui\|ui-v2\|website\|bin\|pkg\|.git)") \
    | docker cp - ${container_id}:/consul
```

