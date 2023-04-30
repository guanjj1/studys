<!--
 * @Author: guanjiajun www.guanjiajun@ewake.com
 * @Date: 2023-04-01 19:26:23
 * @LastEditors: guanjiajun www.guanjiajun@ewake.com
 * @LastEditTime: 2023-04-07 18:11:00
 * @FilePath: \studys\programming\容器技术\docker\mysql.md
 * @Description: 这是默认设置,请设置`customMade`, 打开koroFileHeader查看配置 进行设置: https://github.com/OBKoro1/koro1FileHeader/wiki/%E9%85%8D%E7%BD%AE
-->
### 拉取镜像
```shell
# 拉取镜像
docker pull mysql
 
# 或者
docker pull mysql:latest
 
# 以上两个命令是一致的，默认拉取的就是 latest 版本的
 
# 我们还可以用下面的命令来查看可用版本：
docker search mysql
```
### 查看镜像
```shell
docker images
```
### 启动镜像（经测试，密码设置失败，密码默认为空）
```shell
#确保本地端口可访问
#开放端口用 --add-port
firewall-cmd --permanent --add-port=8080/tcp
#移除端口用 --remove-port
firewall-cmd --permanent --remove-port=8080/tcp
#刷新规则用 --reload
firewall-cmd --reload
#查询端口是否开放用 --query-port
firewall-cmd --query-port=8080/tcp
#建议重启
docker run -di -p 3306:3306 --name mysql --restart=always -v /home/mysql/log:/var/log/mysql -v /home/mysql/data:/var/lib/mysql -v /home/mysql/conf:/etc/mysql -v /home/mysql/mysql-files:/var/lib/mysql-files -e MYSQL_ROOT_PASSWORD=ewake666 mysql:latest

#限制日志文件大小
--log-opt max-size=100m --log-opt max-file=2
```
### 进入mysql容器—并登陆mysql
```shell
格式：docker exec -it   mysql名称   bash

进入mysql容器操作台命令：docker exec -it mysql bash

登录mysql命令：mysql -u root -p

		输入密码：

```
### 修改密码，开启远程访问权限
```shell
命令：use mysql;

命令：select host,user，authentication_string from user;
清空密码
update mysql.user set authentication_string='' where User='root';

flush privileges;
#设置root新的密码 %为host 可能是localhost
命令：ALTER USER 'root'@'%' IDENTIFIED WITH mysql_native_password BY '123456';
#设置远程.可以是具体ip
update user set host='%' where user='';
flush privileges;
GRANT ALL PRIVILEGES ON *.* TO 'root'@'%';

命令：flush privileges;

把root用户的密码改成 mysql_native_password 模式，即可远程连接


		#创建一个账号-admin，用来进行远程访问；
		CREATE USER 'admin'@'%' IDENTIFIED BY '123456';
		 
		 
		 赋予所有权限给之前创建的账号:admin
		GRANT ALL ON *.* TO 'admin'@'%';
		 
		 
		 确认使用密码{123456}登录此账号{admin}
		 密码尽量复杂，安全性更高。
		ALTER USER 'admin'@'%' IDENTIFIED WITH mysql_native_password BY '123456';

		 刷新权限
		FLUSH PRIVILEGES;

```
### 查看docker日志
```shell
命令：docker logs -f --tail 10 a4dac74d48f7
```

