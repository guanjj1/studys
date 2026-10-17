<!--
 * @Author: guanjiajun www.guanjiajun@ewake.com
 * @Date: 2024-07-18 20:40:59
 * @LastEditors: guanjiajun www.guanjiajun@ewake.com
 * @LastEditTime: 2024-07-18 20:41:12
 * @FilePath: \studys\programming\linux\shell脚本\shell语法.md
 * @Description: 这是默认设置,请设置`customMade`, 打开koroFileHeader查看配置 进行设置: https://github.com/OBKoro1/koro1FileHeader/wiki/%E9%85%8D%E7%BD%AE
-->
case
```shell
case $变量名 in
"值1")
    如果变量的值等于值1, 则执行程序1
    ;;
"值2")
    如果变量的值等于值2, 则执行程序2
    ;;
...
*)
    如果变量的值都不是以上的值, 则执行此程序
    ;;
esac

```