# 2.docker镜像

介绍下docker镜像和构建docker镜像的相关知识以及排错

先大家记住下面几个命令，因为本书写知识点顺序可能难以让人理解

* docker run 参数 镜像名  命令 :  从一个镜像创建出一个容器运行命令
* docker ps : 列出当前的容器基本信息
* docker exec 容器名/容器id前几位  命令:  在容器里运行命令
* docker inspect 对象 :  查看对象的一些信息，对象可以为docker的所有对象
* docker history --no-trunc 镜像名 :  查看镜像的构建信息，构建步骤是实际构建步骤

