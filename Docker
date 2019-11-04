docker run ubuntu:15.10 /bin/echo “docker display”
docker run -it ubuntu:15.10  /bin/bash
docker run –itd --name ubuntu-learn ubuntu:15.10 /bin/bash 

exit 或者 ctrl + d

-- 后台模式 –d
docker run –d ubuntu:15.10 /bin/sh –c “while true; do echo hello world;sleep 1;done”
docker ps [-a]

-- 查看容器的标准输出
docker logs [container_id]
docker [stop|restart][container_id]

-- attch命令退出会导致容器退出，推荐使用exec
docker exec –it [container_id] /bin/bash
-- docker exec -- help

-- 导出容器
docker export [container_id] > ubuntu_15.10.tar
-- 导入镜像
cat ubuntu_15.10.tar | docker import – test/ubuntu:v15.10

docker rm –f [container_id]
-- 清除所有终止状态的容器
docker container prune

docker pull training/webapp
-- P:将容器内部使用的网络端口映射到我们使用的主机上。
docker run –d –P 5000:5000 training/webapp python app.py

docer port [container_id]或[container_name]
docker top [container_id]
docker inspect [container_id]

docker search [image_name]
docker rmi [image_name] – 删除镜像

-- 根据运行的容器来创建镜像
docker commit –m “update config” –a = “eason” [container_id] eason/ubuntu:v2.1

根据Dockerfile创建镜像
` docker build –t eason/centos:7 .`
FROM centos:7
MAINTAINER “southwang”
RUN /bin/echo “root:123”|chpasswd
RUN useradd eason
RUN /bin/echo “eason:123”|chpassword
RUN /bin/echo –e “LANG=\"en_US.UTF-8\"" > /etc/default/local
EXPOSE 22
EXPOSE 88
CMD / usr/sbin/sshd -D

为镜像添加标签
docker tag [image_id] [image_name]
为容器建立网络
docker network create –d bridge cd-1309-net
docker run –itd –name basic_server_1 - -network  ubuntu /bin/bash
docker run –itd –name basic_server_2 - -network  ubuntu /bin/bash

配置DNS
`etc/docker/daemon.json`

```shell
{
  "dns" : [
    "114.114.114.114",
    "8.8.8.8"
  ]
}
```


或者
docker run –it - -rm host_ubuntu - -dns=114.114.114.144 -- dns-search=test.com ubuntu

--查看容器的DNS信息
docker run –it - - rm ubuntu cat etc/resolv.conf

仓库相关
docker log
docker logout
docker search ubuntu
docker pull ubuntu
docker tag ubuntu:18.12 eason/ubunutu:12.12
docer image ls
