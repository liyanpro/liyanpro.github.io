---
title: docker命令
date: 2021-02-01 10:24:32
tags:
---
### 1.通过DockFile构建镜像

DockerFile文件

    FROM openjdk:8
    MAINTAINER liyan <liy616@chinaunicom.cn>
    COPY monitor-1.0.0.jar /opt
    WORKDIR /opt
    CMD java -jar  -Xmx2G -Xms1G  /opt/monitor-1.0.0.jar

 dockerfile文件详解
  
### 2. 本地构建镜像
进入dockerfile层级目录

    docker build -t imageName:tag .
    docker build  -f Dockerfile.hub -t  ant-design-pro ./

###3. 本地启动镜像

   - -d: 后台运行容器，并返回容器ID；
   - -p: 指定端口映射，格式为：主机(宿主)端口:容器端口
   - --name="nginx-lb": 为容器指定一个名称；
   
    docker run -d -p  主机端口:容器端口  --name NewContainerName  imageName:tag

   在容器中开启交互终端
   
    docker exec -i -t container_id /bin/bash

###4  保存镜像到本地

    docker save -o filename.tar imageName:tag


###5. 上传到远程仓库
   #### 1）使用前在本地Docker服务中配置Docker Registry，重启服务
    {
    "experimental": false,
    "debug": true,
    "registry-mirrors": [
      "https://eodtyohs.mirror.aliyuncs.com",
       "https://reg-mirror.qiniu.com"
     ],
     "insecure-registries": ["ip"]
    }

   ####2）登录镜像仓库
   
    docker login --username=liyan  ip

   ####3) 推送镜像
   
    docker tag ImageId ip/path/ImagesName:ImageTag
    docker push ip/path/ImagesName:ImageTag

   ####4) 拉取镜像
   
    docker pull ip/path/ImagesName:ImageTag

