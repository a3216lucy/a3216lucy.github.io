---
title: 【SEO 導入】指南
date: 2024-12-06 10:14:00
categories:
- 技術筆記
tags:
- SEO
- Angular
---

紀錄一下如何為 Angular 專案導入 SEO，從觀念到實際改造成 SSR／預渲染的步驟。

<!-- more -->

## SEO 是什麼？

通過優化網站內容和結構，讓網站在搜索引擎（例如 Google ）的自然搜索結果中排名更高，從而吸引更多的訪客。

## SEO 與 Angular 的關係
### 1、Angular 預設使用 CSR

Angular 預設使用 **Client-Side Rendering (CSR)**，這意味著網頁內容是在瀏覽器上動態生成的。

這樣不符合 SEO，因爲：
- 搜索引擎的爬蟲可能無法執行 JavaScript，導致無法看到完整的內容。
- 內容加載時間可能延遲，降低搜索引擎的抓取效率。

### 2、使用 SSR 改善 SEO

**Server-Side Rendering (SSR)** 是 Angular 的解決方案，可以在伺服器上生成完整的 HTML，直接發送給瀏覽器。

這樣：
- 搜索引擎可以直接抓取完整的內容。
- 網頁的首次加載速度更快。
- 提高低功率設備的性能。

## 哪些頁面需要改成支援 SEO？

並非所有頁面都需要支援 SEO，特別是功能性頁面（例如登入、表單提交）通常不會被搜索引擎索引。

以下是判斷哪些頁面需要 SEO 的標準：
### 需要 SEO 支援的頁面：

1. **公開頁面**：不需要登入即可訪問的內容，例如：
	- 平台介紹、關於我們、聯繫我們。
	- 部落格文章、新聞、公告等資訊頁。
	- 特定產品頁、功能說明頁。
2. **需要高排名的頁面**：希望通過 Google 或其他搜索引擎吸引用戶的頁面。
3. **內容類型頁面**：
	- 包含描述性內容的頁面，例如文案、列表、圖片等。
	- 需要分享的頁面（社群平台上的連結預覽需要正確的標題、圖片和描述）。
4. **多語系頁面**：國際化網站，SEO 可幫助不同地區的搜索引擎更好地理解頁面語言版本。

### 不需要 SEO 支援的頁面：

1. **私有頁面**：如登入頁、用戶中心、個人資料設定等，通常不對搜索引擎公開。
2. **動態工具型頁面**：如查詢頁、表單提交頁、即時交互性工具。
3. **API 呼叫依賴頁面**：需要大量客戶端渲染，或數據全部來自後端 API 的頁面。

## 改成 SSR 的影響與注意事項

### 正面影響：

1. **更快的渲染時間**：伺服器端生成的 HTML 能直接返回完整的內容，提升用戶體驗。
2. **更好的 SEO 表現**：搜尋引擎能夠讀取完整的 HTML 結構，進一步改善頁面的可見性。
3. **預加載內容**：減少客戶端 API 請求數量。
### 潛在挑戰：

1. **伺服器負擔加重**：SSR 需要在伺服器端生成完整的 HTML，對伺服器的性能有額外需求。
2. **開發複雜性增加**：需要區分伺服器和客戶端的行為，例如 window 或 document 等 API 不能直接使用。
3. **安全性問題**：伺服器端渲染的數據可能需要額外處理以避免敏感資訊泄露。
4. **驗證碼、表單等動態功能的限制**：這些功能可能需要額外調整以在 SSR 環境下正常運行。

## 如何改造支援 SEO？

以下我將以 kimi-seo-demo 專案為例，跟著操作時，請將 kimi-seo-demo 置換成你所使用的專案名稱。

### 1. 確定使用 SSR 或預渲染（Prerendering）

在開始之前，要先確認哪些頁面要改成支援 SSR 和預渲染，哪些維持舊有的 CSR。

- **SSR**：適合需要即時渲染或依賴動態內容的頁面，例如多語系、需要數據 API 的頁面。
- **預渲染**：適合靜態內容固定的頁面（不需要動態生成內容），如平台介紹、FAQ、隱私政策等。

