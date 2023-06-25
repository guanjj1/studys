<!--
 * @Author: guanjiajun www.guanjiajun@ewake.com
 * @Date: 2023-05-23 11:18:33
 * @LastEditors: guanjiajun www.guanjiajun@ewake.com
 * @LastEditTime: 2023-05-23 11:18:43
 * @FilePath: \studys\programming\server\go\sql2struct.md
 * @Description: 这是默认设置,请设置`customMade`, 打开koroFileHeader查看配置 进行设置: https://github.com/OBKoro1/koro1FileHeader/wiki/%E9%85%8D%E7%BD%AE
-->
# sql2struct

SQL2Struct 是一个可以将 SQL 语句转换成 Go struct 的工具。

## 在线使用

点击 [sql2struct 在线版](https://dou.tools/sql2struct/) 进行使用

## 作为 Chrome 插件使用

1. 使用源码安装 sql2struct

```shell
git clone https://github.com/idoubi/sql2struct.git
cd sql2struct
pnpm install
```

2. 构建 Chrome 插件代码

```shell
pnpm build:chrome
```

3. 本地安装 Chrome 插件

Chrome 浏览器打开 [chrome://extensions/](chrome://extensions/)

点击”加载已解压的扩展程序“，选择 sql2struct 根目录下的 `dist/chrome` 文件夹

## 本地部署使用

1. 使用源码安装 sql2struct

```shell
git clone https://github.com/idoubi/sql2struct.git
cd sql2struct
pnpm install
```

2. 预览

```shell
pnpm dev
```

3. 构建

```shell
pnpm build

# or

pnpm build:web
```

## 如何使用

1. 打开数据库客户端，执行 `show create table xxx\G;` 导出 SQL 建表数据

![20220626221324](https://blogcdn.idoustudio.com/blog/20220626221324.png)

2. 粘贴 SQL 建表语句至 sql2struct 左侧的输入框

![20220626222145](https://blogcdn.idoustudio.com/blog/20220626222145.png)

粘贴完 SQL 建表语句后，右侧输入框自动生成转换后的 Go struct 代码，复制粘贴到 Go 项目中使用即可

3. 转换配置项

默认情况下，转换成的 Go struct 会带有一个 json 标签。你也可以勾选其他标签。目前支持 json、xml、gorm、xorm、mapstruct 等标签。

![20220626222618](https://blogcdn.idoustudio.com/blog/20220626222618.png)

点击 Options 按钮，进入设置页面

在 Special Identifiers 配置项，你可以自定义需要全部转换成大写的关键词。

![20220626222854](https://blogcdn.idoustudio.com/blog/20220626222854.png)

在 Fiedl Maps 配置项，你可以自定义 SQL 字段与 Go struct 字段的映射关系。

![20220626222912](https://blogcdn.idoustudio.com/blog/20220626222912.png)