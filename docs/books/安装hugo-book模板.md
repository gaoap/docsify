# 安装hugo-book模板 <!-- {docsify-ignore-all} -->

第一步：
{{< columns >}}
```tpl
hugo new site mybook; cd mybook
git  clone  https://github.com/alex-shpak/hugo-book.git themes/book
rm -r content 
cp -R themes/book/exampleSite/content 
hugo server  --theme book  // hugo 会生成静态页面。 hugo server 会启动服务，默认端口：1313
```
{{< /columns >}}

执行完后。浏览器打开http://localhost:1313
