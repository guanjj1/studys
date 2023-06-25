<!--
 * @Author: guanjiajun www.guanjiajun@ewake.com
 * @Date: 2023-05-30 14:30:01
 * @LastEditors: guanjiajun www.guanjiajun@ewake.com
 * @LastEditTime: 2023-05-30 14:49:56
 * @FilePath: \studys\programming\数据库\mysql\mysql生僻知识点.md
 * @Description: 这是默认设置,请设置`customMade`, 打开koroFileHeader查看配置 进行设置: https://github.com/OBKoro1/koro1FileHeader/wiki/%E9%85%8D%E7%BD%AE#
-->
### MySQL 删除表数据，重置自增 id 为 0 的两个方式：
1、truncate table table_name;
```shell

truncate table `user`;
```
2、delete 配合 alter 语句
```shell
delete from table_name;

ALTER TABLE table_name AUTO_INCREMENT = 1;

alter table user drop column userId;
alter table user add userId int not null primary key auto_increment first;
```
