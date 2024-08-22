---
title: 【基礎教學】利用 Hexo 與 GitPage 建置個人 Blog
date: 2021-09-09 00:52:41
author: "Kimi"
img: https://miro.medium.com/max/1400/0*wtxNrIzieLhNGPmP
categories: 
- 技術筆記
tags: 
- 教學
- Hexo
---
## 前言

本文主要介紹如何使用 Hexo 來製作個人 Blog，並利用 GitPage 進行自動更新部署，讓沒接觸過新手小白也能輕鬆上手。

## 環境建置

在開始之前，須先：

* 安裝 [Node.js](https://nodejs.org/en/)
* 安裝 [Git](https://git-scm.com/)
* 註冊 [GitHub](https://github.com/) 帳號
* 瞭解 cmd 或 bash 指令（下文中所有以 ‘$’ 為開頭的，皆為 bash 指令）

## 建置 Hexo

### 安裝 Hexo

使用 npm 來安裝 hexo （須先安裝 Node.js）

```
$ npm install -g hexo-cli
```

安裝成功後，輸入以下指令可查看版本。

```
$ hexo version
```

![](https://miro.medium.com/max/1400/1*OUtcy0m_9ASKBBHJS__9jg.png)

### 安裝 Hexo Git

```
$ npm install hexo-deployer-git --save
```

接下來即可初始化我們的部落格了。

執行下方指令，Hexo 就會在指定資料夾 <folder> 新增所需的檔案。

```
hexo init <folder>   # 初始化 blog 資料夾（自己命名）
cd <folder>          # 移動到剛創建的 blog 資料夾裡
npm install          # 安裝相關套件
```

安裝完成之後，docs 的目錄結構如下：

* `_config.yml` 網站 全域配置 檔案，可以在此設置大部分的設定，其設定是用 yaml 格式編寫。
* `package.json` Node.js 的套件版本控制。
* `scaffolds` Hexo 釋出文章的時候使用，建立新文章時，會根據 scaffold 來建立檔案。
* `source` MarkDown 和各種原始檔，放置內容的地方。
* `themes` Hexo 的主題資料夾，會根據主題來產生靜態檔案。

### 本地端啟動

```
$ hexo generate
```

亦可用 `hexo g`，產生靜態檔案，會在目錄下產生 public 資料夾。

```
$ hexo server
```

亦可用 `hexo s`，啟動伺服器，預設是 `http://localhost:4000/`

```
$ hexo deploy
```

亦可用 `hexo d`，部署網站。（ 如：github、heroku 等 ）

完成上述步驟後，在瀏覽器中即可看到本地端啟動的個人網頁了。

## 部署到 GitHub

### 新建 GitHub Repository

接著，我們可以在 Github 的主分支上來建立我們的文件了。有兩種方式可以達成：

* 在一個已存在的專案中建立文件
* 建立一個新的專案 [Create a new repository](https://github.com/new)

簡單起見，假設我們新建立了一個名為 `yourname.github.io` 的倉庫（repository），當然你也可以用一個已經存在的專案繼續下面的操作。

![](https://miro.medium.com/max/1400/1*-mmXEsYov4s11mpPSvu0yA.png)

<p style ="text-align:center"><i>請將上面的 yourname 替換成你 GitHub 的帳號名稱</i></p>

![](https://miro.medium.com/max/1400/1*cOqn6zvl5jWSGLqinnhuAw.png)

<p style ="text-align:center"><i>創建完新 repo 的畫面</i></p>

再來，到已創立的指定資料夾 <folder>中找一個 `_config.yml` 的檔案。

```
# Site
title: /標題(會顯示在網頁標題與頁首)/
subtitle: /子標題(顯示在頁首)/
description: /內容描述(給搜尋引擎看的)/
author: /作者(顯示在頁尾)/
language: /網站預設語言(台灣:zh-tw)/
timezone: /時區(預設使用你電腦的時區)/

# URL
url: /網站的網域位址/
root: /網站根目錄/
permalink: /文章目錄(預設使用 YYYY\MM\DD\文章名稱)/

# Extensions
theme: /網站的佈景主題/# Deployment
deploy:
 type: /發佈型態/ 
 repository: /部署位置/ 
 branch: /分支/
 message: /部署訊息/
```

開啟之後，拉到檔案最底部，可以看到

```
deploy:
 type:
```

將以下資訊對應輸入

```
deploy:
 type: git
 repository: http://github.com/yourname/yourname.github.io.git
 branch: master
```

> <b>注意：</b>
> 設定中的 : 後一定要有一個空格
> 將上面的 yourname 改成你的 GitHub 帳號名稱

### 連線上傳 GitHub

> 過程中，可能會需要輸入GitHub的帳號、密碼做授權。

使用下方的命令在本地 clone 專案：

```
$ git clone http://github.com/yourname/yourname.github.io.git
```

將 `USERNAME` 替換為你的使用者名稱，`REPOSITORY` 替換為你的專案名稱。

```
$ git add .
```

為 commit 留註解。

```
$ git commit -m “first commit”
```

手動新增 GitHub 遠端 branch 名稱、連結

```
$ git remote origin  http://github.com/yourname/yourname.github.io.git
```

推到你的 GitHub master。

```
$ git push origin master
# 只有第一次 push 需要全打，第二次以後執行 git push 即可
```

這樣就把本地的檔案上傳到 GitHub 囉！

### 設定 GitPage

打開 GitHub 專案的 Setting，下面有個 GitHub Page 的地方，將其勾選設定，接著就可以在你個人的 https://yourname.github.io/ 看到畫面了。

![](https://miro.medium.com/max/1400/1*fbLuA1K6Vik-ezAiUbAuJA.jpeg)

## 設定專屬個人 Domain

有了個人的 GitHub Page 後，若想把網址換成專屬網域名稱，提供以下兩個網站可供參考：

1. Gandi.net
2. Godaddy

進去網站後，挑選並確定了自己想購買的網址後，在購物車結帳頁面可以選擇自己想要購買的年限，至少要買一年，若一年後還要再使用可以自行續約。

以 Gandi.net 為例，購買後，到 域名－區域檔紀錄－新增

![](https://miro.medium.com/max/1400/1*Yjbd4zMuUMw2n56nJysL-Q.png)

<p style ="text-align:center"><i>新增紅框中的四筆 DNS</i></p>

新增 DNS 紀錄 （可參考 [GitHub Page 官方說明文件](https://docs.github.com/en/github/working-with-github-pages/managing-a-custom-domain-for-your-github-pages-site)

```
類型： A
TTL： 1800
IPv4： 
185.199.108.153
185.199.109.153
185.199.110.153
185.199.111.153
```

再來，到 GitHub 的 Setting，Custom Domain 欄位輸入購買的網址，按儲存就設定完成了。

> 小建議，下方的 HTTPS 也一同勾選。

![](https://miro.medium.com/max/1400/1*895UBYgeMPzh1OjOyQlgRA.png)

稍待一陣子，就可以用新購買的網址進入個人網站囉。

## 更換主題

Hexo 預設的主題是 [landscape](https://github.com/hexojs/hexo-theme-landscape)，而我們可以到 [Hexo主題庫](https://hexo.io/themes/) 來挑選我們欲安裝的主題，以下以 [Ocean](https://github.com/zhwangart/hexo-theme-ocean) 為例。

### 安裝 Ocean

安裝 Ocean

```
$ git clone https://github.com/zhwangart/hexo-theme-ocean.git themes/ocean
```

修改網站設定 `<folder>/ _config.yml`

```
theme: ocean
```

在 `<folder>/theme/ocean/` 裡面的 `_config.yml` 也可以自己設定

```
# Menu
menu:
  Home: /
  Archives: /archives
  Gallery: /gallery
  About: /about
  Links: /links
rss: /atom.xml

# Miscellaneous
favicon: /favicon.ico
brand: /images/hexo.svg

# Ocean Video
# Because I put videos in multiple formats on the same path, I just labeled the path here.
ocean:
  overlay: true
  path: images/ocean/      # Video storage path, formats: mp4/ogg/webm
  brand: /images/hexo-inverted.svg      # Optional, a small logo

# Content
excerpt_link: Read More...

# fancybox
fancybox: true

# Local search
search_text: Search

# Gitalk
gitalk:
  enable: true
  clientID: # GitHub Application Client ID
  clientSecret: # Client Secret
  repo: # Repository name
  owner: # GitHub ID
  admin: # GitHub ID

# Valine
valine:
  enable: false    # Default: false.
  el: 'vcomments'    # The DOM element to be mounted on initialization.
  appId:    # Application appId from Leancloud.
  appKey:    # Application appKey from Leancloud.
  notify: false    # Mail notifier, Default: false.
  verify: true    # Validation code, Default: true.
  avatar: 'mp'    # Gravatar type.
  pageSize: '10'    # Number of pages per page.
  placeholder: '請輸入...'    #
```

重新啟動 server

```
$ hexo server
```

這樣就更改成功了！

![](https://miro.medium.com/max/1400/1*0Ji-3ir8nokLgZGvtzKVIw.png)

<p style ="text-align:center"><i>修改後的 Ocean 主題</i></p>

## 發佈文章

```
$ hexo new [layout] <title>
```

你可以在指令中指定文章的佈局（layout），預設為 post ，也可以透過修改 `_config.yml` 中的 `default_layout` 設定來指定預設佈局。

### 佈局（Layout）

Hexo 有三種預設佈局：post、page 和 draft，它們分別對應不同的路徑，而你所自定的其他佈局和 post 相同，都儲存至 source/_posts 資料夾。

![](https://miro.medium.com/max/824/1*eFVpwYyoIyZZ2x19f8r8Pw.png)

```
$ hexo new “postName”         # 新建文章
$ hexo new page “pageName”    # 新建頁面
$ hexo publish “draftName”    # 新建草稿
```

### 刪除文章

首先進入到 `source /_post` 資料夾中，找到 `helloworld.md`，在本地直接執行刪除。

![](https://miro.medium.com/max/1400/0*q6PVwd41rT6-e47s.png)

執行以下操作，一段時間後就會發現文章已刪除。

```
$ hexo clean       # 清除快取檔案和已產生的靜態檔案。
$ hexo d -g        # d = deploy, g = generate
```

### 其他指令

```
$ hexo server -p 5000
# 如果 port:4000 被佔用，可以用這個$ hexo render <file1> [file2] 
# 將文件渲染成靜態頁面
```

附錄：[Hexo 官方參考文件](https://hexo.io/docs/)
