<!--
 * @Author: guanjiajun www.guanjiajun@ewake.com
 * @Date: 2024-07-15 19:07:15
 * @LastEditors: guanjiajun www.guanjiajun@ewake.com
 * @LastEditTime: 2024-07-15 19:07:44
 * @FilePath: \studys\programming\linux\debian12\debian 12 设置启动界面（文本或图形.md
 * @Description: 这是默认设置,请设置`customMade`, 打开koroFileHeader查看配置 进行设置: https://github.com/OBKoro1/koro1FileHeader/wiki/%E9%85%8D%E7%BD%AE
-->
```shell
# 查看启动界面现状：
systemctl get-default
# 设置启动界面为文本模式：
systemctl set-default multi-user.target

# 设置启动界面为图形模式：
systemctl set-default graphical.target

reboot
```