> Angular 中可使用  [**Angular Universal**](https://angular.io/guide/universal) **實現 SSR 或 Prerendering。**

### 2. Angular Universal 詳細步驟

#### (1) 設定 Angular Universal

在 Angular 專案中加入 SSR 支援：

```shell
ng add @nguniversal/express-engine
```

這個指令會自動：

• 安裝 Angular Universal 的相關套件
• 設定 SSR 的 Express 伺服器
• 生成一個 `server.ts` 文件，更新你的 `angular.json` 和 `package.json`

#### (2) 檢查並確認 angular.json 設定檔

確認  projects.architect 內有包含以下配置（若無，請將以下內容加入 angular.json）：

```json
"serve-ssr": {
  "builder": "@nguniversal/builders:ssr-dev-server",
  "configurations": {
	"development": {
	  "browserTarget": "kimi-seo-demo:build:dev",
	  "serverTarget": "kimi-seo-demo:server:dev"
	},
	"production": {
	  "browserTarget": "kimi-seo-demo:build:production",
	  "serverTarget": "kimi-seo-demo:server:production"
	}
  },
  "defaultConfiguration": "development"
},
"prerender": {
  "builder": "@nguniversal/builders:prerender",
  "options": {
	"routes": [
	  "/"
	]
  },
  "configurations": {
	"production": {
	  "browserTarget": "kimi-seo-demo:build:production",
	  "serverTarget": "kimi-seo-demo:server:production"
	},
	"development": {
	  "browserTarget": "kimi-seo-demo:build:development",
	  "serverTarget": "kimi-seo-demo:server:development"
	},
	"ssr": {
	  "browserTarget": "kimi-seo-demo:build:production",
	  "serverTarget": "kimi-seo-demo:build:server:ssr"
	}
  },
  "defaultConfiguration": "production"
}
```

#### (3) 確認更新 package.json 設定

在你的 package.json 中，設置應該如下：

```json
"dev:ssr": "ng run kimi-seo-demo:serve-ssr:development",
"serve:ssr": "node dist/kimi-seo-demo/server/main.js",
"build:ssr": "ng build && ng run kimi-seo-demo:server:production",
"prerender": "ng run kimi-seo-demo:prerender"
```

#### (4) 確認 main.server.ts 檔案 import 路徑位置是否正確

在安裝 Angular Universal 後，會自動在 src 下生成 `main.server.ts` ，請確認 `tsconfig.server.json` 中指定該檔案的路徑位置是否正確。

```json
//tsconfig.server.json
{
  "extends": "./tsconfig.app.json",
  "compilerOptions": {
    "outDir": "../out-tsc/server",
    "types": [
      "node"
    ]
  },
  "files": [
    "./main.server.ts",
    "../server.ts"
  ]
}

```
#### (5) 修改應用邏輯

##### 要特別注意：

> 由於 SSR 是透過 nodejs 預先將頁面進行編譯以及載入，因此是不支援 windows 或是 localStorage等其他在 Web 上的功能。需要額外加上判斷，透過瀏覽器去運行才不會報錯。

在每個組件中，可以根據 PLATFORM_ID 和 isPlatformServer 或 isPlatformBrowser 來判斷當前的渲染模式，由不同的渲染方式來決定資料抓取的策略。

**環境判斷：**
- 使用 isPlatformBrowser 判斷客戶端行為。
- 使用 isPlatformServer 判斷伺服器行為。
##### 範例

```ts
import { isPlatformBrowser, isPlatformServer } from '@angular/common';

constructor(@Inject(PLATFORM_ID) private platformId: any) { }

if (isPlatformBrowser(this.platformId)) {
  // CSR 渲染，客戶端邏輯
} else if (isPlatformServer(this.platformId)) {
  // SSR 渲染，伺服器端邏輯
}
```

localStorage 可以改成這樣的寫法

```ts
// 讓 node.js 讀懂的語法
import 'localStorage-polyfill';

global['localStorage'] = localStorage;
```

如果需要操作 DOM（與瀏覽器相關的 API，包含 window、document、 jQuery、牽涉到畫面甚至是動畫效果的），可以參考下面的方式來處理。

```ts
import { Inject, PLATFORM_ID } from '@angular/core';
import { DOCUMENT } from '@angular/common';

constructor(
	@Inject(PLATFORM_ID) private platformId
	@Inject(DOCUMENT) private document: Document,
) {
	this.isBrowser = isPlatformBrowser(this.platformId);
	if (this.isBrowser) window.addEventListener('scroll', (e) => this.onWindowScroll(e));
}

onWindowScroll(e) {
		if (window.scrollY > 30) {
			const element = this.document.getElementById('navbar-top');
			if (element) {
				element?.classList.remove('navbar-transparent');
				element?.classList.add('bg-blue');
			}
		} else {
			const element = this.document.getElementById('navbar-top');
			if (element) {
				element?.classList.add('navbar-transparent');
				element?.classList.remove('bg-blue');
			}
		}
	}
```

透過類似像這樣的方式來判斷當前執行此段程式碼是否為 Browser；若是在 SSR 編譯時期，則該段程式就不會被運行。

#### (6) 減少伺服器端 API 請求

對於需要 CSR 渲染的頁面，可以使用 TransferState 來延遲資料的抓取，這樣就能避免在 SSR 渲染過程中抓取資料。

1. **安裝** @angular/platform-browser ，以便使用 TransferState。
2. **在 SSR 渲染時先傳遞資料到客戶端**，然後在客戶端渲染時使用這些資料。

例如，對於 CSR 頁面，你可以在伺服器端不發出 API 請求，而是讓客戶端控制資料的更新。
##### 使用 TransferState

```ts
import { Injectable } from '@angular/core';
import { TransferState, makeStateKey } from '@angular/platform-browser';
import { HttpClient } from '@angular/common/http';

const DATA_KEY = makeStateKey<string>('data');

@Injectable({
  providedIn: 'root'
})
export class DataService {
  constructor(private http: HttpClient, private transferState: TransferState) {}

  getData() {
    const savedData = this.transferState.get(DATA_KEY, null);
    if (savedData) {
      return savedData;  // 使用 TransferState 傳遞的資料
    } else {
      return this.http.get('https://api.example.com/data');
    }
  }
}
```

#### (7) 測試 SSR

執行：

```shell
npm run build:ssr // 部署
npm run dev:ssr  // 執行 ssr
```

就可以開啟瀏覽器到 http://localhost:4200 確認 Web 是否正常執行。

### 3. 加入基本的 SEO 設定

#### (1) 增加 SEO Metadata

建立 html meta tag，讓網路爬蟲看懂你的網站。
在 angular 中，已有定義好 Meta class，我們可以在每頁的 ngOnInit 階段去定義每頁的 mega tag。

```ts
import { Title, Meta } from '@angular/platform-browser';

constructor(private title: Title, private meta: Meta) {}

ngOnInit() {
  this.title.setTitle('kimi SEO demo');
  this.meta.addTags([
    { name: 'description', content: 'Page description' },
    { name: 'keywords', content: 'SEO, Angular, Universal' },
  ]);

}
```

#### (2) 建立 robots.txt、sitemap.xml

> robots.txt **文件**：

• **控制搜索引擎爬蟲的行為**：robots.txt 是一個用來告訴搜索引擎爬蟲（如 Googlebot）哪些頁面或資料夾不應該被抓取或索引的文件。這對於保護網站的某些私密內容、減少爬蟲負擔、或對 SEO 進行策略性管理非常重要。
• **防止敏感內容被抓取**：可以指定不想被爬蟲抓取的頁面，例如登入頁面或管理頁面。
• **節省伺服器資源**：有時候，某些頁面不需要被搜索引擎索引（例如登錄頁面或資料夾），設置 robots.txt 可以防止這些頁面被爬蟲掃描，減少伺服器的負擔。

> sitemap.xml **文件**：

• **告訴搜索引擎網站的結構**：sitemap.xml 是一個用來告訴搜索引擎網站內所有頁面的檔案。它幫助搜索引擎快速了解網站的頁面結構，並有效地抓取網站內容。
• **提高索引效率**：對於大型網站，sitemap.xml 可以提高頁面的抓取效率，特別是當網站有很多頁面或是頁面經常更新的時候。
• **標示重要頁面**：可以在 sitemap.xml 中指定某些頁面的優先級，告訴搜索引擎哪些頁面應該被優先索引。

---

在專案根目錄下，新增  robots.txt 文件
##### 範例

```ts
//robots.txt
User-agent: *        // 這告訴所有爬蟲（*表示所有爬蟲）
Disallow: /login/    // 這告訴爬蟲不要抓取網站上的 /login/ 路徑
Disallow: /admin/
Allow: /public/      // 這告訴爬蟲可以抓取 /public/ 資料夾中的內容
Sitemap: https://www.kunyoutech.com.tw/sitemap.xml       // 這告訴爬蟲你的網站的 URL
```

在專案根目錄下，新增 sitemap.xml 文件
##### 範例

```xml
<?xml version="1.0" encoding="UTF-8"?>
<urlset xmlns="http://www.sitemaps.org/schemas/sitemap/0.9">
  <url>
	<!-- loc: 指定 URL 位置 -->
    <loc>https://www.example.com/</loc>
    <!-- lastmod：指定頁面最後一次更新的日期 -->
    <lastmod>2024-12-06</lastmod>
    <!-- changefreq：告訴爬蟲該頁面更新的頻率（如 daily、weekly、monthly） -->
    <changefreq>daily</changefreq>
    <!-- priority：指定該頁面的優先級（0.0 到 1.0），1.0 代表最高優先級 -->
    <priority>1.0</priority>
  </url>
  <url>

    <loc>https://www.example.com/about</loc>
    <lastmod>2024-12-06</lastmod>
    <changefreq>monthly</changefreq>
    <priority>0.8</priority>
  </url>
  <!-- 更多頁面 -->
</urlset>
```

##### 常見 XML sitemap 生成工具

• [XML-sitemaps.com](https://www.xml-sitemaps.com/)
• [Screaming Frog SEO Spider](https://www.screamingfrog.co.uk/seo-spider/)

#### (3) 測試 robots.txt、sitemap.xml

##### 方法一：使用 Google Search Console 的「robots.txt 測試工具」

Google Search Console 提供了一個內建的工具來測試 robots.txt 的配置：

- 登入 [Google Search Console](https://search.google.com/search-console)
- 選擇希望測試的網站
- 在左側選單中找到「設定」>「robots.txt 測試工具」
- 輸入想測試的 URL（例如 /login/），並檢查 Googlebot 是否可以抓取該頁面
- 這個工具會顯示哪些頁面被允許或拒絕抓取，確保 robots.txt 設置正確

##### 方法二：使用第三方工具「 Robots.txt Checker」檢查

• [Robots.txt Checker](https://www.mcanerin.com/robots-txt-checker/)。

這個工具可以幫助確認 robots.txt 文件的語法正確，並檢查是否存在配置錯誤。

---
##### 方法一：使用 Google Search Console 的「sitemap 測試工具」

Google Search Console 也有一個專門用來提交和測試 sitemap.xml 的功能：

- 登入 [Google Search Console](https://search.google.com/search-console)
- 選擇希望測試的網站。
- 在左側選單中選擇「索引」>「網站地圖」
- 點擊右上角的「新增/測試網站地圖」
- 輸入 sitemap.xml 文件的 URL（例如 https://www.example.com/sitemap.xml )，然後點擊「提交」， Google Search Console 會顯示提交狀態、抓取的結果，以及可能的錯誤。

##### 方法二：使用第三方工具檢查

有許多第三方工具可以檢查 sitemap.xml 是否有效，並確保網站地圖的語法正確：

-  [XML Sitemap Validator](https://www.xml-sitemaps.com/validate-xml-sitemap.html)：這個工具可以幫助你檢查 XML 地圖文件的有效性。
- [Screaming Frog SEO Spider](https://www.screamingfrog.co.uk/seo-spider/)：這是一個強大的 SEO 工具，可以抓取網站並生成 sitemap.xml，還能夠檢查 URL 是否符合 SEO 最佳實踐。

### 4. 準備靜態檔案，進行 Prerender

#### (1) 提供靜態內容

在專案的 `src/assets` 資料夾中準備靜態內容，因為 Angular CLI 會預設就會將這些內容當作是靜態資源，之後再透過 API 抓取這些內容顯示。

例如：將以下 JSON 檔案命名為 posts.json，用於存放文章的基本資訊（metadata）

```json
// src/assets/blog/posts.json
[
  { "title": "文章 01", "file": "post-1.md", "slug": "post-1" },
  { "title": "文章 02", "file": "post-2.md", "slug": "post-2" },
  { "title": "文章 03", "file": "post-3.md", "slug": "post-3" },
  { "title": "文章 04", "file": "post-4.md", "slug": "post-4" }
]
```

在同一目錄下，準備對應的文章檔案，如 post-1.md：

```md
# 文章標題
這是一篇範例文章。
```

要特別注意：

1. 在執行 Prerender 時，需要執行一個伺服器來提供這些靜態檔案
2. 抓不到資料時，要做出適當的錯誤處理

針對顯示靜態內容的部分，可以使用 lite-server 幫我們快速達成

```shell
npx lite-server --baseDir=./src
```

之後抓資料時必須指定完整的網址才能正常抓到資料：

```typescript
// 要看實際上 lite-server 開啟後的 port
posts$ = this.httpClient.get('http://localhost:3000/assets/blog/posts.json');
```

#### (2) 進行環境設定調整

實際上線時，除了進入該頁時本來就會有靜態內容之外，其餘的過程還是會走 Angular 本身的處理，當透過路由機制切換到其他頁面時，就不可能像 Prerender 時指定一個本地端的位置，因此要把 prerender 的環境也切出來。

我們可以建立一個 `environment.ssr.ts` 來處理伺服器端地抓檔問題，並指定靜態檔案的來源：

```typescript
export const environment = {
  production: true,
  assetsUrl: 'http://localhost:3000/'
};
```

其他的 `environment.ts` 和 `environment.prod.ts` 則可以加上 `assetUrl: ''` 設定，來抓取相對路徑的資源，使用 HttpClient 的寫法就會改成：

```typescript
import { environment } from '../../environment';

...

posts$ = this.httpClient.get(`${environment.assetsUrl}assets/blog/posts.json`)
```

 `angular.json` 內的設定也需要調整，讓我們在 prerender 時能換掉 `environment.ts`。

```json
{
  ...
  "projects": {
    "kimi-seo-demo": {
      ...
      "architect": {
        ...
        "server": {
          ...
          "configurations": {
            "production": { ... },
            "development": { ... },
            "ssr": { // 設定 ssr 取代檔案
              "outputHashing": "media",
              "fileReplacements": [
                {
                  "replace": "src/environments/environment.ts",
                  "with": "src/environments/environment.ssr.ts"
                }
              ]
            },
          },
          "defaultConfiguration": "production"
        },
        ...
        "prerender": {
          ...
          "configurations": {
            "production": { ... },
            "development": { ... },
            "ssr": { // 設定 ssr 在 server 模式取代檔案
              "browserTarget": "kimi-seo-demo:build:production",
              "serverTarget": "kimi-seo-demo:server:ssr"
            }
          },
          "defaultConfiguration": "production"
        }
      }
    }
  },
  "defaultProject": "kimi-seo-demo"
}
```

然後在 prerender 前先用 lite-server 把靜態伺服器打開後，執行 prerender 即可，當然也要記得改用 `ssr` 的相關設定 (或去改 `angular.json` 的 `defaultConfiguration` 也行)

```shell
npm run prerender -- --configuration ssr --routes-file post-routes.txt
```

#### (3) 撰寫 Resolver 來抓取資料

**a. 文章列表 Resolver**

負責抓取 posts.json 的資料，並傳遞給文章列表頁面：

```typescript
// PostsResolver
@Injectable({ providedIn: 'root' })
export class PostsResolver implements Resolve<any[]> {
  constructor(private httpClient: HttpClient) {}

  resolve(): Observable<any[]> {
    return this.httpClient
      .get<any[]>(`${environment.assetsUrl}assets/blog/posts.json`)
      .pipe(catchError(() => of([]))); // 若抓取失敗，回傳空陣列
  }
}
```

**b. 單篇文章 Resolver**

負責抓取對應文章的 Markdown 檔案，並轉換為 HTML：

```typescript
// PostResolver
import markdownIt from 'markdown-it';

@Injectable({ providedIn: 'root' })
export class PostResolver implements Resolve<string> {
  constructor(private httpClient: HttpClient) {}

  resolve(route: ActivatedRouteSnapshot): Observable<string> {
    const slug = route.paramMap.get('slug');
    return this.httpClient
      .get(`${environment.assetsUrl}assets/blog/${slug}.md`, { responseType: 'text' })
      .pipe(
        map((markdown) => markdownIt().render(markdown)), // 將 Markdown 轉為 HTML
        catchError(() => of('文章不存在'))
      );
  }
}
```

#### (4) 配置路由

設定路由以載入 Resolver 提供的資料

```typescript
const routes: Routes = [
  {
    path: '',
    resolve: { posts: PostsResolver },
    component: PostsComponent
  },
  {
    path: 'post/:slug',
    resolve: { post: PostResolver },
    component: PostComponent
  }
];

@NgModule({
  imports: [RouterModule.forRoot(routes)],
  exports: [RouterModule]
})
export class AppRoutingModule {}
```

#### (5) 在元件中顯示資料

**a. 文章列表頁面**

從 Resolver 中取得文章清單，並在頁面中顯示：

```typescript
// 在元件內，可以訂閱 ActivatedRoute.data 來拿到資料
@Component({ ... })
export class PostsComponent {
  posts$ = this.route.data.pipe(map((data) => data.posts));

  constructor(private route: ActivatedRoute) {}
}
```

接著在 html 內顯示取得的資料：

```html
<ul>
  <li *ngFor="let post of posts$ | async">
    <a [routerLink]="'/post/' + post.slug">{{ post.title }}</a>
  </li>
</ul>
```

**b. 單篇文章頁面**

從 Resolver 中取得文章內容，並轉換為安全的 HTML 顯示：

```ts
@Component({ ... })
export class PostComponent {
  content$ = this.route.data.pipe(
    map((data) => this.domSanitizer.bypassSecurityTrustHtml(data.post))
  );

  constructor(private route: ActivatedRoute, private domSanitizer: DomSanitizer) {}
}
```

```html
<div [innerHTML]="content$ | async"></div>
```

#### (6) 執行 Prerender，預覽結果

**a. 產生靜態路由**

準備一個包含需要 Prerender 的路由檔案，例如 post-routes.txt：

```txt
/
post/post-1
post/post-2
post/post-3
```

**b. 執行 Prerender**

啟動靜態伺服器，並執行 Prerender：

```shell
npx lite-server --baseDir=./src
npm run prerender -- --configuration ssr --routes-file post-routes.txt
```

---
##### 方法二

前面我們用了 lite-server 來跑模擬的伺服器，不過 lite-server 在目錄網址時不會提供裡面的 `index.html` 當作內容，因此我們可以自己建立一個伺服器來解決這個問題。

**a. 使用模擬伺服器**

先安裝一些套件：

```shell
npm i -D ts-node connect serve-static @types/node @types/connect @types/serve-static
```

接著建立一支伺服器程式 `ssr-preview.ts`：

```typescript
import * as connect from 'connect';
import * as serveStatic from 'serve-static';
import { join } from 'path';
// 靜態檔案路徑，請自行修改
const distFolder = join(process.cwd(), 'dist', 'ngx-universal-prerender-demo', 'browser');
connect()
  .use(serveStatic(distFolder))
  .listen(4001, () => console.log('Server running on 4001...'));
```

和要給它使用的 `tsconfig.ssr-preview.json`：

```json
{
  "extends": "./get-page-posts.ts",
  "compilerOptions": {
    "module": "commonjs"
  }
}
```

啟動伺服器：

```shell
npx ts-node --project .\tsconfig.ssr-preview.json .\ssr-preview.ts
```

接著可以隨意瀏覽任一個頁面的原始碼，內容都會是被靜態產生的，這樣就算是完成了！

## 總結

依照上述步驟執行，就可以在現行專案導入修改支援 SEO 了。

---
### 附錄：參考連結

- [Mike｜Angular Universal - 使用 Prerender 建立自己的 Static Site Generator](https://fullstackladder.dev/blog/2021/10/16/static-site-generator-using-angular-universal-prerender/)
- [Mike｜Angular Universal - 使用 TransferState 解決畫面閃爍問題](https://fullstackladder.dev/blog/2021/10/31/angular-universal-transfer-state/)
- [Angular Universal教學-將現有專案導入Server Side Render](https://www.dotblogs.com.tw/Leo_CodeSpace/2020/07/24/161101)
- [保哥｜如何在 Angular CLI 建立的專案加入 Angular Universal 伺服器渲染功能](https://blog.miniasp.com/post/2017/06/18/How-to-setup-Angular-Universal-in-an-Angular-CLI-project)
