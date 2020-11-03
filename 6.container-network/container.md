# container

一张实践图说明

![](../.gitbook/assets/image%20%2849%29.png)

网络情况如下图

![](../.gitbook/assets/image%20%2867%29.png)

这种场景纯docker的话可以用来排错，例如某个容器网络不正常，内网情况下这个容器里又没网络排查工具。

我们可以自己做个带一些网络工具的镜像，启动容器的时候使用这个模式然后进容器里抓包分析啥的。k8s里的pod就是一组容器共用一个network namespace，容器之间使用localhost就能通信. k8s起的pod的容器我们也可以使用container附加同一个网络起一个容器排查网络情况。

nsenter也可以附加进入容器的ns，直接使用宿主机上的命令行进入附加进入到容器的ns里调试，但是容器的`/etc/resolv.conf`属于mount ns的，单纯附加`--net`

```text
$ kubectl get po
NAME                     READY   STATUS    RESTARTS   AGE
nginx-668cd5d7b5-f6bxf   1/1     Running   7          43d
[root@k8s-m1 ~]# docker ps -a | grep nginx-668
a7abc0e4af98        nginx                                            "/docker-entrypoint.…"   3 weeks ago         Up 3 weeks                                         k8s_nginx_nginx-668cd5d7b5-f6bxf_default_d37bb01b-fb50-11ea-8a97-5254009b2a38_7
c56b4931d003        gcr.azk8s.cn/google_containers/pause-amd64:3.1   "/pause"                 3 weeks ago         Up 3 weeks                                         k8s_POD_nginx-668cd5d7b5-f6bxf_default_d37bb01b-fb50-11ea-8a97-5254009b2a38_25
$ docker inspect a7abc0e4af98 | grep -m1 -i pid
            "Pid": 3376,
$ cat /etc/resolv.conf
nameserver 10.236.158.114
nameserver 10.236.158.106
$ nsenter --net -t 3376
$ netstat -nlptu
Active Internet connections (only servers)
Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name    
tcp        0      0 0.0.0.0:80              0.0.0.0:*               LISTEN      3376/nginx: master  
tcp6       0      0 :::80                   :::*                    LISTEN      3376/nginx: master  
$ cat /etc/resolv.conf
nameserver 10.236.158.114
nameserver 10.236.158.106
# 这个说明--net只是附加网络，/etc/resolv.conf并不能附加
$ docker exec a7abc0e4af98 cat /etc/resolv.conf
nameserver 10.96.0.10
search default.svc.cluster.local svc.cluster.local cluster.local
options ndots:5
```

nsenter带上 `--mount` 才能看到容器的`/etc/resolv.conf`，但是这样进去就是容器的rootfs了，没有排查命令，所以更多时候还是准备一个工具镜像使用`docker run --net container`去排查最保险

```text
$ docker run --rm --net container:a7abc0e4af98 alpine:latest cat /etc/resolv.conf
nameserver 10.96.0.10
search default.svc.cluster.local svc.cluster.local cluster.local
options ndots:5
```

