# ENTRYPOINT

和CMD用法也一样两种格式，唯一要注意的就是区别

CMD和ENTRYPOINT只有一个或者两者都有都可以，容器最终运行的命令为

**&lt;ENTRYPOINT&gt; &lt;CMD&gt;**

例如我们做个测试

![](../../.gitbook/assets/image%20%285%29.png)

可以很形象的证明了是这个关系，最终运行的是**&lt;ENTRYPOINT&gt; &lt;CMD&gt;**，同时不光在docker run的时候覆盖掉CMD，也可以覆盖掉默认的entrypoint。很多时候我们可以主进程bash或者sh进去手动启动看看。老版本接触不多，不确定老版本有没有--entrypoint的选项

