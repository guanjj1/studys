<!--
 * @Author: guanjiajun www.guanjiajun@ewake.com
 * @Date: 2023-03-02 14:11:51
 * @LastEditors: guanjiajun www.guanjiajun@ewake.com
 * @LastEditTime: 2023-03-07 19:13:24
 * @FilePath: \studys\前端\vue3\vue,vite,ts项目搭建.md
 * @Description: 这是默认设置,请设置`customMade`, 打开koroFileHeader查看配置 进行设置: https://github.com/OBKoro1/koro1FileHeader/wiki/%E9%85%8D%E7%BD%AE
-->
###1.安装node.js
1.官网安装
2.安装校验
```bash
node -v

```
###2.vscode构建项目
1.插件安装
- volar
- TypeScript Vue Plugin
```typescript
/* 在.d.ts文件中添加以下代码，同TypeScript Vue Plugin作用 
使ts识别.vue文件*/
declare module '*.vue' {
    import type { DefineComponent } from 'vue'
    const component: DefineComponent<{}, {}, any>
    export default component
}
```
2.使用vite构建项目
```bash
npm i -g pnpm
#方式一
pnpm create vite
#方式二，my-vue-app项目名称，使用vue-ts模板
pnpm create vite my-vue-app --template vue-ts
#方式三，my-vue-app项目名称，使用vue-ts模板 可预先配置vue的一些常用依赖.
pnpm create vue my-vue-app --template vue-ts

```
