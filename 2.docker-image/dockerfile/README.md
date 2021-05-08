# Dockerfile

Dockerfile里的指令描述了构建镜像的步骤，常见的指令为下面，**指令不一定需要大写，但是尽量大写**

* FROM 
* MAINTAINER （已弃用）
* ENV 
* ARG 
* RUN 
* COPY 
* ADD
* USER 
* WORKDIR 
* ONBUILD 
* EXPOSE 
* CMD 
* ENTRYPOINT 
* VOLUME 
* STOPSIGNAL 
* SHELL 
* HEALTHCHECK

哪些指令只能最后一个生效我会说明的。基本都是可以多个使用，但是有些指令实际上只是一个属性的记录，所以最终那个才生效覆盖住前面的。很多指令有两种写法，那种大括号写法的也可以称作 json 写法，引号必须是双引号

