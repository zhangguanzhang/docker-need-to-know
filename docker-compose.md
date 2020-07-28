# 7.编排利器docker-compose

其实到现在依然有人在有了一套docker服务后去写个启动脚本或者docker run一大堆参数，这样非常不方便。早期有人针对启动一套服务二开发了一个项目，好像仅仅只是两个开发者开发了一个叫fig的项目，用户用yaml声明服务和容器参数最后用api去调用docker daemon创建容器。最终Fig已经被Docker公司收购并成为官方支持的解决方案。

实际上我们在dockerhub上寻找镜像的时候，跳转到对应的github上的readme多多少少会有docker-compose的yaml示例文件

例如我

```yaml
version: "2"
services:
  web:
    image: nginx
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - /etc/nginx/nginx.conf:/etc/nginx/nginx.conf
      - /etc/nginx/ssl:/etc/nginx/ssl:ro
      - /data:/data
    networks:
      - code-network
  mysql:
    image: mysql:5.7
    volumes:
      - /data/mysql:/var/lib/mysql
    ports:
      - "3306:3306"
    environment:
      MYSQL_DATABASE: test
      MYSQL_USER: root
      MYSQL_PASSWORD: 123456789
      MYSQL_ROOT_PASSWORD: 123456789
    networks:
      - code-network
  php:
    image: php:7.2-fpm
    volumes:
      - /data:/data
    ports:
      - "9000:9000"
    environment:
      TZ: Asia/Shanghai
    networks:
      - code-network

networks:
  code-network:
    driver: bridge
```

更多的可以看别人写的，这里仅仅是介绍

[https://github.com/buxiaomo/docker-compose](https://github.com/buxiaomo/docker-compose)

