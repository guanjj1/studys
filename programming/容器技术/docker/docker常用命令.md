# docker安装
<https://blog.csdn.net/PyongSen/article/details/123053374>
## centos安装docker
<https://www.cnblogs.com/shineen/p/16440302.html>\
<https://www.cnblogs.com/iforeverhz/p/16255720.html>\
<https://cloud.tencent.com/developer/article/1701451>\
<https://www.coonote.com/docker/centos-install-docker.html>
### docker安装
```shell
sudo yum update
#安装需要的软件包， yum-util 提供yum-config-manager功能，另两个是devicemapper驱动依赖
sudo yum install -y yum-utils device-mapper-persistent-data 1vm2
#yum源
yum-config-manager --add-repo http://download.docker.com/linux/centos/docker-ce.repo
#推荐
sudo yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
#查看可用版本
yum list docker-ce --showduplicates | sort -r
#选择一个版本并安装
yum install docker-ce-版本号
#最新版
sudo yum install docker-ce

docker -v
```
### docker添加镜像服务
```shell
sudo vim /etc/docker/daemon.json

#添加
{
  "registry-mirrors": ["https://y0qd3iq.mirror.aliyuncs.com"]
}
```
### docker启动
```shell
systemctl start docker
```
### 查看docker状态
```shell
systemctl status docker
```
### 停止docker
```shell
systemctl stop docker
```
### 重启docker
```shell
systemctl restart docker
```
### 设置开机自启动
```shell
systemctl enable docker
```

## docker常用命令
### 镜像常用命令
#### 查看镜像
```shell
docker images
```
#### 搜索镜像
```shll
docker search 镜像名
```
#### 拉取镜像
```shell
docker pull 镜像名称
```
#### 删除镜像
```shell
docker rmi 镜像id
```
#### 删除多个镜像
```shell
docker rmi -f 镜像名A:tag 镜像名B:tag
```
#### 删除所有镜像
```shell
docker rmi -f $(docker images -aq)
```
### 容器常用命令
#### 查看正在运行的容器
```shell
docker ps

# 查看所有
docker ps -a
```
### docker时区
```shell
# 容器中设置时区一直是独立于宿主机的。 可以通过挂载 /etc/timezone 的方式保持与宿主机时间一致。
docker run --rm -it -v /etc/timezone:/etc/timezone debian bash

# 报错
# 原因是centos7.6中/etc/timezone是一个文件夹,而不是一个文件,执行如下命令:
 
echo 'Asia/Shanghai' > /etc/timezone/timezone
 
# 然后执行
docker run -d --name sys-app  -v /etc/timezone/timezone:/etc/timezone  -v 
/etc/localtime:/etc/localtime  -p 8001:8001  --restart=always --net=host 
 sys-app:latest


docker run -p 3306:3306 --name mysql -v /etc/localtime:/etc/localtime

docker run  -e TZ="Asia/Shanghai" -p 8090:8090 -d --name ch ch/ch

```
#### 创建启动容器
命令及参数\
命令：docker run\
-i: 表示运行容器\
-t: 表示容器启动后会进入其命令行\
--name: 为创建的容器命名\
-v: 表示目录映射关系\
-d: 会创建一个守护容器在后台运行（这样创建容器不会自动登录容器，如果只加-i-t两个参数，创建后自动登录容器）\
-p: 表示端口映射，前者是宿主主机端口，后者是容器内的映射端口\
--restart=always 便表示，该容器随docker服务启动而自动启动
```shell
docker run [选项] 镜像名
选项
-d 后台运行
-it 提供容器交互
--name 设置容器名
--cpus 设置cpu个数
--env 设置环境变量
--mount type=bind,source=/root/target,target=/app或者--mount type=tmpfs,destination=/app 
--volume <host>:<container>:[rw|ro]挂载一个磁盘卷 例如 --volume /home/hyzhou/docker:/data:rw
--restart 设置重启策略on-failure,no,always
--privileged 使用该参数，container内的root拥有真正的root权限。否则，container内的root只是外部的一个普通用户权限。privileged启动的容器，可以看到很多host上的设备，并且可以执行mount。甚至允许你在docker容器中启动docker容器。
```
```shell
# 修改时区
docker run  -e TZ="Asia/Shanghai" -p 8090:8090 -d --name ch ch/ch

```
```shell
docker run -itd --name redis002 -p 8888:6379 --restart=always  redis:5.0.5 /bin/bash
```
限制日志文件的大小
启动容器时，可以通过参数设置日志文件的大小、日志文件的格式。
```shell
docker run -it --log-opt max-size=10m --log-opt max-file=3 alpine ash
```
(1)交互式创建容器
```shell
docker run -it --name=容器名称(取的名字) 镜像名称:标签 /bin/bash

docker run -it --name=mycentos centos:7 /bin/bash
docker run ‐‐name mytomcat ‐d tomcat:latest
#查看启动的容易
docker ps 
```
(2)守护式方式创建容器
```shell
docker run -di name=容器名称 镜像名称:标签
```
登录守护式方式
```shell
docker exec -it 容器名称（或者容器id) /bin/bash(命令行解释器，也可能为/bin/sh)
```
#### 常见较好的启动方式
```shell
# 运行一个docker redis 容器 进行 端口映射 两个数据卷挂载 设置开机自启动
docker run -d -p 6379:6379 --name redis505 --restart=always  -v /var/lib/redis/data/:/data -v /var/lib/redis/conf/:/usr/local/etc/redis/redis.conf  redis:5.0.5 --requirepass "password"
```
#### 停止容器
```shell
docker stop 容器名称（或者容器id) 
```
#### 启动容器
```shell
docker start 容器名称（或者容器id) 
``` 
#### 文件拷贝
```shell
#docker cp 容器ID/名称:文件路径  要拷贝到外部的路径   |     要拷贝到外部的路径  容器ID/名称:文件路径
#从容器内 拷出
docker cp 容器ID/名称: 容器内路径  容器外路径
#从外部 拷贝文件到容器内
docker  cp 容器外路径 容器ID/名称: 容器内路径


``` 
#### 目录挂载(同步)[启动过镜像后进行目录挂载](#mount)
```shell
#-v 宿主机文件存储位置:容器内文件位置

docker run -di --name=mycentos3 -v /usr/local/myhtml:/usr/localmyhtml centos:7 
``` 
#### 查看容器ip地址
```shell
docker inspect 容器名称
``` 
#### 删除容器
```shell
#删除一个容器
docker rm -f 容器名/容器ID
#删除多个容器 空格隔开要删除的容器名或容器ID
docker rm -f 容器名/容器ID 容器名/容器ID 容器名/容器ID
#删除全部容器
docker rm -f $(docker ps -aq)

``` 
#### 查看容器日志
```shell

docker logs 容器名称/id

``` 
#### 容器端口与服务器端口映射（端口映射前要停止，删除容器）
```shell
docker run -itd --name redis002 -p 服务器端口:容器端口 redis:5.0.5 /bin/bash

``` 
#### 从容器内 退出到自己服务器中
```shell
#-----直接退出  未添加 -d(持久化运行容器) 时 执行此参数 容器会被关闭  
exit
# 优雅退出 --- 无论是否添加-d 参数 执行此命令容器都不会被关闭
Ctrl + p + q
``` 
### 迁移与备份
#### 容器保存为镜像
```shell
docker commit mynginx(容器名称) mynginx_i（镜像名称）
``` 
#### 镜像备份
```shell
docker save -o mynginx.tar(导出的文件) mynginx_i（镜像名称）
``` 
#### 镜像恢复
```shell
docker save -o mynginx.tar(导出的文件) mynginx_i（镜像名称）
``` 

