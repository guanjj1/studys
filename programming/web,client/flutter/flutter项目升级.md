<!--
 * @Author: guanjiajun www.guanjiajun@ewake.com
 * @Date: 2023-08-25 17:46:07
 * @LastEditors: guanjiajun www.guanjiajun@ewake.com
 * @LastEditTime: 2023-08-25 18:13:15
 * @FilePath: \studys\programming\web,client\flutter\flutter项目升级.md
 * @Description: 这是默认设置,请设置`customMade`, 打开koroFileHeader查看配置 进行设置: https://github.com/OBKoro1/koro1FileHeader/wiki/%E9%85%8D%E7%BD%AE
-->
<https://flutter.cn/docs/release/upgrade>
### 升级 Flutter SDK
要查看你当前使用的哪个渠道，使用下面的命令：
```shell
flutter channel
```
要切换到其它渠道，使用 flutter channel <channel-name>。当你切换了渠道以后，使用 flutter upgrade 下载 Flutter SDK 和依赖的 packages。例如：
```shell
flutter channel beta
flutter upgrade
```
建议使用stable
如果你需要某个特定的 Flutter SDK 版本, 你可以从 SDK 版本 页面下载.
<https://flutter.cn/docs/release/archive?tab=windows>
### 仅更新 packages
如果你修改了 pubspec.yaml 文件，或者想仅更新项目依赖的 packages，而不是同时更新 packages 和 Flutter SDK，可以选择使用下面提到的 flutter pub 命令。

为了把 pubspec.yaml 文件里列出的所有依赖更新到 最新的兼容版本 ，可以使用使用 upgrade 命令：

```shell
 flutter pub upgrade
 ```
为了把 pubspec.yaml 文件里列出的所有依赖更新到 最新的版本 ，可以使用使用 upgrade --major-versions 命令：

```shell
 flutter pub upgrade --major-versions
 ```
这个命令也会自动更新 pubspec.yaml 文件中的约束条件。

如果需要自动判断那些过时了的 package 依赖以及获取更新建议，现在你可以使用 outdated 命令。更多相关的信息，请参考 Dart 文档中关于 pub outdated 的说明。

```shell
 flutter pub outdated
 ```