# 安装hugo及安装hugo-book主题 <!-- {docsify-ignore-all} -->

## 安装hugo:

1、首先去页面[Releases · gohugoio/hugo · GitHub](https://github.com/gohugoio/hugo/releases) 下载对应版本的程序包。例如：windows下的可以下载列表中[hugo_extended_0.88.1_Windows-64bit.zip](https://github.com/gohugoio/hugo/releases/download/v0.88.1/hugo_extended_0.88.1_Windows-64bit.zip) 这里建议下载带扩展包的程序。

2、下载到本地后解压，得到hugo.exe 文件。

3、配置环境变量

假如hugo.exe 解压到目录E:\hugo\bin下。在环境变量中配置PATH ，增加 E:\hugo\bin 即可。

4、初始化网站，并下载主题hugo-book

```bash
hugo new site mybook; cd mybook
git  clone  https://github.com/alex-shpak/hugo-book.git themes/book
rm -r content 
cp -R themes/book/exampleSite/content 
hugo server  --theme book  // hugo 会生成静态页面。 hugo server 会启动服务，默认端口：1313
```
执行完后。浏览器打开http://localhost:1313

## 生成静态页面及Nginx配置：

1、执行hugo命令，站点目录下会新建文件夹public/，生成的所有静态网站页面都会存储到这个目录，使用nginx作为web服务配置root dir 指向public/ 即可；

2、修改nginx配置，添加监听域名
vi /etc/nginx/conf.d/域名.conf

```bash
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





