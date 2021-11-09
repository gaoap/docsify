# MySQL 安装

第一步：

```shell
docker run -p 3306:3306  --name MySQL8 --restart=always \
-e MYSQL_ROOT_PASSWORD=123456 \
-v /opt/mysql/data:/var/lib/mysql \
-v /opt/mysql/conf:/etc/mysql \
-v /opt/mysql/logs:/var/log/mysql \
-v /opt/mysql/mysql-files:/var/lib/mysql-files \
-d mysql


```

第二步：

```shell
docker exec -it MySQL8   bash
```

第三步：设置远程连接权限

```shell
mysql -uroot -p123456
use mysql;

select host, user, authentication_string, plugin from user;

GRANT ALL ON *.* TO 'root'@'%';
```

