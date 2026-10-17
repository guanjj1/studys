<!--
 * @Author: guanjiajun www.guanjiajun@ewake.com
 * @Date: 2025-09-19 15:13:23
 * @LastEditors: guanjiajun www.guanjiajun@ewake.com
 * @LastEditTime: 2025-09-19 15:29:37
 * @FilePath: \studys\programming\web,client\flutter\flutter版本管理.md
 * @Description: 这是默认设置,请设置`customMade`, 打开koroFileHeader查看配置 进行设置: https://github.com/OBKoro1/koro1FileHeader/wiki/%E9%85%8D%E7%BD%AE
-->
<https://github.com/leoafarias/fvm/releases>
<https://yuluyao.com/flutter/cf-fvm>

### 基本命令
#### 版本管理
```shell
# 列出可用的 Flutter 版本
fvm releases

# 安装指定版本
fvm install 3.16.0
fvm install stable
fvm install beta

# 列出已安装的版本
fvm list

# 删除指定版本
fvm remove 3.16.0
```

#### 项目配置
```shell
# 为当前项目设置 Flutter 版本
fvm use 3.16.0

# 为当前项目设置稳定版
fvm use stable

# 全局设置默认版本
fvm global 3.16.0
```

#### 运行命令
```shell
# 使用项目配置的版本运行 Flutter 命令
fvm flutter --version
fvm flutter pub get
fvm flutter run
fvm flutter build apk

# 使用指定版本运行
fvm flutter --flavor=3.16.0 --version
```
