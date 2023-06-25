<!--
 * @Author: guanjiajun www.guanjiajun@ewake.com
 * @Date: 2023-06-09 13:01:08
 * @LastEditors: guanjiajun www.guanjiajun@ewake.com
 * @LastEditTime: 2023-06-09 16:38:44
 * @FilePath: \studys\programming\web,client\typescript\ts基础知识.md
 * @Description: 这是默认设置,请设置`customMade`, 打开koroFileHeader查看配置 进行设置: https://github.com/OBKoro1/koro1FileHeader/wiki/%E9%85%8D%E7%BD%AE
-->
### ts编译成js
<https://www.typescriptlang.org/docs/handbook/tsconfig-json.html>\
```shell
# ts安装
npm install typescript -g
```
tsconfig.json生成,前提ts环境完备
```shell
tsc --init
```
编译全部ts文件
```shell
tsc
```
tsc 指定ts文件，默认忽略tsconfig.json（可能导致编译失败），tsconfig.json添加files
```json
{
  "compilerOptions": {
    "module": "commonjs",
    "noImplicitAny": true,
    "removeComments": true,
    "preserveConstEnums": true,
    "sourceMap": true
  },
  "files": [
    "core.ts",
    "sys.ts",
    "types.ts",
    "scanner.ts",
    "parser.ts",
    "utilities.ts",
    "binder.ts",
    "checker.ts",
    "emitter.ts",
    "program.ts",
    "commandLineParser.ts",
    "tsc.ts",
    "diagnosticInformationMap.generated.ts"
  ]
}
```
单命令运行
```shell
# 安装ts-node
npm install ts-node -g
```
tsc命令报错（tsc : 无法加载文件 C:\Users\Administrator\AppData\Roaming\npm\tsc.ps1，因为在此系统上禁止运行脚本）,以管理员身份打开 Windows PowerShell
```shell
set-ExecutionPolicy RemoteSigned
```