<!--
 * @Author: guanjiajun www.guanjiajun@ewake.com
 * @Date: 2024-04-05 17:27:57
 * @LastEditors: guanjiajun www.guanjiajun@ewake.com
 * @LastEditTime: 2024-04-08 18:24:50
 * @FilePath: \studys\programming\web,client\flutter\project from 0 to start\request\model转换.md
 * @Description: 这是默认设置,请设置`customMade`, 打开koroFileHeader查看配置 进行设置: https://github.com/OBKoro1/koro1FileHeader/wiki/%E9%85%8D%E7%BD%AE
-->
# freezed
<https://pub-web.flutter-io.cn/packages/freezed>\
## 搭建
```shell
flutter pub add freezed_annotation
flutter pub add dev:build_runner
flutter pub add dev:freezed
# if using freezed to generate fromJson/toJson, also add:
flutter pub add json_annotation
flutter pub add dev:json_serializable
```
### Disabling invalid_annotation_target warning and warning in generates files. 
If you plan on using Freezed in combination with json_serializable, recent versions of json_serializable and meta may require you to disable the invalid_annotation_target warning.

To do that, you can add the following to the analysis_options.yaml file at the root of your project:
```yaml
analyzer:
  errors:
    invalid_annotation_target: ignore
```
### Run the generator 
To run the code generator, execute the following command:
```shell
dart run build_runner build
```
Note that like most code-generators, Freezed will need you to both import the annotation (freezed_annotation) and use the part keyword on the top of your files.

As such, a file that wants to use Freezed will start with:
```dart
import 'package:freezed_annotation/freezed_annotation.dart';

part 'my_file.freezed.dart';
```
CONSIDER also importing package:flutter/foundation.dart.
The reason being, importing foundation.dart also imports classes to make an object nicely readable in Flutter's devtool.
If you import foundation.dart, Freezed will automatically do it for you.
## Freezed extension for VSCode 
The Freezed extension might help you work faster with freezed. For example :

* Use Ctrl+Shift+B (Cmd+Shift+B on Mac) to quickly build using build_runner.
* Quickly generate a Freezed class by using Ctrl+Shift+P (Cmd+Shift+P on Mac)> Generate Freezed class.