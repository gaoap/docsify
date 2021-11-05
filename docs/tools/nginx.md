# nginx安装

第一步：

```shell
docker run  --name myNginx  -d -p 80:80  nginx
```

第二步：

```shell
mkdir -p /home/mynginx/nginx/conf/ 
docker cp myNginx:/etc/nginx/nginx.conf /home/mynginx/nginx/conf/ 
docker cp myNginx:/usr/share/nginx/html /home/mynginx/nginx/ 
docker cp myNginx:/etc/nginx/conf.d /home/mynginx/nginx/ 
docker cp myNginx:/var/log/nginx /home/mynginx/nginx/ 
```

第三步：

删除第一步创建的ngnix

```
docker rm -f  myNginx
```



第四步：

```shell
docker run \
  --name myNginx \
  -d -p 80:80 \
  -d -p 443:443 \
  -v /home/mynginx/nginx/html:/usr/share/nginx/html:ro  \
  -v /home/mynginx/nginx/conf/nginx.conf:/etc/nginx/nginx.conf:ro \
  -v /home/mynginx/nginx/conf.d:/etc/nginx/conf.d \
  -v /home/mynginx/nginx/logs:/var/log/nginx \
 --restart=always  nginx

```

可以使用如下命令，进入容器检查：

```shell
docker exec -it myNginx  /bin/bash
```

