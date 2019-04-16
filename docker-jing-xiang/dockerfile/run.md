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

