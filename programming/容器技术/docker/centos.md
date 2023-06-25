<https://blog.csdn.net/qq_38616723/article/details/125276545>\
#### 拉取镜像
```shell
docker pull centos:centos7

```
#### 查看本地镜像，验证是否安装成功
```shell
docker images

```
#### 运行容器
```shell
docker run -di --name centos-test -p 60001:22  --privileged centos:centos7 /usr/sbin/init 
# --privileged 给予root权限 docker中运行docker
```
#### 进入到Centos容器
```shell
docker exec -it centos-test /bin/bash

```
#### 安装ssh服务和网络必须软件
```shell
 yum install net-tools.x86_64 -y
 yum install -y openssh-server

```
#### 安装完后重启SSH服务:
```shell
systemctl restart sshd

```
#### 安装passwd软件（用于设置centos用户密码，便于用Xshell连)
```shell
yum install passwd -y 

```
#### 设置root用户密码
```shell
passwd root

```