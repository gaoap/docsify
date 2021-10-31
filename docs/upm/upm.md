# 一步一步搭建权限管理系统（一）

基于SpringBoot+MyBatis-Plus等框架，一步一步开发一个权限管理系统。作为今后快速开发的脚手架。

1、利用MyBatis-Plus的代码生成器，自动生成基础代码。

首先创建一个maven工程。参考引入依赖如下：

```xml
 <dependencies>
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <version>8.0.22</version>
        </dependency>
        <dependency>
            <groupId>com.baomidou</groupId>
            <artifactId>mybatis-plus-boot-starter</artifactId>
            <version>3.4.3.4</version>
        </dependency>
        <dependency>
            <groupId>com.baomidou</groupId>
            <artifactId>mybatis-plus-generator</artifactId>
            <version>3.5.1</version>
        </dependency>
        <!-- velocity 模板引擎, Mybatis Plus 代码生成器需要 -->
        <dependency>
            <groupId>org.apache.velocity</groupId>
            <artifactId>velocity-engine-core</artifactId>
            <version>2.0</version>
        </dependency>
    </dependencies>
```

2、搭建完项目后，工程目录结果如下：(仅供参考)

![image-20211031151221708](https://i.loli.net/2021/10/31/ob87ijUYhq5avNJ.png)

3、创建配置文件generator.properties

参靠路径：src/main/resources/generator.properties

内容如下：

```properties
##数据库连接地址
dataSource.url=jdbc:mysql://localhost:3306/opf_upm?useUnicode=true&characterEncoding=utf8
##数据库账户
dataSource.username=upm
##数据库密码
dataSource.password=upm
##package基础路径
package.base=com.gaoap.opf
##生成package基础路径规则：package.base+“.”+package.moduleName
package.moduleName=upm
##作者
global.author=gaoyd
##输出路径
global.outputDir=E:\\upm
##运行完毕，是否打开输出目录
global.open=true
```

 4、创建自动生成代码java:MyBatisPlusGenerator.java

内容如下：

```java
/**
 * MyBatisPlus代码生成器
 */
public class MyBatisPlusGenerator {

    /**
     * 读取控制台输入内容
     */
    private final Scanner scanner = new Scanner(System.in);

    /**
     * 执行 run
     */
    public static void main(String[] args) throws SQLException {
        Props props = new Props("generator.properties");
        //第一步配置数据源
        FastAutoGenerator.create(initDataSourceConfig())
                // 全局配置
                .globalConfig(builder ->
                        {
                            if (props.getBool("global.open", true) == false) { //输出完毕，是否打开文件夹
                                builder.disableOpenDir();
                            }
                            builder.author(props.getStr("global.author")).fileOverride()
                                    .dateType(DateType.ONLY_DATE)
                                    .outputDir(props.getStr("global.outputDir"));

                        }
                )

                // 包配置
                .packageConfig(builder -> builder.parent(props.getStr("package.base")).moduleName(props.getStr("package.moduleName")))// 设置父包模块名
                // 策略配置
                .strategyConfig((scanner, builder) -> builder.likeTable(new LikeTable(scanner.apply("请输入表名称？逻辑为'%tableName%':"))))
                .strategyConfig(builder ->
                        builder.entityBuilder().disableSerialVersionUID().enableLombok()
                                .mapperBuilder().enableMapperAnnotation()
                                .controllerBuilder().enableRestStyle()
                )
                /*
                    模板引擎配置，默认 Velocity 可选模板引擎 Beetl 或 Freemarker
                   .templateEngine(new BeetlTemplateEngine())
                   .templateEngine(new FreemarkerTemplateEngine())
                 */
                .execute();
    }

    /**
     * 初始化数据源配置
     */
    private static DataSourceConfig.Builder initDataSourceConfig() {
        Props props = new Props("generator.properties");
        DataSourceConfig.Builder DATA_SOURCE_CONFIG = new DataSourceConfig
                .Builder(props.getStr("dataSource.url"), props.getStr("dataSource.username"), props.getStr("dataSource.password"));
        return DATA_SOURCE_CONFIG;
    }


}
```

5、执行MyBatisPlusGenerator程序

执行MyBatisPlusGenerator时，根据提示“请输入表名称？逻辑为'%tableName%':”，输入表名称，即可自动生成基础代码。如：

![image-20211031152702265](https://i.loli.net/2021/10/31/wlBrRq6VemxEvD8.png)

利用代码生成器，基础业务代码生成完毕。

6、参考简图结构：

![image-20211031155946940](https://i.loli.net/2021/10/31/mq95woIUOrAGT3X.png)

7、参考业务sql如下：

```sql
CREATE TABLE `opf_upm_menu` (
  `id` bigint(20) NOT NULL AUTO_INCREMENT,
  `parent_id` bigint(20) DEFAULT NULL COMMENT '父级ID',
  `create_time` datetime DEFAULT NULL COMMENT '创建时间',
  `title` varchar(100) DEFAULT NULL COMMENT '菜单名称',
  `level` int(4) DEFAULT NULL COMMENT '菜单级数',
  `sort` int(4) DEFAULT NULL COMMENT '菜单排序',
  `name` varchar(100) DEFAULT NULL COMMENT '前端名称',
  `icon` varchar(200) DEFAULT NULL COMMENT '前端图标',
  `hidden` int(1) DEFAULT NULL COMMENT '前端隐藏',
  `modify_time` datetime DEFAULT NULL COMMENT '最后更新时间',
  `create_user` bigint(20) DEFAULT NULL COMMENT '创建人',
  `modify_user` bigint(20) DEFAULT NULL COMMENT '最后修改人',
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=29 DEFAULT CHARSET=utf8 COMMENT='后台菜单表';

CREATE TABLE `opf_upm_resource` (
  `id` bigint(20) NOT NULL AUTO_INCREMENT,
  `create_time` datetime DEFAULT NULL COMMENT '创建时间',
  `name` varchar(200) DEFAULT NULL COMMENT '资源名称',
  `url` varchar(200) DEFAULT NULL COMMENT '资源URL',
  `description` varchar(500) DEFAULT NULL COMMENT '描述',
  `category_id` bigint(20) DEFAULT NULL COMMENT '资源分类ID',
  `modify_time` datetime DEFAULT NULL COMMENT '最后更新时间',
  `create_user` bigint(20) DEFAULT NULL COMMENT '创建人',
  `modify_user` bigint(20) DEFAULT NULL COMMENT '最后修改人',
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=32 DEFAULT CHARSET=utf8 COMMENT='后台资源表';

CREATE TABLE `opf_upm_resource_category` (
  `id` bigint(20) NOT NULL AUTO_INCREMENT,
  `create_time` datetime DEFAULT NULL COMMENT '创建时间',
  `name` varchar(200) DEFAULT NULL COMMENT '分类名称',
  `sort` int(4) DEFAULT NULL COMMENT '排序',
  `modify_time` datetime DEFAULT NULL COMMENT '最后更新时间',
  `create_user` bigint(20) DEFAULT NULL COMMENT '创建人',
  `modify_user` bigint(20) DEFAULT NULL COMMENT '最后修改人',
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=9 DEFAULT CHARSET=utf8 COMMENT='资源分类表';

CREATE TABLE `opf_upm_role` (
  `id` bigint(20) NOT NULL AUTO_INCREMENT,
  `name` varchar(100) DEFAULT NULL COMMENT '名称',
  `description` varchar(500) DEFAULT NULL COMMENT '描述',
  `user_count` int(11) DEFAULT NULL COMMENT '后台用户数量',
  `create_time` datetime DEFAULT NULL COMMENT '创建时间',
  `status` int(1) DEFAULT '1' COMMENT '启用状态：0->禁用；1->启用',
  `sort` int(11) DEFAULT '0',
  `modify_time` datetime DEFAULT NULL COMMENT '最后更新时间',
  `create_user` bigint(20) DEFAULT NULL COMMENT '创建人',
  `modify_user` bigint(20) DEFAULT NULL COMMENT '最后修改人',
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=9 DEFAULT CHARSET=utf8 COMMENT='后台用户角色表';

CREATE TABLE `opf_upm_role_menu_relation` (
  `id` bigint(20) NOT NULL AUTO_INCREMENT,
  `role_id` bigint(20) DEFAULT NULL COMMENT '角色ID',
  `menu_id` bigint(20) DEFAULT NULL COMMENT '菜单ID',
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=111 DEFAULT CHARSET=utf8 COMMENT='后台角色菜单关系表';

CREATE TABLE `opf_upm_role_resource_relation` (
  `id` bigint(20) NOT NULL AUTO_INCREMENT,
  `role_id` bigint(20) DEFAULT NULL COMMENT '角色ID',
  `resource_id` bigint(20) DEFAULT NULL COMMENT '资源ID',
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=216 DEFAULT CHARSET=utf8 COMMENT='后台角色资源关系表';

CREATE TABLE `opf_upm_user` (
  `id` bigint(20) NOT NULL AUTO_INCREMENT,
  `username` varchar(64) DEFAULT NULL COMMENT '用户名',
  `password` varchar(64) DEFAULT NULL COMMENT '密码',
  `icon` varchar(500) DEFAULT NULL COMMENT '头像',
  `email` varchar(100) DEFAULT NULL COMMENT '邮箱',
  `nick_name` varchar(200) DEFAULT NULL COMMENT '昵称',
  `note` varchar(500) DEFAULT NULL COMMENT '备注信息',
  `create_time` datetime DEFAULT NULL COMMENT '创建时间',
  `login_time` datetime DEFAULT NULL COMMENT '最后登录时间',
  `status` int(1) DEFAULT '1' COMMENT '帐号启用状态：0->禁用；1->启用',
  `modify_time` datetime DEFAULT NULL COMMENT '最后更新时间',
  `create_user` bigint(20) DEFAULT NULL COMMENT '创建人',
  `modify_user` bigint(20) DEFAULT NULL COMMENT '最后修改人',
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=16 DEFAULT CHARSET=utf8 COMMENT='后台用户表';

CREATE TABLE `opf_upm_user_login_log` (
  `id` bigint(20) NOT NULL AUTO_INCREMENT,
  `user_id` bigint(20) DEFAULT NULL,
  `create_time` datetime DEFAULT NULL,
  `ip` varchar(64) DEFAULT NULL,
  `address` varchar(100) DEFAULT NULL,
  `user_agent` varchar(100) DEFAULT NULL COMMENT '浏览器登录类型',
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=287 DEFAULT CHARSET=utf8 COMMENT='后台用户登录日志表';

CREATE TABLE `opf_upm_user_role_relation` (
  `id` bigint(20) NOT NULL AUTO_INCREMENT,
  `user_id` bigint(20) DEFAULT NULL,
  `role_id` bigint(20) DEFAULT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=40 DEFAULT CHARSET=utf8 COMMENT='后台用户和角色关系表';


```

