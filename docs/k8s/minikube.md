## k8s之minikube安装

### 环境准备

minikube不允许安装在root用户下,依赖docker环境：

```
添加docker组，将当前用户加入该组，新建用户加入该组
sudo groupadd docker
sudo usermod -aG docker ${USER}
useradd k8stest
passwd k8stest
sudo usermod -aG docker k8stest
service docker restart
su - k8stest
```

下载和安装：官方网站：[minikube start | minikube (k8s.io)](https://minikube.sigs.k8s.io/docs/start/)

1、Installation

```shell
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
sudo install minikube-linux-amd64 /usr/local/bin/minikube
提示：k8stest 不在 sudoers 文件中。此事将被报告。
是因为k8stest不能使用sudo命令。解决方式如下：
回到root账户
执行命令chmod u+w /etc/sudoers或者执行chmod 640 /etc/sudoers
vim /etc/sudoers
由：
root    ALL=(ALL)       ALL
改为：
root    ALL=(ALL)       ALL
k8stest ALL=(ALL)       ALL
将文件权限复原，命令chmod u-w /etc/sudoers或者执行chmod 440 /etc/sudoers
```

2、 Start your cluster

```shell
minikube start
```

3、Interact with your cluster

如果本地已经安装kubectl 

```
kubectl get po -A
```

也可以通过minikube安装合适的版本

```
minikube kubectl -- get po -A
```

简化命令，使用别名

```
alias kubectl="minikube kubectl --"
```

启动面板仪表盘

```
minikube dashboard
```

4、 Deploy applications

Create a sample deployment and expose it on port 8080:

```shell
kubectl create deployment hello-minikube --image=k8s.gcr.io/echoserver:1.4
kubectl expose deployment hello-minikube --type=NodePort --port=8080
```

It may take a moment, but your deployment will soon show up when you run:

```shell
kubectl get services hello-minikube
```

The easiest way to access this service is to let minikube launch a web browser for you:

```shell
minikube service hello-minikube
```

Alternatively, use kubectl to forward the port:

```shell
kubectl port-forward service/hello-minikube 7080:8080
```

Tada! Your application is now available at http://localhost:7080/.

You should be able to see the request metadata from nginx such as the `CLIENT VALUES`, `SERVER VALUES`, `HEADERS RECEIVED` and the `BODY` in the application output. Try changing the path of the request and observe the changes in the `CLIENT VALUES`. Similarly, you can do a POST request to the same and observe the body show up in `BODY` section of the output.

### LoadBalancer deployments

To access a LoadBalancer deployment, use the “minikube tunnel” command. Here is an example deployment:

```shell
kubectl create deployment balanced --image=k8s.gcr.io/echoserver:1.4  
kubectl expose deployment balanced --type=LoadBalancer --port=8080
```

In another window, start the tunnel to create a routable IP for the ‘balanced’ deployment:

```shell
minikube tunnel
```

To find the routable IP, run this command and examine the `EXTERNAL-IP` column:

```shell
kubectl get services balanced
```

Your deployment is now available at <EXTERNAL-IP>:8080

## **5**、Manage your cluster

Pause Kubernetes without impacting deployed applications:

```shell
minikube pause
```

Unpause a paused instance:

```shell
minikube unpause
```

Halt the cluster:

```shell
minikube stop
```

Increase the default memory limit (requires a restart):

```shell
minikube config set memory 16384
```

Browse the catalog of easily installed Kubernetes services:

```shell
minikube addons list
```

Create a second cluster running an older Kubernetes release:

```shell
minikube start -p aged --kubernetes-version=v1.16.1
```

Delete all of the minikube clusters:

```shell
minikube delete --all
```

##  





补充alias命令：

**1.设置别名**

alias 别名=’原命令 -选项/参数’

例如:

```
alias ll='ls -lt'1
```

这样设置了ls -lt命令的别名是ll，在终端输入ll时，则相当于输入了ls -lt命令

注意： 在定义别名时，等号两边不能有空格，否则[shell](https://www.linuxcool.com/)不能决定您需要做什么。仅在命令中包含空格或特殊字符时才需要引号。如果键入不带任何参数的alias 命令，将显示所有已定义的别名。

**2.查看已经设置的别名列表**

```
alias -p1
```

**3.删除别名**

unalias 别名1

例如：

```
unalias ll1
```

**4.设置别名每次登入可用**

alias命令只作用于当次登入的操作。如果想每次登入都能使用这些命令的别名，则可以把相应的alias命令存放在 ~/.bashrc 文件中。

打开~/.bashrc文件，输入要设置的alias命令，保存，然后运行

```
source ~/.bashrc1
```

如果这样还不行，表示没有~/.bash_profile文件或文件中没有执行~/.bashrc文件

可以在~/.bash_profile中加入命令 source ~/.bashrc 后保存

这样就可以每次登入后都可以使用设置好的命令别名。