### 启动过镜像后进行目录挂载<a id="mount"></a>
#### 查看要新增挂载的容器id
```shell
docker ps
```
#### 进入要新增文件夹挂载的目录
```shell
cd /var/lib/docker/containers/
ll
cd 容器id为前缀的目录
```
#### 关闭容器、关闭docker
```shell
systemctl stop docker
```
#### 修改config.v2.json
```shell
#添加映射
{
    ......
     
    "MountPoints": {
        ......,
        "/opt/shardingsphere-proxy/logs": {
            "Source": "/atguigu/server/proxy-a/logs",
            "Destination": "/opt/shardingsphere-proxy/logs",
            "RW": true,
            "Name": "",
            "Driver": "",
            "Type": "bind",
            "Propagation": "rprivate",
            "Spec": {
                "Type": "bind",
                "Source": "/atguigu/server/proxy-a/logs",
                "Target": "/opt/shardingsphere-proxy/logs"
            },
            "SkipMountpointCreation": false
        }
    },
     
    ......
     
}
```
#### 修改hostconfig.json
```shell
{
    "Binds": [
        ......,
        "/atguigu/server/proxy-a/logs:/opt/shardingsphere-proxy/logs"
    ],
    ......
}
```
#### 重启docker与容器
```shell
systemctl start docker
docker start 容器id
```
#### 注意
config.v2.json和hostconfig.json文件修改之前，需要关闭docker与容器！如果先修改了这两个文件，再关闭docker，这两个文件中的内容会被重置掉，导致新增的目录映射失效 （自己在这里折腾了很久才发现的！）