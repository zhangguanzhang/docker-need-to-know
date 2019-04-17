---
description: 在这一层执行命令
---

# RUN

有两种形式

* RUN command \( 该命令在shell中运行，默认情况下在Linux上是`/bin/sh -c`或windows的`cmd /S /C`\)
* RUN \["executable", "param1", "param2"\] \(exec 形式\)

exec形式不会调用shell先展开变量，也就是不会解析ENV或者ARG的变量，所以一般来讲用得比较多的就是第一种形式，多行的话可以利用`\`换行

```text
RUN  .....\
    && addgroup -S nginx \
	&& adduser -D -S -h /var/cache/nginx -s /sbin/nologin -G nginx nginx \
	&& apk add --no-cache --virtual .build-deps \
	.....
```

这里要注意的是一个RUN是一层，dockerfile的一些涉及到文件的指令和RUN都会是新的一层，主要是构建过程实际上还是容器去commit，**目的相同的RUN尽量合并在同一个RUN里减少大小**。下面我做个例子来说明原因

```text
FROM alpine
RUN apk add wget  && wget https://www.baidu.com -O test.html
RUN echo 123 > test.html
```

构建并运行

```text
$ docker build -t test .
Sending build context to Docker daemon  57.34kB
Step 1/3 : FROM alpine
 ---> 5cb3aa00f899
Step 2/3 : RUN apk add wget  && wget https://www.baidu.com -O test.html
 ---> Running in 5b3657e06a8e
fetch http://dl-cdn.alpinelinux.org/alpine/v3.9/main/x86_64/APKINDEX.tar.gz
fetch http://dl-cdn.alpinelinux.org/alpine/v3.9/community/x86_64/APKINDEX.tar.gz
(1/1) Installing wget (1.20.3-r0)
Executing busybox-1.29.3-r10.trigger
OK: 6 MiB in 15 packages
--2019-04-16 17:54:33--  https://www.baidu.com/
Resolving www.baidu.com... 61.135.169.121, 61.135.169.125
Connecting to www.baidu.com|61.135.169.121|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 2443 (2.4K) [text/html]
Saving to: 'test.html'

     0K ..                                                    100%  252M=0s

2019-04-16 17:54:33 (252 MB/s) - 'test.html' saved [2443/2443]

Removing intermediate container 5b3657e06a8e
 ---> b7fc2afcf946
Step 3/3 : RUN echo 123 > test.html
 ---> Running in 10c384962a0c
Removing intermediate container 10c384962a0c
 ---> 6382e6e749fa
Successfully built 6382e6e749fa
Successfully tagged test:latest
```

运行然后查看docker的存储目录查找

```text
[root@CentOS76 ~]# docker run --rm test cat test.html
123
[root@CentOS76 ~]# find /var/lib/docker/overlay2/ -type f -name test.html
/var/lib/docker/overlay2/730523f21eca6a6719d0d78bac606b1d071df3af7048a82d87a4381f271c454c/diff/test.html
/var/lib/docker/overlay2/7d20662a7afb25471760a6619d5b1cf94348e02d8cdef325f88f163fc28958e8/diff/test.html
^C
[root@CentOS76 ~]# cat /var/lib/docker/overlay2/7d20662a7afb25471760a6619d5b1cf94348e02d8cdef325f88f163fc28958e8/diff/test.html
123
[root@CentOS76 ~]# cat /var/lib/docker/overlay2/730523f21eca6a6719d0d78bac606b1d071df3af7048a82d87a4381f271c454c/diff/test.html
<!DOCTYPE html>
<!--STATUS OK--><html> <head><meta http-equiv=content-type content=text/html;charset=utf-8><meta http-equiv=X-UA-Compatible content=IE=Edge><meta content=always name=referrer><link rel=stylesheet type=text/css href=https://ss1.bdstatic.com/5eN1bjq8AAUYm2zgoY3K/r/www/cache/bdorz/baidu.min.css><title>百度一下，你就知道</title></head> <body link=#0000cc> <div id=wrapper> <div id=head> <div class=head_wrapper> <div class=s_form> <div class=s_form_wrapper> <div id=lg> <img hidefocus=true src=//www.baidu.com/img/bd_logo1.png width=270 height=129> </div> <form id=form name=f action=//www.baidu.com/s class=fm> <input type=hidden name=bdorz_come value=1> <input type=hidden name=ie value=utf-8> <input type=hidden name=f value=8> <input type=hidden name=rsv_bp value=1> <input type=hidden name=rsv_idx value=1> <input type=hidden name=tn value=baidu><span class="bg s_ipt_wr"><input id=kw name=wd class=s_ipt value maxlength=255 autocomplete=off autofocus=autofocus></span><span class="bg s_btn_wr"><input type=submit id=su value=百度一下 class="bg s_btn" autofocus></span> </form> </div> </div> <div id=u1> <a href=http://news.baidu.com name=tj_trnews class=mnav>新闻</a> <a href=https://www.hao123.com name=tj_trhao123 class=mnav>hao123</a> <a href=http://map.baidu.com name=tj_trmap class=mnav>地图</a> <a href=http://v.baidu.com name=tj_trvideo class=mnav>视频</a> <a href=http://tieba.baidu.com name=tj_trtieba class=mnav>贴吧</a> <noscript> <a href=http://www.baidu.com/bdorz/login.gif?login&amp;tpl=mn&amp;u=http%3A%2F%2Fwww.baidu.com%2f%3fbdorz_come%3d1 name=tj_login class=lb>登录</a> </noscript> <script>document.write('<a href="http://www.baidu.com/bdorz/login.gif?login&tpl=mn&u='+ encodeURIComponent(window.location.href+ (window.location.search === "" ? "?" : "&")+ "bdorz_come=1")+ '" name="tj_login" class="lb">登录</a>');
                </script> <a href=//www.baidu.com/more/ name=tj_briicon class=bri style="display: block;">更多产品</a> </div> </div> </div> <div id=ftCon> <div id=ftConw> <p id=lh> <a href=http://home.baidu.com>关于百度</a> <a href=http://ir.baidu.com>About Baidu</a> </p> <p id=cp>&copy;2017&nbsp;Baidu&nbsp;<a href=http://www.baidu.com/duty/>使用百度前必读</a>&nbsp; <a href=http://jianyi.baidu.com/ class=cp-feedback>意见反馈</a>&nbsp;京ICP证030173号&nbsp; <img src=//www.baidu.com/img/gs.gif> </p> </div> </div> </div> </body> </html>
[root@CentOS76 ~]# 
```

我们发现两个文件都存在，前面说到了容器在读取文件的时候从上层往下查找，查找到了就返回，但是我的这个Dockerfile里第一个RUN下载了index页面，第二个改了文件内容。

可以证明一个RUN是一层，也证明了之前容器读取文件的逻辑。同时假设我们的目的是最终的123，我们可以俩个RUN合并了，这样就不会有多余的第一个RUN产生的test.html文件

