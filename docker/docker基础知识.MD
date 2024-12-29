
1. docker配置代理https://zhuanlan.zhihu.com/p/703032938
    - /etc/docker/daemon.json
    ```
    {
        "proxies": {        
            "http-proxy": "http://127.0.0.1:7890",
            "https-proxy": "http://127.0.0.1:7890",
            "no-proxy": "localhost,127.0.0.0/8"
        }
    }
    ```
    - vim ~/.docker/config.json
    ```
    {
        "proxies": {
            "default": {
                "httpProxy": "http://127.0.0.1:7890",
                "httpsProxy": "http://127.0.0.1:7890",
                "noProxy": "127.0.0.0/8"
            }
        }
    }
    ```
    - systemctl restart docker 重启docker
2. Dockerfile用于如何构建镜像
    - docker build -t test:latest .  使用Dockerfile打包镜像 镜像名称为test，tag为latest .表示当前目录。
    - 以下是Dockerfile文件结构。
    ```
    # syntax=docker/dockerfile:1.8.1
    ```
    这是Dockerfile语法镜像，将版本固定为1.8.1就不会每次都下载新的。也可以填1就表示会下载大版本1下的最新小版本镜像。
    ```
    FROM mirekphd/python3.11-ubuntu22.04:20230107
    ```
    表示把 mirekphd/python3.11-ubuntu22.04:20230107这个镜像作为基础镜像。
    ```
    WORKDIR /web
    ```
    表示容器内部工作目录为web
    ```
    RUN apt update && apt install -y libpython3.11-dev && apt install -y gcc && apt install -y libsqlite3-dev
    ```
    RUN 表示在容器内运行shell命令
    ```
    COPY requirements.txt /web/
    RUN pip3 install -r requirements.txt --no-cache-dir
    COPY . /web/
    ```
    COPY 表示拷贝哪些文件到容器里面 /web是容器内部的路径。
    ```
    EXPOSE 8080
    ```
    EXPOSE 表示容器运行时会将8080端口暴露出来作为外部接口。
    ```
    ENV APP_NAME=docker_test
    ```
    添加环境变量，这个是容器运行时全局的环境变量。
    ```
    CMD ["bash","run.sh"]
    ```
    CMD 表示使用bash run.sh应用程序。

3. .dockerignore COPY拷贝文件时排除指定文件，有多个Dockerfile时，可以使用Dockerfile的名字+.dockerignore 作为整个dockerfile的忽略文件
```
**/__pycache__
docker-compose.yml
Dockerfile
```
表示.dockerignore所在目录的Dockerfile文件和docker-compose.yml文件不会拷贝到web目录下。并且所有目录下的__pycache__文件都不会拷贝到web目录下
4. 容器指令
```
docker container run -it mongo /bin/bash -p 80:2222
```
mongo镜像启动是运行mongodb数据库进程，可以指定要bash来运行镜像的命令行程序
-p指定端口映射格式：主机(宿主)端口:容器端口
```
docker container exec -it 446564353a05 bash
```
指定容器id可以连接到正在运行的容器的终端当中
```
docker container start 容器id
docker container stop  容器id
```
运行和关闭指定容器

5. 查看容器log
```
docker logs 容器id
```
查看容器log，主机上docker容器日志路径是/var/lib/docker/containers/<容器ID>/<容器ID>-json.log

6. 多阶段构建

Dockerfile构建可以有多个阶段，使用FROM拷贝上一个阶段的内容，使用AS为这个阶段命名
```
FROM alpine:latest AS builder
RUN apk --no-cache add build-base

FROM builder AS build1
COPY source1.cpp source.cpp
RUN g++ -o /binary source.cpp

FROM builder AS build2
COPY source2.cpp source.cpp
RUN g++ -o /binary source.cpp
```
可以单独编译一个阶段，通常用来调试
```
docker build --target builder -t hello .
```
7. dockers compose
使用docker-compose.yml文件定义和运行多容器应用程序
以下是compose 例子
```
services:
  mongo:
    image: mongo:7.0.9
    restart: always
    environment:
      MONGO_INITDB_ROOT_USERNAME: admin
      MONGO_INITDB_ROOT_PASSWORD: password
    healthcheck:
       test: echo 'db.runCommand("ping").ok' | mongosh localhost:27017/test --quiet
       interval: 10s
       timeout: 10s
       retries: 5
    volumes:
      - type: bind
        source: ./mongodb/mongo_conf/mongod.conf
        target: /etc/mongod.conf
      - type: volume
        source: mongo_data
        target: /data/db
        volume:
          subpath: db
      - type: volume
        source: mongo_data
        target: /var/log/mongodb/mongodb.log
        volume:
          subpath: mongodb.log
    networks:
        - django-net
  web:
    build: ./
    ports:
      - "0.0.0.0:80:8080"
    networks:
      - django-net
    volumes:
      - type: volume
        source: django-vol
        target: /data
      - type: bind
        source: ~/django_db
        target: /db
    depends_on
      mongo:
        condition: service_healthy
        restart: true

networks:
  django-net: 
    driver: bridge
volumes:
  django-vol: {}
  mongo_data: {}
```
services元素下用于定义有哪些应用，每个应用可以使用image指定镜像或者使用build 指定需要编译的镜像，depends_on字段定义的是多个应用程序之间的依赖，比如上面的配置是web应用需要mongo应用程序启动并且能正常连接才能启动web引用，什么才算正常连接由mongo应用的healthcheck定义，其中的test是运行的命令，这条命令返回0才算正常连接interval，timeout，retries分别定义了每次运行命令之间的间隔，单条命令运行的超时时间，重试的次数。
8. docker 卷

9. docker 网络
以下是创建网络和在指定网络上运行容器的命令
```
docker network create -d bridge my-net
docker run --network=my-net -itd --name=container3 busybox
```
容器网络有以下类型
* bridge：本机桥接网络，主机外部不能访问，连接到一个bridge网络的容器可以互相通信。在不指定网络时容器会连接到一个默认的桥接网络，但通常只能使用ip互相访问服务器。运行在用户自定义的网络上的容器可以使用容器名来访问。
* host： 创建的容器与主机共享网络命名空间，容器可以访问主机的网络接口，无需nat速度很快，但容易导致端口冲突
* overlay： 覆盖网络可以将多个docker守护进程连接在一起，使swarm服务下的容器可以跨主机通信
* macvlan：Macvlan 网络允许您为容器分配 MAC 地址，使其在您的网络上显示为物理设备
参考https://icloudnative.io/posts/netwnetwork-virtualization-macvlan/
10. 参考资料
更多是参考官方文档https://docs.docker.com/
比如https://docs.docker.com/reference 这里有Dockerfile，dockers compose file文件的语法规则

