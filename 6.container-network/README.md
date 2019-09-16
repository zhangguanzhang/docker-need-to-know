# 6.容器网络

可以在docker run --net=xxx  来指定容器使用的网络类型

![](../.gitbook/assets/image%20%2838%29.png)

* 默认的bridge
* 不隔离network namespaces的host模式，容器网络和宿主机一样
* none docker不会去设置容器的网络信息，用户可以自行去设置，或者说容器根本不需要ip，例如一个专门跑编译工作的容器
* container   容器启动的时候选择和已存在的容器共用同一个network namespaces
* --link 过时的容器互联方式，不要使用 



