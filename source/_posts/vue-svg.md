---
title: 在 Vue 聰明使用 SVG-Icon
date: 2020/5/18 20:46:25
tags: [vus.js,svg,webpack]
---
一般來說我們在Vue的專案裡使用SVG，會有兩種比較簡單的方式：
## 第一種：Using SVG as an ＜img＞
利用 `<img>` 標籤來引入，此時 SVG 被視為一個圖檔載入，最大的缺點就是無法利用 CSS 來改變 SVG 的樣式。
```html
<img src="icon.svg" />
```
很不幸的，如果你的 icon 會有改變顏色的需求，你就需要兩張不同顏色的SVG，兩個 `<img>` 標籤，然後用 `display: none` 來控制，就是那麼厚工。
</br>
## 第二種：Inline SVG
直接將 `<svg>` 標籤放進 Html 結構中，這種方法雖然解決了改變顏色的問題，但卻讓程式碼看起來非常雜亂。
```html
<svg xmlns="http://www.w3.org/2000/svg" width="16" height="16" viewBox="0 0 16 16">
  <g fill-rule="evenodd" stroke="#EA475B" stroke-linecap="round">
    <path d="M8 3.5v9M4.5 9.5l3.5 4 3.5-4"/>
  </g>
</svg>
```
---
## SVG Sprites - 更清爽的使用 SVG
為了更有效的使用 SVG，就來試試用 `svg-sprite-loader` 製作一套 SVG Sprites 吧！

![](https://ithelp.ithome.com.tw/upload/images/20200309/201254314jc3SUNivv.png)

先在 src 下新增 `assets` 資料夾，底下再建立 `icon` 資料夾，之後的 `.svg` 檔都會放在這裡。
```
$ npm install svg-sprite-loader -D
$ yarn add svg-sprite-loader -D
```
安裝今天的主角 `svg-sprite-loader` ，在 [官方文件](https://github.com/JetBrains/svg-sprite-loader) 中有說明如何配置 webpack，但剛好 vue-cli 支援 `webpack-chain` ，我們就用 `webpack-chain` 來設定吧！


* vue.config.js

```javascript
module.exports = {
  chainWebpack: config => {
  
    // 先刪除預設的svg配置
    config.module.rules.delete("svg")
    
    // 新增 svg-sprite-loader 設定
    config.module
      .rule("svg-sprite-loader") 
      .test(/\.svg$/)
      .include
      .add(resolve("src/assets/icon")）
      .end()
      .use("svg-sprite-loader")
      .loader("svg-sprite-loader")
      .options({ symbolId: "[name]" })
      
    // 修改 images-loader 配置
    config.module
      .rule("images")
      .exclude.add(resolve("src/assets/icon"))
  }
}
```

比較重要的是，記得把 SVG 的路徑 `src/assets/icon` 給 `add` 進來，並且使用 `options` 的 `symbolId` 屬性來決定 SVG 的 `symbolId` 該以什麼方式命名，這次用的是 `"[name]"`，以檔名來命名。

另外我把原本的 images-loader 排除了 `icon` 資料夾的路徑，這樣只要放在 `src/assets/icon` 的 SVG 就不能用 `<img>` 引入了。

大功告成，這樣之後就可以在 vue 元件中引入 `.svg` 檔。

```javascript
import "@/src/assets/icon/target.svg";
```

然後在 `template` 裡使用下面這樣簡便的寫法，就可以清爽的使用 svg-icon 了，而且還可以隨心所欲的改變顏色！

```html
<svg><use xlink:href="#target" /></svg>
<!-- #target 改成svg的檔名就好囉，記得加井字號 -->
```

---
## 每個元件都要引入 SVG 好麻煩
雖然已經解決了改變icon顏色以及程式碼雜亂的問題，但每個icon都要引入一次似乎也挺麻煩的，所以就一次性的匯入 SVG 吧！
* main.js

```javascript
const requireAll = requireContext => requireContext.keys().map(requireContext)
const req = require.context("@/src/assets/icon", true, /\.svg$/)
requireAll(req)
```

在 `main.js` 加入這段，讓 SVG 全域性的引入，元件中就不用特地 `import` 了。
另外可以全域註冊一個新的元件 `SvgIcon`：

```javascript
import SvgIcon from "@/components/common/SvgIcon"
Vue.component("icon", SvgIcon)
```

</br>

```vue
<template>
  <svg :class="svgClass" aria-hidden="true">
    <use :xlink:href="`#${iconName}`" />
  </svg>
</template>

<script>
export default {
  name: "SvgIcon",
  props: {
    iconName: {
      type: String,
      required: true
    }
  }
};
</script>

<style lang="scss" scoped>
.svg-icon {
  width: 16px;
  height: 16px;
  vertical-align: -0.15em;
  fill: currentColor !important;
  overflow: hidden;
}
</style>
```

這樣就可以直接用元件的方式使用 SVG 囉！

```html
<icon iconName="target" />
<!-- 就像是在用 FontAwesome 一樣舒服 -->
```

---
### 補充
icon 的顏色是吃父層的 css:color ，如果發現 icon 顏色改不了，記得把 SVG 檔裡的 fill 或 stroke 改成 currentColor ，或是請設計師幫你設定一下輸出的檔案配置喔！
