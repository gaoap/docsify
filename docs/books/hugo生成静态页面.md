# hugo生成静态页面 <!-- {docsify-ignore-all} -->

1、执行hugo命令，站点目录下会新建文件夹public/，生成的所有静态网站页面都会存储到这个目录，使用nginx作为web服务配置root dir 指向public/ 即可；

2、修改nginx配置，添加监听域名
vi /etc/nginx/conf.d/域名.conf

```linux
server {
        listen       80;
        #设置站点域名
        server_name  www.datals.com;
        #指向hugo public 文件夹
        root         /www.datals.com/public;

    location / {
    }

   error_page 404 /404.html;
        location = /40x.html {
    }

    error_page 500 502 503 504 /50x.html;
        location = /50x.html {
    }
}
```

