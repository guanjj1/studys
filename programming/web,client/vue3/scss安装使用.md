<!--
 * @Author: guanjiajun www.guanjiajun@ewake.com
 * @Date: 2023-03-14 15:22:38
 * @LastEditors: guanjiajun www.guanjiajun@ewake.com
 * @LastEditTime: 2023-03-14 15:23:10
 * @FilePath: \studys\programming\web,client\vue3\scss安装使用.md
 * @Description: 这是默认设置,请设置`customMade`, 打开koroFileHeader查看配置 进行设置: https://github.com/OBKoro1/koro1FileHeader/wiki/%E9%85%8D%E7%BD%AE
-->
```bash
pnpm uninstall node-sass    // 卸载
pnpm uninstall --save sass-loader // 卸载

pnpm install sass --save
pnpm install node-sass --save
pnpm install sass-loader --save
//sass-loader的版本过高导致的编译错误 npm install sass-loader@7.3.1 --save （推荐使用）
pnpm install style-loader --save
或者 pnpm install node-sass sass-loader style-loader --save