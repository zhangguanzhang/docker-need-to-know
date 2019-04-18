# STOPSIGNAL--与信号转发

格式，缺省信号为`SIGTERM`

```text
STOPSIGNAL signal
------
STOPSIGNAL SIGTERM
STOPSIGNAL 9
```

可以是kill -l的信号名字也可以信号数字:

```text
kill -l
 1) SIGHUP	 2) SIGINT	 3) SIGQUIT	 4) SIGILL	 5) SIGTRAP
 6) SIGABRT	 7) SIGBUS	 8) SIGFPE	 9) SIGKILL	10) SIGUSR1
11) SIGSEGV	12) SIGUSR2	13) SIGPIPE	14) SIGALRM	15) SIGTERM
16) SIGSTKFLT	17) SIGCHLD	18) SIGCONT	19) SIGSTOP	20) SIGTSTP
21) SIGTTIN	22) SIGTTOU	23) SIGURG	24) SIGXCPU	25) SIGXFSZ
26) SIGVTALRM	27) SIGPROF	28) SIGWINCH	29) SIGIO	30) SIGPWR
31) SIGSYS	34) SIGRTMIN	35) SIGRTMIN+1	36) SIGRTMIN+2	37) SIGRTMIN+3
38) SIGRTMIN+4	39) SIGRTMIN+5	40) SIGRTMIN+6	41) SIGRTMIN+7	42) SIGRTMIN+8
43) SIGRTMIN+9	44) SIGRTMIN+10	45) SIGRTMIN+11	46) SIGRTMIN+12	47) SIGRTMIN+13
48) SIGRTMIN+14	49) SIGRTMIN+15	50) SIGRTMAX-14	51) SIGRTMAX-13	52) SIGRTMAX-12
53) SIGRTMAX-11	54) SIGRTMAX-10	55) SIGRTMAX-9	56) SIGRTMAX-8	57) SIGRTMAX-7
58) SIGRTMAX-6	59) SIGRTMAX-5	60) SIGRTMAX-4	61) SIGRTMAX-3	62) SIGRTMAX-2
63) SIGRTMAX-1	64) SIGRTMAX
```

docker run的选项可以覆盖镜像定义的STOPSIGNAL信号

```text
--stop-signal string           Signal to stop a container (default "SIGTERM")
```

在docker stop停止运行容器的时候指定发送给容器里pid为1角色的信号。默认超时10秒，超时则发送kill强杀进程。一般业务进程都是pid为1，所有官方的进程都会处理收到的SIGTERM信号进行优雅收尾退出。

前面说过了如果CMD是/bin/sh格式的话，主进程是一个sh -c的进程，shell不用trap处理的话是无法转发信号的。下面我举个例子

例子是是网上找的，两种CMD方式启动的redis

```text
FROM ubuntu:14.04
RUN apt-get update && apt-get -y install redis-server && rm -rf /var/lib/apt/lists/*
EXPOSE 6379
CMD /usr/bin/redis-server
----------------------------
FROM ubuntu:14.04
RUN apt-get update && apt-get -y install redis-server && rm -rf /var/lib/apt/lists/*
EXPOSE 6379
CMD ["/usr/bin/redis-server"]
```

构建两种镜像，然后docker run -d img\_name，然后docker stop这俩镜像启动的容器会发现exec的redis能在docker stop的时候收到信号优雅退出`Received SIGTERM, scheduling shutdown`

```text
...
[1] 11 Feb 08:13:01.633 * The server is now ready to accept connections on port 6379
[1 | signal handler] (1455179074) Received SIGTERM, scheduling shutdown...
[1] 11 Feb 08:24:34.259 # User requested shutdown...
[1] 11 Feb 08:24:34.259 * Saving the final RDB snapshot before exiting.
[1] 11 Feb 08:24:34.262 * DB saved on disk
[1] 11 Feb 08:24:34.262 # Redis is now ready to exit, bye bye...
```

而/bin/sh的形式的redis在docker stop后去docker logs看日志会发现根本没有优雅退出，类似于强制杀掉一样。

```text
...
[5] 11 Feb 08:12:40.109 * The server is now ready to accept connections on port 6379
```

这是因为/bin/sh形式启动的redis主进程是一个sh，shell不会转发信号，所以最后sh被超时的docker stop发送了kill信号杀掉，整个容器生存周期结束，redis没有触发`signal handler` 

`参考:`

* [http://www.cnblogs.com/ilinuxer/p/6188303.html](http://www.cnblogs.com/ilinuxer/p/6188303.html)

