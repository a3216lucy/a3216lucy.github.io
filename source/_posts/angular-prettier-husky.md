---
title: Angular 專案整合 Prettier、Husky 與 lint-staged
date: 2024-10-11 11:27:00
categories:
  - 技術筆記
tags:
  - Angular
  - Prettier
  - Husky
  - Git
---

團隊協作時，程式格式若完全仰賴人工約定，通常很快就會出現不同的縮排、引號與換行風格。將 Prettier 放進 Git commit 前的流程，可以讓提交進版本庫的程式碼維持一致，同時避免每次格式化都改動整個專案。

<!-- more -->

## 安裝套件

在 Angular 專案根目錄安裝 Prettier、Husky 與 lint-staged：

```shell
npm install --save-dev prettier husky lint-staged
```

`lint-staged` 會只處理已加入 Git 暫存區的檔案，比在 pre-commit 執行 `prettier --write .` 更安全，也能避免一次產生大量與功能無關的格式化變更。

## 建立 Prettier 設定

在專案根目錄建立 `.prettierrc.json`：

```json
{
  "singleQuote": true,
  "semi": true,
  "trailingComma": "all",
  "printWidth": 100,
  "tabWidth": 2,
  "bracketSameLine": false
}
```

Prettier 的預設值已適合大多數專案，設定檔應只保留團隊真正有共識的選項。舊版常見的 `jsxBracketSameLine` 已被 `bracketSameLine` 取代。

再建立 `.prettierignore`，排除不需要格式化的輸出與依賴檔案：

```gitignore
node_modules/
dist/
coverage/
.angular/
package-lock.json
```

## 設定 Git Hook

初始化 Husky：

```shell
npx husky init
```

這會建立 `.husky/pre-commit`，並在 `package.json` 加入 `prepare` script。將 `.husky/pre-commit` 的內容改為：

```shell
npx lint-staged
```

接著在 `package.json` 新增 lint-staged 設定：

```json
{
  "lint-staged": {
    "*.{js,jsx,ts,tsx,html,css,scss,json,md,yml,yaml}": "prettier --write"
  }
}
```

提交時，Hook 會格式化 staged files，並將修改後的內容放回暫存區。若格式化失敗，commit 會中止，開發者可先修正問題再重新提交。

## VS Code 儲存時自動格式化

安裝 Prettier VS Code 擴充功能後，可在 `.vscode/settings.json` 設定：

```json
{
  "editor.formatOnSave": true,
  "[typescript]": {
    "editor.defaultFormatter": "esbenp.prettier-vscode"
  },
  "[html]": {
    "editor.defaultFormatter": "esbenp.prettier-vscode"
  },
  "[scss]": {
    "editor.defaultFormatter": "esbenp.prettier-vscode"
  }
}
```

儲存時格式化改善個人開發體驗；pre-commit Hook 則是團隊最後一道一致性保護。兩者並不衝突。

## 小結

Prettier 負責統一格式，Husky 負責掛載 Git Hook，lint-staged 則確保只處理本次要提交的檔案。這套組合容易導入，也能讓程式碼 review 更聚焦在真正的邏輯變更。

