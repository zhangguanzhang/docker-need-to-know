# init层与过时的--link

好像是到1.9版本之前docker的容器网络都是他们自己做的，后面收购了一个做网络的团队还是公司来着，容器的网络改变成现在这样了，--link好像就是遗留的产物，不少教程和一些dockerhub上官方demo里居然还有使用它的。先说下如何使用它

![](../.gitbook/assets/image%20%2847%29.png)

实际上还有层init层

![](../.gitbook/assets/image%20%2839%29.png)

它是一个以“-init”结尾的层，夹在只读层和读写层之间。Init 层是 Docker 项目单独生成的一个内部层，专门用来存放 /etc/hosts、/etc/resolv.conf 等信息。

hostname和dns设置都是针对当前容器生效，所以这层单独挂载，docker commit的时候只会提交容器那层读写层。我们启动的时候也可以设置init层的hostname和dns，可以docker run --help看看

--link就是用init层设置注入的hosts实现容器用名字互通，实际上这种非常low，容器重启了之类的ip会变是很正常的，hosts可不会改变

