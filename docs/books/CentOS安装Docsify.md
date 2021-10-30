# CentOS安装Docsity  <!-- {docsify-ignore-all} -->

## 第一步安装NPM: 

下载：

```bash
cd /usr/local/
wget https://npm.taobao.org/mirrors/node/v10.14.1/node-v10.14.1-linux-x64.tar.gz
```

解压

```bash
tar -xvf node-v10.14.1-linux-x64.tar.gz
```

转移

```bash
mv node-v10.14.1-linux-x64 node
```

配置环境变量

```bash
vim /etc/profile
```

注意目录位置：

```bash
#set for nodejs  
export NODE_HOME=/usr/local/node  
export PATH=$NODE_HOME/bin:$PATH
```

生效配置文件

```bash
source /etc/profile
node -v
npm -v
```

npm阿里云镜像源加速

```bash
npm config set registry "https://registry.npm.taobao.org"
```

验证npm设置阿里云源是否设置成功

```bash
npm config get registry
```

返回 https://registry.npm.taobao.org 即表示设置阿里源成功

以上npm安装和设置完毕。

## 第二步：安装Docsify

推荐全局安装 `docsify-cli` 工具，可以方便地创建及在本地预览生成的文档。

```bash
npm i docsify-cli -g
```

初始化项目

如果想在项目的 `./docs` 目录里写文档，直接通过 `init` 初始化项目。

```bash
docsify init ./docs
```

开始写文档

初始化成功后，可以看到 `./docs` 目录下创建的几个文件

- `index.html` 入口文件
- `README.md` 会做为主页内容渲染
- `.nojekyll` 用于阻止 GitHub Pages 忽略掉下划线开头的文件

直接编辑 docs/README.md 就能更新文档内容，当然也可以添加更多页面。

本地预览命令：

```
docsify serve docs
```

默认端口为：3000

浏览器访问: http://IP:3000/ 即可访问。

## 第三步：Docker方式安装

1. 创建 docsify 的文件

你需要准备好初始文件，而不是在容器中制作。
请参阅 快速开始 部分，了解如何手动或使用 [docsify-cli](https://github.com/docsifyjs/docsify-cli) 创建这些文件。

```sh
index.html
README.md
```

1. 创建 Dockerfile

创建Docker需要提前准备好index.html和README.md文件。

```docker
FROM node:latest
LABEL description="A demo Dockerfile for build Docsify."
WORKDIR /docs
RUN npm install -g docsify-cli@latest
EXPOSE 3000/tcp
ENTRYPOINT docsify serve .
```

2. 创建成功后当前的目录结构应该是这样的：


```sh
index.html
README.md
Dockerfile
```

3. 构建 docker 镜像

```bash
docker build -f Dockerfile -t docsify/demo .
```

4. 运行 docker 镜像

```bash
docker run -itp 3000:3000 --name=docsify -v $(pwd):/docs docsify/demo
```

浏览器访问: http://IP:3000/ 即可访问。

