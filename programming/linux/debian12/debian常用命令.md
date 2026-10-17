- [添加用户进sudoers](#添加用户进sudoers)
- [查看端口](#查看端口)
- [文件权限](#文件权限)
- [开机自启](#开机自启)
- [update-rc.d命令详解](#update-rcd命令详解)
- [服务](#服务)
### 添加用户进sudoers
```shell
usermod -aG sudo 用户名
```
### 查看端口
```shell
netstat -pnltu

lsof -i
```
### 文件权限
```shell
#查看文件信息
ls -ld 文件或文件夹的路径


chmod [options] mode file_or_directory 
# 其中mode可以是数字形式的权限表示，也可以是符号表示。数字形式的权限由3位数字组成，每位数字代表不同的用户类别的权限：

# 第一位数字代表文件所有者的权限

# 第二位数字代表同一组内用户的权限

# 第三位数字代表其他用户的权限

# 每位数字又是由3位组成：

# 第一位数字代表读（r）权限 4

# 第二位数字代表写（w）权限 2

# 第三位数字代表执行（x）权限 1

# -R 递归
chmod -R 755 directory_name
```
### 开机自启
自启动shell脚本格式（注意，开关机都会执行此脚本）
```shell
#!/bin/sh
### BEGIN INIT INFO
# Default-Start:     2 3 4 5
# Default-Stop:      0 1 6
### END INIT INFO

#"$1"为服务传进来的命令systenctl $1 xxx.service ,如start,restart,stop,status等

case "$1" in
    start)
        wait_for_docker
        ;;
    restart)
        wait_for_docker
        ;;
    *)
        exit 0
        ;;
esac


exit 0
```

```shell
# 开机启动脚本通常放置在/etc/init.d目录下
# 使脚本可执行
sudo chmod +x /etc/init.d/my_script
# 使用update-rc.d命令配置自启动 数字尽量大，不够大，该服务的依赖服务未启动会导致该服务启动失败

查看自启动的服务，可通过这个确定defaults数字
sudo systemctl list-unit-files --type=service

sudo update-rc.d my_script defaults 99
# 删除自启动的命令 
update-rc.d -f test.sh remove

```
### update-rc.d命令详解
```shell
从所有的运行级别中删除指定启动项

update-rc.d -f ＜basename＞ remove
按指定顺序、在指定运行级别中启动或关闭

update-rc.d ＜basename＞ start|stop ＜order＞ ＜runlevels＞

update-rc.d apachectl start 20 2 3 4 5 . stop 20 0 1 6 .
<!-- 解析：表示在2、3、4、5这五个运行级别中，由小到大，第20个开始运行apachectl；在 0 1 6这3个运行级别中，第20个关闭apachectl。这是合并起来的写法，注意它有2个点号，效果等于下面方法： -->

update-rc.d apachectl defaults
# A启动后B才能启动，B关闭后A才关闭

update-rc.d A defaults 80 20

update-rc.d B defaults 90 10
# 启动和关闭顺序为90，级别默认

update-rc.d ＜basename＞ defaults 90
```
### 服务
<https://www.cnblogs.com/crow1840/p/17507194.html>
```shell
# 查看日志
 journalctl -u xxx.service
# 查看2月19号15：10之后的日志

journalctl --since "2021-02-19 15:10:00"
journalctl --since "2021-02-19 15:10:00" --until   "2021-02-19 15:15:00"
journalctl --since yesterday
# 10点到2小时之后的日志
journalctl --since 10:00 --until "2 hour ago"

# 查看实时日志
journalctl -f
# 查看最后n行
journalctl -n 200

```