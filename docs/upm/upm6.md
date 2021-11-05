# 一步一步搭建权限管理系统（六）

配置接口文档：Dockerfile

配置Dockerfile如下

```dockerfile
###这个是springboot 官方推荐的分层方式。
FROM openjdk:17 as builder
WORKDIR application
ARG JAR_FILE=target/*.jar    #maven的打包输出的目录为target。这个是对应的输出目录
COPY ${JAR_FILE} application.jar
RUN java -Djarmode=layertools -jar application.jar extract

################################

FROM openjdk:17
MAINTAINER gaoyd <gaoyd@gaoap.com>
WORKDIR application
COPY --from=builder application/dependencies/ ./
COPY --from=builder application/spring-boot-loader/ ./
COPY --from=builder application/snapshot-dependencies/ ./
COPY --from=builder application/application/ ./

# JVM_XMS and JVM_XMX configs deprecated for removal in halov1.4.4
ENV JVM_XMS="256m" \
    JVM_XMX="256m" \
    JVM_OPTS="-Xmx256m -Xms256m" \
    TZ=Asia/Shanghai \
    PARAMS_JAVA="" \
    PARAMS_SPRING=""
    #PARAMS给我们要传的参数一个初始值 主要是传递：-Dspring.profiles.active=

RUN ln -sf /usr/share/zoneinfo/$TZ /etc/localtime \
    && echo $TZ > /etc/timezone

ENTRYPOINT java -Xms${JVM_XMS} -Xmx${JVM_XMX} ${JVM_OPTS} -Djava.security.egd=file:/dev/./urandom ${PARAMS_JAVA} org.springframework.boot.loader.JarLauncher ${PARAMS_SPRING}



#说明：
#
#TZ： 时区设置，而 Asia/Shanghai 表示使用中国上海时区。
#
#-Djarmode=layertools： 指定构建 Jar 的模式。
#
#extract： 从 Jar 包中提取构建镜像所需的内容。
#
#-from=builder 多级镜像构建中，从上一级镜像复制文件到当前镜像中。
```

pom.xml配置：

低版本的springboot，需要配置pom.xml进行分层设置。高版本不需要。2.4.0以下版本，不需要以下配置。例如：

```xml
<plugin>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-maven-plugin</artifactId>
    <version>2.3.3.RELEASE</version>
    <configuration>
        <layers>
            <enabled>true</enabled>
        </layers>
    </configuration>
</plugin>
```

2.4.0以下版本，不需要上述配置。例如：

```xml
<plugin>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-maven-plugin</artifactId>
</plugin>
```

在pom.xml中添加docker插件：

```xml
 <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
                <configuration>
                    <excludes>
                        <exclude>
                            <groupId>org.projectlombok</groupId>
                            <artifactId>lombok</artifactId>
                        </exclude>
                    </excludes>
                </configuration>
            </plugin>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
                <configuration>
                    <layers>
                        <enabled>true</enabled>
                    </layers>
                </configuration>
            </plugin>
            <plugin>
                <groupId>io.fabric8</groupId>
                <artifactId>docker-maven-plugin</artifactId>
                <version>0.37.0</version>
                <!--全局配置-->
                <configuration>
                    <!--配置远程docker守护进程url-->
                    <dockerHost>http://192.168.31.82:2375</dockerHost>
                    <!--认证配置,用于私有registry认证-->
                    <!--                    <authConfig>-->
                    <!--                        <username>admin</username>-->
                    <!--                        <password>Harbor12345</password>-->
                    <!--                    </authConfig>-->
                    <!--镜像相关配置,支持多镜像-->
                    <images>
                        <!-- 单个镜像配置 -->
                        <image>
                            <!--镜像名(含版本号)-->
                            <name>registry.cn-beijing.aliyuncs.com/gaoap/${project.name}:${project.version}</name>
                            <!--registry地址,用于推送,拉取镜像-->
                            <!--  <registry>registry.cn-beijing.aliyuncs.com</registry>-->
                            <!--镜像build相关配置-->
                            <build>
                                <!--使用dockerFile文件-->
                                <dockerFile>${project.basedir}/Dockerfile</dockerFile>
                            </build>
                        </image>
                    </images>
                    <authConfig>
                            <!--远程仓库账号-->
                        <username>${auth.username}</username>
                             <!--远程仓库密码-->
                        <password>${auth.password}</password>
                    </authConfig>
                </configuration>
            </plugin>
        </plugins>
    </build>
```

需要远程访问Docker守护进程，需要启用`tcp`套接字。

### 修改docker.service

```shell
vim /usr/lib/systemd/system/docker.service
```

在[Service]部分，修改ExecStart参数，在最后增加-H tcp://0.0.0.0:2375，监听所有网络接口上的2375端口。

如：

```shell
ExecStart=/usr/bin/dockerd -H fd:// --containerd=/run/containerd/containerd.sock -H tcp://0.0.0.0:2375
```

重新加载配置文件和启动服务

```shell
systemctl daemon-reload && systemctl restart docker
```

执行命令参考如下：

```shell
mvn clean package docker:build -DskipTests 
mvn docker:push -DskipTests 
```

启动docker镜像命令：

```shell
docker run  --name opf-upm -d -p 9021:9021  gaoap/opf-upm:0.0.1-SNAPSHOT
```

期中：

gaoap/opf-upm:0.0.1-SNAPSHOT 为发布的镜像名称

-p 9021:9021  格式为 -p 对外服务暴露端口:docker内部服务端口
