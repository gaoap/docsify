# 一步一步搭建权限管理系统（三）

1、实现一个用户登录的基本接口服务。

编写OpfUpmUserController类，提供http服务接口。代码如下：

```Java
//使用@RestController注解，可以省去每个方法体单独使用@ResponseBody。
@RestController
//Swagger定义的接口名称
@Api(tags = "后台用户管理")
//访问此http服务的基础路径
@RequestMapping("/upm/opfUpmUser")
@Slf4j
public class OpfUpmUserController {
    @Value("${opf.upm.jwt.tokenHeader}")
    private String tokenHeader;
    @Value("${opf.upm.jwt.tokenHead}")
    private String tokenHead;
    @Autowired
    private IOpfUpmUserService userService;
    @Autowired
    private IOpfUpmRoleService roleService;
    //Swagger定义的接口名称
    @ApiOperation(value = "登录以后返回token")
    //访问此接口服务的完整路径为：http://ip:9021/upm/opfUpmUser/login需要使用post协议
    @PostMapping(value = "/login")
    //@Validated代表参数校验 @RequestBody 代表参数传递使用JSON格式  ，参数约定见类：OpfUpmUserParam
    //参数如果违反约定，会由src/main/java/com/gaoap/opf/upm/common/exception/
    // GlobalExceptionHandler.java 统一处理（这个类涉及Springboot的机制，回头分析用法）
    public CommonResult login(@Validated @RequestBody OpfUpmUserLoginParam umsUserLoginParam) {
        //log使用过注解@Slf4j定义的。主要是利用Lombok插件，简化代码开发
        log.info("准备登录。。。。。。");
        //请求登录实际处理类。如果返回token，则表明登录成功。没有返回token则登录失败
        String token = userService.login(umsUserLoginParam.getUsername(), umsUserLoginParam.getPassword());
        if (token == null) {
            return CommonResult.validateFailed("用户名或密码错误");
        }
        log.info("准备登录。。。。。。{}", token);
        Map<String, String> tokenMap = new HashMap<>();
        //具体的token内容
        tokenMap.put("token", token);
        // token的前缀
        tokenMap.put("tokenHead", tokenHead);
        //headers中储存token的key
        tokenMap.put("tokenHeader", tokenHeader);
        return CommonResult.success(tokenMap);
        /**
         * 登录成功返回的格式为：
         * {
         *     "code": 200,
         *     "message": "操作成功",
         *     "data": {
         *         "tokenHead": "Bearer",
         *         "tokenHeader": "Authorization",
         *         "token": "eyJhbGciOiJIUzUxMiJ9.eyJzdWIiOiJhZG1pbiIs"
         *     }
         * }
         */
    }
```

以上是Controller类举例。基本代码框架再使用代码生成工具“opf-generate”时已经完成。开发时根据业务需要，增加业务代码即可。例如本例中的“登录”方法。