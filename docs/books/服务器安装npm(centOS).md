# 服务器安装npm(centOS) <!-- {docsify-ignore-all} -->

下载

```bash
cd /usr/local/
wget https://npm.taobao.org/mirrors/node/v10.14.1/node-v10.14.1-linux-x64.tar.gz
```

解压

```
tar -xvf node-v10.14.1-linux-x64.tar.gz
```

转移

```
mv node-v10.14.1-linux-x64 node
```

配置环境变量

```
vim /etc/profile
```

注意目录位置：

```
#set for nodejs  
export NODE_HOME=/usr/local/node  
export PATH=$NODE_HOME/bin:$PATH
```

生效配置文件

```
source /etc/profile
node -v
npm -v
```

npm阿里云镜像源加速

```text
npm config set registry "https://registry.npm.taobao.org"
```

验证npm设置阿里云源是否设置成功

```text
npm config get registry
```

![img](https://pic3.zhimg.com/80/v2-f376726bd6089cf267114e78e78a28ee_720w.png)



 [快速开始](https://docsify.js.org/#/zh-cn/quickstart?id=快速开始)

推荐全局安装 `docsify-cli` 工具，可以方便地创建及在本地预览生成的文档。

```bash
npm i docsify-cli -g
```

## [初始化项目](https://docsify.js.org/#/zh-cn/quickstart?id=初始化项目)

如果想在项目的 `./docs` 目录里写文档，直接通过 `init` 初始化项目。

```bash
docsify init ./docs
```

## [开始写文档](https://docsify.js.org/#/zh-cn/quickstart?id=开始写文档)

初始化成功后，可以看到 `./docs` 目录下创建的几个文件

- `index.html` 入口文件
- `README.md` 会做为主页内容渲染
- `.nojekyll` 用于阻止 GitHub Pages 忽略掉下划线开头的文件

直接编辑 `docs/README.md` 就能更新文档内容，当然也可以[添加更多页面](https://docsify.js.org/#/zh-cn/more-pages)。





## Docker

- 创建 docsify 的文件

你需要准备好初始文件，而不是在容器中制作。
请参阅 快速开始 部分，了解如何手动或使用 [docsify-cli](https://github.com/docsifyjs/docsify-cli) 创建这些文件。

```sh
index.html
README.md
```

- 创建 Dockerfile

```dockerfile
FROM node:latest
LABEL description="A demo Dockerfile for build Docsify."
WORKDIR /docs
RUN npm install -g docsify-cli@latest
EXPOSE 3000/tcp
ENTRYPOINT docsify serve .
```

创建成功后当前的目录结构应该是这样的：

```sh
index.html
README.md
Dockerfile
```

- 构建 docker 镜像

```sh
docker build -f Dockerfile -t docsify/demo .
```

- 运行 docker 镜像

```sh
docker run -itp 3000:3000 --name=docsify -v $(pwd):/docs docsify/demo
```