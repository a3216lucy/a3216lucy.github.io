---
title: 【基礎教學】 TailwindCSS 逐步上手
date: 2022-01-19 17:44:17
categories: 
- 技術筆記
tags:
- 教學
- 技術文件
- TailwindCSS
---
正好想學，在研究之餘，順便寫個教學留個紀錄。

## Tailwind 安裝流程 （使用 npm）

依照 [Tailwind 的官網](https://tailwindcss.com/docs/installation)，安裝有兩種方式：

- 使用 CDN
- 使用 npm

而今天主要是以 npm 的方式來展示。

因為 Tailwind 若是靠 CDN 的方式，會導致很多強大的功能沒有全開，我們這邊就簡單舉例幾個：

- 基本的，就是無法 **新增自訂顏色** 跟 **支援深色模式**。
- 再進階一點，就是無法使用 **`` `@apply` ``** 指令將語法整理起來。
- 最嚴重的，就是 **無法壓縮檔案**，CDN 版大小約 2.79MB。 （完整版約 3.7MB~4MB）

所以如果只是為了 **玩玩、體驗 Tailwind 的特性及帶來的方便性**，完全可以**使用 CDN** 的方式就好，但如果是想要做公司產品／成品上線，這個沒壓縮的大小很有可能讓你被火了XDDD

## 前置作業 - 安裝 node.js

首先，如果你會用 linux 類系統，這個你肯定用過。
不過新手們的話就別跳過，還是都跑一遍吧，以免你有指令沒跑到或東西沒裝到。

```shell
sudo apt update
```

接著，這個很重要，要安裝 curl。 因為等一下就要靠他了。

```shell
sudo apt-get install curl
```

再來，使用到剛剛安裝的 curl，把 node.js 12版的安裝腳本下載下來。請注意，這邊一定要抓12版或更新的！

```shell
curl -sL https://deb.nodesource.com/setup_12.x | sudo -E bash -
```

這下，就可以放心的把 node.js 安裝下去了~

```shell
sudo apt-get install nodejs
```

## 安裝 Tailwind

> 注意：Tailwind CSS 需要 Node.js 版本在 12.13.0 以上

1. 首先，因為 Thailwind 有 node.js 的版本限制，可以先執行以下的指令檢查 node.js 的版本。

```shell
node --version
```

2. 如果檢查完，確定 node 是 12 版以上，那就可以放心地執行下面這行指令了。

```shell
npm init
```

在打入 `NPM init` 後，會被要求輸入幾個欄位

> `package name`: 你這個 Project 要叫什麼名字
> `version`: 你決定這個 Project 現在該是第幾版
> `description`: Project 基本介紹
> `entry point`: 進入點，如果要跑你的 Project 應該要執行哪個檔案
> `author`: 作者（自己）
> `license`: 你這個 Project 是採用什麼授權的
> `test command`: 這個不太重要，待會會說明

基本上結束後，你可以看到這個資料夾底下，新增了一個 `Package.json`

```javascript=
{
  "name": "my-project",
  "version": "1.0.0",
  "description": "this is my project",
  "main": "app.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "author": "Kimi",
  "license": "ISC"
}
```

基本上使用 NPM 來創建的 Project，他會連一些專案的資訊都幫你做管理。

> **Optional Info**：
> 如果你覺得上面要一直輸入很冗，可以使用  `npm init -y` 。
> 他跟  `npm init`  幾乎一樣，只是它會幫你把預設選項全部跳過，產生一個空白的 `package.json` （懶人專用）

3. 接著， 依照官方教學，需安裝 Tailwind CSS、PostCSS，還有 autoprefixer 這三個檔案；因為 Tailwind 不會自動加上瀏覽器前缀詞到產生的 CSS 中，所以瀏覽器無法讀取 CSS，要加上 autoprefixer 這個套件去處理此問題。
   我們使用一行指令就三個安裝好了，輕鬆愉快。

```shell
npm install tailwindcss@latest postcss@latest autoprefixer@latest
```

4. 再來，為了以防萬一，我們執行一次 `` `npm install` `` 來更新和安裝其他依賴套件的狀態。

```shell
npm install
```

那這樣就算安裝完了，只是距離使用，中間還有一些東西需要設定。
如果你還會透過 postCSS 安裝其他插件、或是像官方的推薦使用 autoprefixer的話，可以看一下下方的 **安裝 PostCSS 套件。

### Optional Info

如果在安裝過程中，有遇到以下錯誤，那可能是 **將 Tailwind 與舊版本的 PostCSS 混合使用** 所導致的結果， v2.0 版本的 Tailwind 依賴於 PostCSS 8。

```shell
Error: PostCSS plugin tailwindcss requires PostCSS 8.
```

這時請安裝 [PostCSS 7](https://www.tailwindcss.cn/docs/installation#post-css-7) 版本，才能兼容。

<hr>

#### 安裝 PostCSS 7

```shell
npm uninstall tailwindcss
npm install tailwindcss@npm:@tailwindcss/postcss7-compat
```

<hr>

#### 安裝 PostCSS 套件

```shell
npm install tailwindcss autoprefixer postcss postcss-cli
```

上面這個指令，就是幫你的專案透過 npm 安裝 tailwindcss、postcss、autoprefixer，及 postcss-cli 這四個東西。
那如果有看過官方教學，對前三個東西應該不陌生，只是最後一個也是必須的，算是幫官方安裝過程的補充。

在前面的步驟產生完 Tailwind 設定檔之後，我們要來產生 PostCSS 的設定檔，不然 PostCSS 不會幫我們載入 tailwind 和 autoprefixer。

- **Windows用戶**

```
在專案目錄下新增檔案 -> 檔名為 "postcss.config.js"
```

- **Linux、Ubuntu、MacOS 類用戶**，直接下指令即可（ 在專案目錄下）

```shell
touch postcss.config.js
```

新增完 `` `postcss.config.js` ``，我們要在檔案中加入下面的內容：

```javascript
// postcss.config.js
module.exports = {
  plugins: [
    require('tailwindcss'),
    require('autoprefixer'),
  ]
}
```

或是用官方給的格式也可以~

```js
// postcss.config.js
module.exports = {
  plugins: {
    tailwindcss: {},
    autoprefixer: {},
  }
}
```

最後的最後，只要把前面設定執行的地方，改成

```js
scripts:[
    /* ... 前面是你其它的設定值，略 .... */
    "{ 你的執行名稱 }": "postcss {你 css 檔擺放的路徑}/{要寫樣式的css檔名稱} -o {你 css 檔擺放的路徑}/{頁面要掛載的css檔名稱}"
],
```

剩下的步驟，都跟上面 tailwind 的教學一樣囉~

## 設定 Tailwind

1. 依照上方的流程，安裝完 Tailwind 應該會看到 `package.json` 中有新增後來安裝的 Tailwind CSS、PostCSS 和 autoprefixer。

```javascript
// package.json
{
  "dependencies": {
    "autoprefixer": "^10.2.6",
    "postcss": "^8.3.0",
    "tailwindcss": "^2.1.4"
  }
}
```

2. 在使用 Tailwind CSS 之前，我們得產生 tailwind 必要的設定檔，這也是要達到 tailwind 客製化前最重要的一步。

```shell
npx tailwindcss init
```

執行上方的指令後，在專案目錄下應該會多出一個叫做 `` `tailwind.config.js` `` 的檔案。

```javascript
module.exports = {
  purge: [],
  darkMode: false, // or 'media' or 'class'
  theme: {
    extend: {},
  },
  variants: {
    extend: {},
  },
  plugins: [],
};
```

這麼一來，就可以自訂專屬於你的 Tailwind 設定了！

## 新增 Tailwind 到 CSS 中

現在需要在你存放 CSS 樣式檔的資料夾裡，建立兩個 CSS 檔：

- 一個是你頁面要掛載的 `style.css` （名稱可自訂）
- 一個是用來寫 CSS 樣式的 `tailwind.css` （名稱也可自訂）

這樣講可能有點抽象，拿我的專案目錄來舉例好了。

![](https://i.imgur.com/SLrk4vo.png)

在我的專案下，擺放 CSS 檔的路徑是 `/public/css` ，那就在這個資料夾底下產生兩個 CSS檔，名稱自訂，我這邊是把要掛載的叫做 `app.css`，要寫樣式內容的叫做 `tailwind.css`。

建立完之後，我們在要用來寫樣式的 CSS 檔 （我這邊叫做 `tailwind.css` ）裡面貼上下面的 CSS 語法。

```css
/* 貼在用來寫樣式的css檔，範例這邊是 tailwind.css */

@tailwind base; 
@tailwind components; 
@tailwind utilities;
```

> 我們要掛載的 CSS 檔 (這邊叫做 `app.css`) 保持空白就好，以後所有要補充的樣式都寫在另一個 CSS 檔（`tailwind.css`）裡。
> 因為如果接下來 **進行編譯**， 要掛載的 CSS 檔 (`app.css`) 的 **內容會被覆蓋掉**。

## 編譯

終於進入最後一步了。
開啟剛剛 `` `npm init` `` 執行後產生的 `package.json`，在 scripts 裡面加一行：

```js
scripts:[
    /* ... 前面是你其它的設定值，略 .... */
    "{ 你的執行名稱 }": "tailwind build {你 css 檔擺放的路徑}/{要寫樣式的css檔名稱} -o {你 css 檔擺放的路徑}/{頁面要掛載的css檔名稱}"
],
```

如果把上述的名稱都取代完之後，在我這邊的範例，會長的像下面這樣：

```js
scripts:[
    /* ... 前面是你其它的設定值，略 .... */
    "build-css": "tailwind build public/css/tailwind.css -o public/css/app.css"
],
```

最後，我們只要執行 ``npm run {你的執行名稱}``，那他就會開始進行編譯了！

```shell
npm run build-css
```

編譯完成後，應該會在命令列 (終端機) 看到下面的訊息：

![](https://i.imgur.com/vYO0NIn.png)

這樣就算編譯完成了，現在只要你在頁面掛上要掛載的 CSS 檔，並用上 Tailwind 語法，就會有效果囉！

## 使用 Tailwind CSS

在你網頁中插入的 CSS 檔路徑要改成剛剛上面說過的，用來掛載在頁面上的 CSS 檔。

```html
<!-- 你專案中要使用 Tailwind CSS 的頁面 -->
<html>
    <head>
        <!-- 前略 -->
        <link rel="stylesheet" href="./{你CSS檔的路徑}/{要掛載在頁面的CSS檔名稱}">
    </head>
    <body>
        <!-- 略 -->
    </body>
</html>
```

底下這邊是我依照我專案所改寫後的。

```html
<!-- 這範例是我專案中的 index.html -->
<html>
    <head>
        <!-- 前略 -->
        <link rel="stylesheet" href="./css/app.css">
    </head>
    <body>
        <!-- 馬上用 Tailwind 寫一個熱騰騰的黃綠藍三色漸層 Hello World 方塊 -->
        <div class="m-3 p-10 bg-gradient-to-r from-yellow-400 via-green-400 to-blue-400 text-white">
          Hello World ! I'm using Tailwind CSS Now ~
        </div>
    </body>
</html>
```

這樣就把 Tailwind CSS 安裝完了，這樣你就可以盡情地使用 Tailwind 寫出一堆美美的介面了啦！

## Optional：幫 CSS 瘦身

Tailwind 因為是 Utility 的關係，會把整包檔案匯入，如果整包檔案要選染於網頁，效能就會低落。

Tailwind 官方文件有提供可以瘦身的方法：

1. 選到 `tailwind.config.js` 檔案，將我可能會使用到的內容寫在  `purge` 屬性內

```javascript
//tailwind.config.js

module.exports = {
  purge: {
    enabled: true,
    content: ["./**/*.html"],
  },
  darkMode: false, // or 'media' or 'class'
  theme: {
    extend: {},
  },
  variants: {
    extend: {},
  },
  plugins: [],
};

```

2. 這樣程式執行時，就會去監聽我哪支檔案有使用到，只會編譯使用的檔案，真的很方便！
