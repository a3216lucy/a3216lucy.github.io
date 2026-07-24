---
title: Angular 本機開發遇到 CORS 問題：使用 Proxy 代理 API
date: 2025-02-20 09:43:00
categories:
  - 技術筆記
tags:
  - Angular
  - CORS
  - localhost
---

> 在本機開發 Angular 應用程式時，前端通常運行於 `http://localhost:4200`，API 則可能位於另一個 port 或網域。若直接從瀏覽器呼叫 API，便可能遇到 CORS（Cross-Origin Resource Sharing，跨來源資源共用）錯誤。

本篇以 Angular 開發伺服器的 Proxy 功能處理本機開發情境，讓瀏覽器以同源方式呼叫 API，而由開發伺服器在內部轉送請求。

<!-- more -->

## 為什麼會出現 CORS 錯誤？

瀏覽器會以「協定、網域、Port」判斷兩個來源是否相同。

以下兩個網址的埠號不同，因此屬於不同來源：

```text
http://localhost:4200
http://127.0.0.1:5000
```

當前端直接呼叫 `http://127.0.0.1:5000/api/users` 時，後端必須在回應中明確允許 `http://localhost:4200`，否則瀏覽器會阻擋程式讀取回應。

正式解法仍應由後端正確設定 CORS。
若是在本機開發階段，則可使用 Angular Proxy，避免前端程式直接跨來源請求。

## 使用 Angular Proxy 代理 API 請求

### 1. 建立 Proxy 設定檔

在 Angular 專案根目錄建立 `proxy.conf.json`：

```json
{
  "/api/**": {
    "target": "http://127.0.0.1:5000",
    "secure": false,
    "changeOrigin": true
  }
}
```

設定含義如下：

- `"/api/**"`：攔截所有以 `/api/` 開頭的請求。
- `target`：實際 API 伺服器的位置；此處為示範用的本機後端。
- `secure: false`：Proxy 連線至使用自簽憑證的 HTTPS 後端時，不驗證其憑證。若後端是 HTTP，此設定通常不影響結果。
- `changeOrigin: true`：轉送請求時調整 `Host` 標頭，部分後端服務需要此設定。

### 2. 啟動開發伺服器

啟動 Angular 時傳入 Proxy 設定：

```shell
ng serve --proxy-config proxy.conf.json
```

也可以將 `proxyConfig` 寫入 `angular.json` 的 `serve.options`，之後直接執行 `ng serve`：

```json
{
  "projects": {
    "my-app": {
      "architect": {
        "serve": {
          "options": {
            "proxyConfig": "proxy.conf.json"
          }
        }
      }
    }
  }
}
```

實際的專案設定可能還會包含 builder、build target 與 configurations；只需要在既有的 `serve.options` 加上 `proxyConfig` 即可。

### 3. 前端改用相對路徑

前端不應再寫死後端的完整網址：

```ts
// 不建議：瀏覽器會直接對另一個來源發出請求
fetch('http://127.0.0.1:5000/api/users');
```

改為呼叫相對路徑：

```ts
// Angular 開發伺服器會將請求代理至 target
fetch('/api/users');
```

瀏覽器看到的請求仍是 `http://localhost:4200/api/users`，因此不會觸發跨來源限制；Angular 開發伺服器再將它轉送到 `http://127.0.0.1:5000/api/users`。

## 常見問題

### 修改 Proxy 設定後沒有生效

Proxy 設定檔不會自動更新。
修改 `proxy.conf.json` 後，請停止並重新執行 `ng serve`。

### Proxy 無法連線至 `localhost`

部分 Node.js 與作業系統環境中，`localhost` 可能優先解析為 IPv6 位址 `::1`；若後端僅監聽 IPv4，就可能出現 `ECONNREFUSED`。此時可將 `target` 改成 `http://127.0.0.1:5000`。

### 是否應該停用瀏覽器的 CORS 安全性？

不建議。
以停用瀏覽器安全性來測試，無法反映真實使用者環境，也可能讓其他分頁暴露在風險中。
開發時應優先使用 Proxy；部署環境則由後端設定明確、最小範圍的 CORS 規則。

## 小結

Angular Proxy 是處理本機開發 CORS 問題的實用工具；前端以相對路徑呼叫 API，開發伺服器負責轉送請求。
不過它只解決開發環境的便利性，正式環境仍需由 API 伺服器妥善設定 CORS。
