修改为阿里云的源
```shell
# 先备份源文件，防止出错
mv /etc/yum.repos.d/CentOS-Base.repo /etc/yum.repos.d/CentOS-Base.repo.bak
&&
wget -O /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-7.repo #这里的7可以改为想要的版本

```
修改为清华大学的源
```shell
# 对于 CentOS 7
sudo sed -e 's|^mirrorlist=|#mirrorlist=|g' \
         -e 's|^#baseurl=http://mirror.centos.org/centos|baseurl=https://mirrors.tuna.tsinghua.edu.cn/centos|g' \
         -i.bak \
         /etc/yum.repos.d/CentOS-*.repo

```
更新
```shell
yum clean all
yum makecache
yum update
```
