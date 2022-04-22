# 不安装Docker生成image的插件介绍

今天找到了一个maven插件，不用本地安装Docker即可生成image，并发布到指定仓库。记录如下：

```XML
 <build>
        <plugins>
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
                <groupId>com.google.cloud.tools</groupId>
                <artifactId>jib-maven-plugin</artifactId>
                <version>3.2.1</version>
                <configuration>
                    <!-- 拉取所需的基础镜像 - 这里用于运行springboot项目 -->
                    <from>
                        <image>openjdk:alpine</image>
                    </from>
                    <!-- 最后生成的镜像配置 -->
                    <to>
                        <!-- push docer-hub官方仓库。用户名/镜像名：版本号， -->
                        <image>registry.cn-beijing.aliyuncs.com/gaoap/${project.name}</image>
                        <!-- 如果是阿里云的容器镜像仓库，则使用容器的配置 前缀/命名空间/仓库名 -->
                        <!--<image>registry.cn-chengdu.aliyuncs.com/renbaojia/ctfo</image>-->
                        <tags>
                            <!--版本号-->
                            <tag>${project.version}</tag>
                        </tags>
                        <auth>
                            <!--在docker-hub或者阿里云上的账号和密码-->
                            <username>hxxxxxxxx@aliyun.com</username>
                            <password>Qianxxxxxx</password>
                        </auth>
                    </to>
                    <container>
<!--                        springboot项目的入口类-->
                        <mainClass>com.gaoap.opf.upm.OpfUpmApplication</mainClass>
                        <useCurrentTimestamp>true</useCurrentTimestamp>
                        <ports>
<!--                            指定镜像端口 , 这里没用 docfile的操作-->
                            <port>80</port>
                        </ports>
                    </container>
                </configuration>
            </plugin>
        </plugins>
    </build>
```

执行发布命令：

```shell
  mvn compile jib:build 
  或者
  mvn compile jib:build   -Djib.to.auth.username=hixxxxxx@aliyun.com  -Djib.to.auth.password=xxxxx
```

拉取阿里私有image方式：

```shell
docker login --username=hxxxxxxx@aliyun.com registry.cn-beijing.aliyuncs.com
docker pull  registry.cn-beijing.aliyuncs.com/gaoap/opf-upm_crir:0.0.1-SNAPSHOT
```

额外记录一个启动命令：

```shell
docker run  -d  --name opf-upm \
-p 9021:9021  \
-v /opt/webapps/gao.api/test/doc:/opt/webapps/gao.api/test/doc \
-v /opt/webapps/gao.api/test/file:/opt/webapps/gao.api/test/file \
-v /opf-upm:/opf-upm \
--restart=always \
 registry.cn-beijing.aliyuncs.com/gaoap/opf-upm_crir:0.0.1-SNAPSHOT 
```

