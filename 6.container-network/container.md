# container

一张实践图说明

![](../.gitbook/assets/image%20%2849%29.png)

这种场景纯docker的话可以用来排错，例如某个容器网络不正常，内网情况下这个容器里又没网络排查工具，我们可以自己做个带一些网络工具的镜像，启动容器的时候使用这个模式然后进容器里抓包分析啥的。k8s里的pod就是一组容器共用一个network namespace，容器之间使用localhost就能通信. k8s起的pod的容器我们也可以使用container附加同一个网络起一个容器排查网络情况

![](../.gitbook/assets/image%20%2867%29.png)

