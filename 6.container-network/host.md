# host

因为桥接多了层转发，如果对网络想最优可以在起容器的时候加上`--net host`，这样容器将不会获得一个独立的 Network Namespace，而是和宿主机共用一个 Network Namespace。容器将不会虚拟出自己的网卡，配置自己的 IP 等，而是使用宿主机的 IP 和端口。但是，容器的其他方面，如文件系统、进程列表等还是和宿主机隔离的。

![](../.gitbook/assets/image%20%2832%29.png)

但是要注意这样一个主机无法跑两个端口一样的容器，另外如果一些自己开发的应用或者移植的应用在非容器跑得好好的，然后容器里网络报错或者慢的可以考虑使用 host 模式启动看看问题消失没。

host 网络的容器只有每次启动的时候才会去同步宿主机的 hosts 文件，另外如果 host 网络下 `--add-host` 的话，容器里的 hosts 不会和宿主机一样。

host 网络下，java 相关的要注意，java很多项目启动会解析 hostname，无法解析会无法启动，例如zookeeper，之前也遇到过 springboot的一个项目在 `/etc/hosts` 里没有 hostname 的解析记录启动要15分钟，加了hostname 的解析后3分钟就启动了。

