---
description: 用户可以docker build时使用--build-arg <varname>=<value> 将该变量传递给构建器替换默认变量
---

# ARG

格式有两种

```text
ARG key
----------------
ARK key=value
ARG key=value \
    key2=value2 \
    key3=value3 
```

一般来讲第二种用得多，表明build的时候不传入变量设置默认值，无值也就是第一种下用户在docker build的时候必须传入值，否则就报错。例如我们可以把nginx官方dockerfile的第一个ENV改成ARG，我们想构建哪个版本直接build的时候传入变量就行了

当然ARG是唯一一个可以用于FROM前面的指令，例如下面这样我们可以通过命令行传递参数来改变FROM的base镜像

```text
ARG jdk=1.8xxxx
FROM openjdk:$jdk
```

Docker其实也预定了一些ARG方便我们构建的时候使用代理

* HTTP\_PROXY 
* http\_proxy 
* HTTPS\_PROXY 
* https\_proxy 
* FTP\_PROXY 
* ftp\_proxy 
* NO\_PROXY 
* no\_proxy

