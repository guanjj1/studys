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
#方式三，my-vue-app项目名称，使用vue-ts模板 可预先配置vue的一些常用依赖
pnpm create vue my-vue-app --template vue-ts

```
