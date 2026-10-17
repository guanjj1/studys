<https://blog.csdn.net/weixin_55629186/article/details/131365012>\
<https://blog.csdn.net/weixin_44814196/article/details/135300290>\

### 本地文件跳转
```md
// 跳到当前目录下的Readme1.md文件
[Readme1](Readme1.md)

// 跳到当前目录下的Readme2文件夹
[Readme2](Readme2)

// 相对路径跳到上级目录的某个文件
[Readme3](../Docs/Readme3.md)

// 绝对路径跳转到C盘下的某个md文件
[Readme4](C:/Develop/Docs/Readme4.md)

```
### 文内跳转
#### html语法
```md
<a href="#锚点1">点击这里跳转到锚点1</a>
<a href="#锚点名2">点击这里跳转到锚点2</a>

<a id="锚点1">标题1，作为锚点1</a>

```
#### markdowwn语法
```md

[点击这里跳转到锚点2](#锚点名2)

<a id="锚点名2">标题2，作为锚点2</a>

```
#### 标题跳转
```md
- [consumer](#consumer)
- [provider类型](#provider类型)
  - [provider](#provider)
    - [官方](#官方)
    - [小结](#小结)
      - [小结1](#小结1)
      - [小结2](#小结2)
  - [(Async)NotifierProvider](#asyncnotifierprovider)
  - [StateNotifierProvider](#statenotifierprovider)
  - [FutureProvider](#futureprovider)
  - [StreamProvider](#streamprovider)
  - [StateProvider](#stateprovider)
  - [ChangeNotifierProvider](#changenotifierprovider)
```