<!--
 * @Author: guanjiajun www.guanjiajun@ewake.com
 * @Date: 2024-04-04 14:02:00
 * @LastEditors: guanjiajun www.guanjiajun@ewake.com
 * @LastEditTime: 2024-04-04 18:01:34
 * @FilePath: \studys\programming\web,client\flutter\project from 0 to start\本地化存储\sharedperferences.md
 * @Description: 这是默认设置,请设置`customMade`, 打开koroFileHeader查看配置 进行设置: https://github.com/OBKoro1/koro1FileHeader/wiki/%E9%85%8D%E7%BD%AE
-->
<https://pub.dev/packages/shared_preferences>\
```shell
flutter pub add shared_preferences
```
### Write data
```dart
// Obtain shared preferences.
final SharedPreferences prefs = await SharedPreferences.getInstance();

// Save an integer value to 'counter' key.
await prefs.setInt('counter', 10);
// Save an boolean value to 'repeat' key.
await prefs.setBool('repeat', true);
// Save an double value to 'decimal' key.
await prefs.setDouble('decimal', 1.5);
// Save an String value to 'action' key.
await prefs.setString('action', 'Start');
// Save an list of strings to 'items' key.
await prefs.setStringList('items', <String>['Earth', 'Moon', 'Sun']);
```
### Read data
```dart
// Try reading data from the 'counter' key. If it doesn't exist, returns null.
final int? counter = prefs.getInt('counter');
// Try reading data from the 'repeat' key. If it doesn't exist, returns null.
final bool? repeat = prefs.getBool('repeat');
// Try reading data from the 'decimal' key. If it doesn't exist, returns null.
final double? decimal = prefs.getDouble('decimal');
// Try reading data from the 'action' key. If it doesn't exist, returns null.
final String? action = prefs.getString('action');
// Try reading data from the 'items' key. If it doesn't exist, returns null.
final List<String>? items = prefs.getStringList('items');
```
### Remove an entry
// Remove data for the 'counter' key.
```dart
await prefs.remove('counter');
```