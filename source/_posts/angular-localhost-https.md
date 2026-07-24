---
title: Angular 本機開發如何使用 HTTPS 啟動 ng serve
date: 2025-03-05 10:12:00
categories:
  - 技術筆記
tags:
  - Angular
  - HTTPS
  - localhost
---

當本機開發需要測試 HTTPS 限定的瀏覽器功能，例如 Secure Cookie、Web Crypto API 或第三方登入流程時，可以讓 Angular 開發伺服器透過 HTTPS 啟動。

本篇整理憑證、`ng serve` 與 API Proxy 的基本設定方式。

<!-- more -->

## 為什麼本機開發需要 HTTPS？

許多瀏覽器 API 與安全性設定只會在安全來源（secure context）中運作。若要模擬正式環境的 HTTPS 行為，便可在本機使用憑證啟動 Angular 開發伺服器。

> 本文的憑證僅供本機開發使用，不應用於正式環境。

## 1. 建立本機憑證

可使用 OpenSSL 建立自簽憑證。在專案根目錄建立僅供本機使用的資料夾後，執行下列指令：

```shell
mkdir -p .cert
openssl req -x509 -newkey rsa:4096 -sha256 -nodes \
  -keyout .cert/localhost-key.pem \
  -out .cert/localhost-cert.pem \
  -days 365 \
  -subj "/CN=localhost" \
  -addext "subjectAltName=DNS:localhost,IP:127.0.0.1"
```

執行後會產生兩個檔案：

- `localhost-key.pem`：私鑰，必須妥善保管。
- `localhost-cert.pem`：本機使用的自簽憑證。

> 若使用的 OpenSSL 版本不支援 `-addext`，可先移除該參數完成產生；或改用支援 Subject Alternative Name 的憑證工具。

## 2. 使用 HTTPS 啟動 Angular

Angular CLI 的 `ng serve` 提供 `--ssl`、`--ssl-key` 與 `--ssl-cert` 選項，可直接指定憑證檔案：

```shell
ng serve --ssl \
  --ssl-key .cert/localhost-key.pem \
  --ssl-cert .cert/localhost-cert.pem
```

預設啟動後，可透過以下網址開啟應用程式：

```text
https://localhost:4200
```

自簽憑證不受作業系統或瀏覽器信任，因此初次開啟時可能出現憑證警告。這是本機測試的正常現象；若希望不出現警告，可改用能將本機開發憑證加入信任鏈的工具，例如 `mkcert`。

## 3. 透過 Proxy 存取 HTTP API

當前端改用 HTTPS、後端 API 仍使用 HTTP 時，瀏覽器會封鎖直接發出的 HTTP 請求，這就是混合內容（Mixed Content）問題。

可將前端請求統一使用相對路徑，例如 `/api/users`，再由 Angular 開發伺服器代理至本機後端。

建立 `proxy.conf.json`：

```json
{
  "/api/**": {
    "target": "http://127.0.0.1:5000",
    "secure": false,
    "changeOrigin": true
  }
}
```

接著啟動開發伺服器：

```shell
ng serve --ssl \
  --ssl-key .cert/localhost-key.pem \
  --ssl-cert .cert/localhost-cert.pem \
  --proxy-config proxy.conf.json
```

前端呼叫 `/api/users` 時，瀏覽器只會看到對 `https://localhost:4200` 的請求；開發伺服器則會在內部將請求轉送到 `http://127.0.0.1:5000`。

> Proxy 設定修改後，需要重新啟動 `ng serve` 才會生效。新版 Angular 的 Vite 開發伺服器中，`/api/**` 可匹配所有 API 子路徑。

## 4. 不要將私鑰提交至版本控制

將本機憑證放在 `.cert` 資料夾後，請在 `.gitignore` 加入：

```gitignore
# Local development HTTPS certificates
.cert/
*.pem
```

私鑰一旦提交到遠端儲存庫，即使日後刪除，仍可能保留在 Git 歷史紀錄中。若不慎提交，應立即撤銷該憑證並重新建立新的私鑰。

## 小結

透過自簽憑證與 Angular CLI 的 SSL 選項，可以在本機模擬 HTTPS 環境；若 API 尚未提供 HTTPS，則可搭配 Proxy 避免混合內容問題。這些設定只應限於開發環境，正式環境仍應使用由受信任憑證機構簽發的憑證。
