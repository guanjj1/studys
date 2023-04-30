<!--
 * @Author: guanjiajun www.guanjiajun@ewake.com
 * @Date: 2023-04-29 15:02:11
 * @LastEditors: guanjiajun www.guanjiajun@ewake.com
 * @LastEditTime: 2023-04-30 15:24:14
 * @FilePath: \studys\programming\server\python\exe文件打包.md
 * @Description: 这是默认设置,请设置`customMade`, 打开koroFileHeader查看配置 进行设置: https://github.com/OBKoro1/koro1FileHeader/wiki/%E9%85%8D%E7%BD%AE
-->
<https://junyiseo.com/python/479.html>\
<https://blog.csdn.net/weixin_29031161/article/details/113036547>\
<https://blog.csdn.net/BearStarX/article/details/81054134>\
在这里提醒大家，在代码里面尽量不要用import，能from…import…就尽量用这个，因为如果是import的话，在打包的时候，会将整个包都打包到exe里面，没有意义的增大了工具的大小！
```shell
pip install pyinstaller
```

```shell
# 使用-F参数，pyinstaller会将python脚步打包成单个exe文件

# 使用-D参数，pyinstaller会将python脚步打包成一个文件夹，运行程序时，需要进入该文件夹，点击运行相应的可执行程序
# 开源通过 -i 参数指定程序的icon(图标)
# -w 使用Windows子系统执行.当程序启动的时候不会打开命令行(只对Windows有效)
#-p ：指定你自己的python 的所有第三放库路径。
pyinstall -i xxx.ico -n xxx -w -D main.py

```
### 打包找不到第三方库
<https://code84.com/787782.html>\
<https://blog.csdn.net/ldg513783697/article/details/119762461>\
<>
#### 方式1
##### 指定第三方库路径
1.打开pycharm，在设置中找到打包成功后提示缺少的那个第三方库（如我的keyboard），鼠标悬浮在上面，观察到它的安装路径（如D:\Programminglearning\Pycharm\Python-WorkSpace\venv\Lib\site-packages）。\
2.对Python源程序文件.py重新打包，但不同的是加上参数-p和python包所在路径。
```shell
pyinstaller -F -p D:\Programminglearning\Pycharm\Python-WorkSpace\venv\Lib\site-packages TextRecognition.py
```
例子：爬虫找不到ddddocr的部件
<https://blog.csdn.net/qq_19309473/article/details/123692301>\
('G:\\PycharmProjects\\pythonProject1\\venv\\Lib\\site-packages\\onnxruntime\\capi\\onnxruntime_providers_shared.dll', 'onnxruntime\\capi'),('G:\PycharmProjects\pythonProject1\venv\Lib\site-packages\ddddocr\common.onnx','ddddocr')\
修改spec data不生效
```shell
pyinstaller app-t.py --add-data "F:\\Local\\test.db;.\\app\\models\\db"
```
#### 方式2
打开pyinstaller的目录，在hooks目录下创建文件，文件名一定是hook-第三方库.py，比如我用的eventlet库，我就需要创建hook-eventlet.py。
在该文件中添加如下代码：
```python
from PyInstaller.utils.hooks import collect_all

datas, binaries, hiddenimports =collect_all('eventlet') #collect_all的参数一定也是第三方库名，不要写错。

```
### 注意
使用python时，要养成使用 os.path.join 的习惯，这不仅可以避免跨平台的路径坑(windows路径表达与类Unix是不同)，又可以在打包时不会出现相对路径的问题，很多python程序员编写路径喜欢使用 + 号来链接路径，这会增加项目的维护成本