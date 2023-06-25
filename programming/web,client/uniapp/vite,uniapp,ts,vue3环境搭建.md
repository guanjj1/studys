<!--
 * @Author: guanjiajun www.guanjiajun@ewake.com
 * @Date: 2023-05-12 14:54:28
 * @LastEditors: guanjiajun www.guanjiajun@ewake.com
 * @LastEditTime: 2023-05-19 08:44:14
 * @FilePath: \studys\programming\web,client\uniapp\vite,uniapp,ts,vue3环境搭建.md
 * @Description: 这是默认设置,请设置`customMade`, 打开koroFileHeader查看配置 进行设置: https://github.com/OBKoro1/koro1FileHeader/wiki/%E9%85%8D%E7%BD%AE
-->
<https://megasu.gitee.io/uni-app-shop-note/rabbit-shop/#%E5%BC%95%E5%85%A5-uni-ui-%E7%BB%84%E4%BB%B6%E5%BA%93>\
<https://blog.csdn.net/G_ing/article/details/129100923>\
<https://blog.csdn.net/qq_25753445/article/details/127450588>\
<https://www.dandelioncloud.cn/article/details/1568123822723526658>\
<https://zhuanlan.zhihu.com/p/528057312?utm_id=0>\
<https://blog.csdn.net/cookcyq__/article/details/125457031>
### 项目创建
```shell
npx degit dcloudio/uni-preset-vue#vite-ts my-vue3-project

#pnpm missing 依次pnpm add -D或pnpm install
pnpm install

#npm强制执行会失败 安装时间过长
npm install

#安装时间过长
yarn

pnpm add vant
pnpm add 

```
### 引入 uni-ui 组件库
安装 uni-ui 组件库
```shell
pnpm i @dcloudio/uni-ui
```
配置自动导入组件
```json
// pages.json
{
  // 组件自动导入
  "easycom": {
    "autoscan": true,
    "custom": {
      // uni-ui 规则如下配置  
      "^uni-(.*)": "@dcloudio/uni-ui/lib/uni-$1/uni-$1.vue" 
    }
  },
  "pages": [
    // …省略
  ]
}
```
安装类型声明文件(代码提示)uni-helper 插件
```shell
pnpm i -D @uni-helper/uni-ui-types
pnpm i -D @uni-helper/uni-app-types
```
配置类型声明文件
```json
// tsconfig.json
{
  "compilerOptions": {
    "types": [
      "@dcloudio/types",
      "@uni-helper/uni-app-types", 
      "@uni-helper/uni-ui-types" 
    ]
  }
}
```
### 统一代码风格
安装 eslint + prettier
```shell
pnpm i -D eslint prettier eslint-plugin-vue @vue/eslint-config-prettier @vue/eslint-config-typescript @rushstack/eslint-patch @vue/tsconfig
```
新建 .eslintrc.cjs 文件，添加以下 eslint 配置
```json
/* eslint-env node */
require('@rushstack/eslint-patch/modern-module-resolution')

module.exports = {
  root: true,
  extends: [
    'plugin:vue/vue3-essential',
    'eslint:recommended',
    '@vue/eslint-config-typescript',
    '@vue/eslint-config-prettier',
  ],
  // 小程序全局变量
  globals: {
    uni: true,
    wx: true,
    WechatMiniprogram: true,
    getCurrentPages: true,
    UniApp: true,
    UniHelper: true,
  },
  parserOptions: {
    ecmaVersion: 'latest',
  },
  rules: {
    'prettier/prettier': [
      'warn',
      {
        singleQuote: true,
        semi: false,
        printWidth: 100,
        trailingComma: 'all',
        endOfLine: 'auto',
      },
    ],
    'vue/multi-word-component-names': ['off'],
    'vue/no-setup-props-destructure': ['off'],
    'vue/no-deprecated-html-element-is': ['off'],
    '@typescript-eslint/no-unused-vars': ['off'],
  },
}
```
在根目录新建.eslintignore 文件，将不需要检测文件路径写入其中即可，如下(可不写)
```
# 忽略dist目录下文件的语法检查

/dist
```
配置 package.json
```json
{
    "scripts": {
        // eslint . 为指定lint当前项目中的文件
        // --ext 为指定lint哪些后缀的文件
        // --fix 开启自动修复
        "lint": "eslint . --ext .vue,.js,.ts,.jsx,.tsx --fix"
    }
}
```
配置.prettierrc.json
```json
{
  "$schema": "https://json.schemastore.org/prettierrc",
  "semi": false,
  "tabWidth": 2,
  "singleQuote": true,
  "printWidth": 100,
  "trailingComma": "none"
}
or
{
  "arrowParens": "always",
  "bracketSameLine": false,
  "bracketSpacing": true,
  "embeddedLanguageFormatting": "auto",
  "htmlWhitespaceSensitivity": "css",
  "insertPragma": false,
  "jsxSingleQuote": false,
  "printWidth": 80,
  "proseWrap": "preserve",
  "quoteProps": "as-needed",
  "requirePragma": false,
  "semi": true,
  "singleQuote": true,
  "tabWidth": 2,
  "trailingComma": "es5",
  "useTabs": false,
  "vueIndentScriptAndStyle": false,
  "endOfLine": "auto"
}
or
{
  // 超过最大值换行
  printWidth: 100,
  // 缩进字节数
  tabWidth: 2,
  // 缩进不使用tab，使用空格
  useTabs: false,
  // 句尾添加分号
  semi: true,
  vueIndentScriptAndStyle: true,
  // 使用单引号代替双引号
  singleQuote: true,
  quoteProps: 'as-needed',
  // 在对象，数组括号与文字之间加空格 "{ foo: bar }"
  bracketSpacing: true,
  // 在对象或数组最后一个元素后面是否加逗号（在ES5中加尾逗号)
  trailingComma: 'es5',
  // 在jsx中把'>' 是否单独放一行
  jsxBracketSameLine: false,
  // 在jsx中使用单引号代替双引号
  jsxSingleQuote: false,
  // (x) => {} 箭头函数参数只有一个时是否要有小括号。avoid：省略括号
  arrowParens: 'always',
  insertPragma: false,
  requirePragma: false,
  // 默认值。因为使用了一些折行敏感型的渲染器（如GitHub comment）而按照markdown文本样式进行折行
  proseWrap: 'never',
  htmlWhitespaceSensitivity: 'strict',
  // 结尾是 \n \r \n\r auto
  endOfLine: 'lf',
  rangeStart: 0,
}
```
配置 package.json
```json
{
  "script": {
    // ... 省略 ...
    "lint": "eslint . --ext .vue,.js,.ts --fix --ignore-path .gitignore",
    "format": "prettier --write src/",
  }
}
```
运行
```shell
pnpm lint
```
### Git 工作流规范(前提项目是git项目)
安装并初始化 husky
```shell
pnpm dlx husky-init
```
安装 lint-staged\
有没有办法能让这些工具只检查&修复我们修改过的文件就好呢？ lint-staged 就可以做到。
```shell
pnpm i lint-staged -D
```
配置 package.json
```json
{
  "script": {
    // ... 省略 ...
  },
  "lint-staged": {
    "*.{vue,ts,js}": ["eslint --fix"]
  }
}
```
修改 .husky/pre-commit 文件
```diff
-npm test   
+pnpm lint-staged  
or
+pnpm lint && pnpm format
+yarn run lint && yarn run format  
```
```shell
git commit -m'test' -n
# 其中 -n 表示忽略 pre-commit 钩子，直接提交上去。
```