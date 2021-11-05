# vue部署

1、打包命令

```shell
 npm run build
```

2、将 vue 打包好的 dist 文件丢到 nginx 的html 目录下

3、配置nginx

cd /etc/nginx/conf.d

vim upm.gaoap.cn.conf 添加如下内容 。注意：#upm.gaoap.cn.conf 这个名字自己随便写

```xml
server {
    listen       80;
    listen  [::]:80;
    server_name  upm.gaoap.cn; #host中配置了域名  192.168.31.82 upm.gaoap.cn 

    #access_log  /var/log/nginx/host.access.log  main;

    location / {
        root   /usr/share/nginx/html/dist;
        index  index.html index.htm;
        try_files $uri $uri/ @router; # 需要指向下面的@router否则会出现vue的路由在nginx中刷新出现404
    }
    #对应上面的@router，主要原因是路由的路径资源并不是一个真实的路径，所以无法找到具体的文件
    location @router {
    #因此需要rewrite到index.html中，然后交给路由再处理请求资源
    		rewrite ^.*$ /index.html last;
    }

    # 请求后台服务器，代理设置。所有路径为/upm的请求，转发到http://192.168.31.82:9021
    #
    location ~ /upm {
        proxy_pass   http://192.168.31.82:9021;  #这个是前后端分离的后端服务，自行部署
    }

    # pass the PHP scripts to FastCGI server listening on 127.0.0.1:9000
    #
    #location ~ \.php$ {
    #    root           html;
    #    fastcgi_pass   127.0.0.1:9000;
    #    fastcgi_index  index.php;
    #    fastcgi_param  SCRIPT_FILENAME  /scripts$fastcgi_script_name;
    #    include        fastcgi_params;
    #}

    # deny access to .htaccess files, if Apache's document root
    # concurs with nginx's one
    #
    #location ~ /\.ht {
    #    deny  all;
    #}
}

```

浏览器访问：http://upm.gaoap.cn/ 即可