# swagger-ui测试接口如何使用token 

第一步：打开地址 http://localhost:9021/swagger-ui/#/

出现如下界面表示swagger服务成功：

![image-20220321114713594](image-20220321114713594.png)

第二步：打开地址：http://localhost:9021/swagger-ui/#/后台用户管理/loginUsingPOST  

点击“ Try it out ” ,填写参数：

![image-20220321115005242](image-20220321115005242.png)

点击执行“Execute” 。获取token。

![image-20220321115121185](image-20220321115121185.png)

期中的：eyJhbGciOiJIUzUxMiJ9.eyJzdWIiOiJhZG1pbiIsImNyZWF0ZWQiOjE2NDc4MzQ2NTgzNTYsImV4cCI6MTY1NTAzNDY1OH0.nG7t53pfyC_dBjoDcIHx9IviEo9KCafOGrX_Dhmfb6EnYywTn8NUW7GJ24As3BWh8nPsFuuC2YRSrcJCtrmDcQ

就是所需的token

第三步：在swagger添加token认证信息

![image-20220321115329546](image-20220321115329546.png)

点击页面中的“Authorize”按钮：

在弹出页面中填写token：

![image-20220321115440687](image-20220321115440687.png)

点击"Authorize",完成登录。如图：

![image-20220321115526884](image-20220321115526884.png)

再访问其他所有的API接口，就会包含用户的token信息，完成权限认证。
