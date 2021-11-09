docker常用命令

```shell
删除build过程中产生的镜像 : docker image prune -a -f
删除所有镜像：docker rmi -f $(docker images -q)
删除所有容器：docker rm -f $(sudo docker ps -a -q)

docker-compose 启动： docker-compose -f opf-docker-compose.yaml up
```

```shell
查看防火墙：
systemctl status firewalld.service
关闭防火墙：
systemctl stop firewalld service
systemctl disable firewalld.service，开机禁止防火墙服务器
systemctl enable firewalld.service，开机启动防火墙服务器


redis :
docker pull redis:latest
docker run -itd --name redis-test -p 6379:6379 --restart=always redis

docker exec -it redis-test /bin/bash
```

