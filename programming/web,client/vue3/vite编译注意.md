<!--
 * @Author: guanjiajun www.guanjiajun@ewake.com
 * @Date: 2023-05-15 10:04:17
 * @LastEditors: guanjiajun www.guanjiajun@ewake.com
 * @LastEditTime: 2023-05-15 10:04:43
 * @FilePath: \studys\programming\web,client\vue3\vite编译注意.md
 * @Description: 这是默认设置,请设置`customMade`, 打开koroFileHeader查看配置 进行设置: https://github.com/OBKoro1/koro1FileHeader/wiki/%E9%85%8D%E7%BD%AE
-->
需要在vite.config.js中配置base属性(相对路径默认用./)
```typescript
export default defineConfig({
    base: "./", //等同于  assetsPublicPath :'./'
    plugins: [
        vue(),
        styleImport({
            resolves: [VantResolve()],
        }),
    ],
    resolve: {
        alias: {
            "@": path.resolve(__dirname, "src"),
            "view": path.resolve(__dirname, "src/view"),
            "components": path.resolve(__dirname, "src/components"),
            "assets": path.resolve(__dirname, "src/assets"),
            "store": path.resolve(__dirname, "src/store"),
            "mixins": path.resolve(__dirname, "src/mixins"),
        },
    },
    css: {
        preprocessorOptions: {
            scss: {
                additionalData: '@import "./src/assets/styles/common.scss";'
            }
        }
    },
    server: {
        host: "0.0.0.0",
    },
});
```
```shell
pnpm build
```


