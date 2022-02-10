# 特权构建镜像

docker run 或者 docker build的时候用`--security-opt seccomp:unconfined`参数来允许容器执行全部的系统的调用。有条件的话也可以去用 `Kaniko`

