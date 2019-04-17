# ONBUILD

用法

```text
ONBUILD [INSTRUCTION]
```

构建的时候并不会执行，只有在构建出来的镜像被FROM的时候才执行，例如

```text
FROM xxxx
ONBUILD RUN cd /root/ && wget xxxx 
```

然后构建出镜像B里root目录并没有下载东西，只有FROM B构建的镜像才会执行这个RUN，这个用得很少，记住即可

