---
title: Angular 前後台專案該選 Monorepo 還是雙 Repo？
date: 2026-06-16 00:00:00
categories:
  - 技術筆記
tags:
  - Angular
  - Monorepo
  - Nx
  - CI/CD
---

當一個 Angular 前台系統即將新增後台管理介面時，團隊常會面臨架構選擇：前後台應放在同一個 Repository，還是拆成兩個獨立 Repository？

這個選擇不只影響目錄結構，也會影響共用程式碼、CI/CD 執行時間、權限控管與未來交付原始碼的方式。

本文比較三種常見方案，並整理適合各自情境的判斷原則。

<!-- more -->

## 常見情境

假設團隊有以下需求：

- 同一個團隊同時維護前台與後台。
- 兩個應用程式會共用 API 型別、驗證邏輯或 UI 元件。
- 專案已使用 CI/CD，並希望避免不必要的建置與測試。
- 未來可能加入程式碼品質掃描與架構規範。

常見的選項可分成三種：Angular CLI Workspace、Nx Workspace 和 雙 Repository。

## 方案一：Angular CLI Workspace

Angular CLI Workspace 可以在同一個 Repository 中管理多個應用程式。例如保留既有前台，再用 Angular CLI 建立後台應用程式，統一由 `angular.json` 管理。

```text
workspace/
├── projects/
│   ├── frontend/
│   └── admin/
├── angular.json
├── package.json
└── tsconfig.json
```

### 優點

- 對 Angular 團隊來說學習成本低。
- 不必額外導入 Monorepo 工具。
- 前後台可共用 TypeScript path alias 與部分程式碼。
- 適合應用程式數量少、架構仍單純的專案。

### 限制

- 程式碼共用與依賴方向主要仰賴團隊約定，缺少明確的邊界約束。
- 可自行設計 CI 規則只建置特定應用程式，但不具備以專案依賴圖為核心的 affected 工作流程。
- 現代 Angular CLI 已具備 persistent disk cache，因此不應把「沒有快取」當成缺點；真正差異在於多專案任務編排、變更影響分析與架構治理能力。

### 適合情境

- 應用程式數量少，且短期內不會快速成長。
- 前後台共用程式碼不多。
- 團隊優先追求簡單、低遷移成本的方案。

## 方案二：Nx Workspace

Nx 是以專案圖（project graph）管理多個應用程式與函式庫的工具。前台、後台與共用程式碼可被明確拆分，並由 Nx 管理它們的依賴關係。

```text
workspace/
├── apps/
│   ├── frontend/
│   └── admin/
├── libs/
│   ├── shared-api/
│   ├── shared-ui/
│   └── shared-utils/
├── nx.json
├── package.json
└── tsconfig.base.json
```

### 優點

- 可用 `nx affected` 依變更範圍，只執行受影響專案的 lint、test 或 build。
- 內建任務快取機制，可降低本機與 CI 重複執行相同任務的成本。
- 可用 tags 與 ESLint 規則限制模組依賴方向，例如禁止 feature 直接引用另一個 feature。
- 共用 API model、UI 元件與工具函式時，不需要額外發佈內部 npm 套件。
- 適合隨專案數量成長而持續維護的架構。

### 限制

- 團隊需要理解 project、target、task cache 與 dependency graph 等概念。
- 設定與升級成本高於單純的 Angular CLI Workspace。
- `libs/` 不代表所有程式碼都應共用；若沒有清楚的依賴規範，Monorepo 仍可能演變成難以維護的大型程式庫。

### 適合情境

- 同一團隊長期維護多個應用程式。
- 前後台共享 API 型別、設計系統或領域邏輯。
- CI/CD 時間開始成為開發流程的瓶頸。
- 需要以工具落實模組邊界與程式碼品質規範。

## 方案三：前後台拆成雙 Repository

前台與後台各自維護獨立 Repository。
若需要共用程式碼，通常會把它抽成可版本化的 npm 套件，或改由 API schema 產生型別與 client。

```text
frontend-repository/
├── src/
├── package.json
└── .gitlab-ci.yml

admin-repository/
├── src/
├── package.json
└── .gitlab-ci.yml
```

### 優點

- 專案、權限與部署流程天然隔離。
- 可以獨立交付某一個應用程式的完整原始碼。
- 前後台可以使用不同技術棧、不同發版節奏與不同維護團隊。

### 限制

- 共用 API model 或元件需要發佈、安裝與升版流程。
- 跨 Repository 的型別相容性與變更追蹤成本較高。
- CI/CD、品質掃描與依賴管理通常需要維護兩份設定。

### 適合情境

- 前後台由不同團隊維護，或存取權限必須隔離。
- 合約或資安規範要求只交付其中一個應用程式的原始碼。
- 兩個應用程式技術棧不同，或發版節奏完全無關。

## 三種方案比較

| 面向 | Angular CLI Workspace | Nx Workspace | 雙 Repository |
| --- | --- | --- | --- |
| 共用 API model／元件 | path alias 與團隊約定 | 以 libraries 明確管理 | npm 套件或 schema 產生 |
| 變更影響分析 | 需自行設計 CI 規則 | `nx affected` | 各自獨立 |
| 任務快取 | Angular CLI disk cache | 專案任務快取，可延伸至 CI | 各自管理 |
| 模組邊界 | 主要靠約定 | 可由 lint 規則強制 | Repository 天然隔離 |
| 部署獨立性 | 需自行設定 | 需自行設定 | 天然獨立 |
| 原始碼交付隔離 | 較困難 | 較困難 | 容易 |
| 初始導入成本 | 低 | 中 | 低 |
| 長期維護成本 | 中 | 視治理而定，通常較低 | 共用程式碼多時較高 |

## 原始碼交付與掃描需求會改變答案

技術便利性並不是唯一考量。如果必須只交付前台原始碼，而後台程式碼不得隨附，雙 Repository 往往比 Monorepo 更合適。

雖然可以從 Monorepo 匯出某個應用程式目錄，例如：

```shell
git archive HEAD:apps/frontend --output=frontend-source.zip
```

但這通常不是完整的交付品：前台依賴的 libraries、根目錄設定檔與 lock file 可能不在封包內。若要讓交付後的程式碼可獨立閱讀、建置與掃描，還需額外維護匯出規則與相依清單。

相對地，動態掃描（DAST）著重於已部署站台的行為，與程式碼放在 Monorepo 或雙 Repository 沒有直接關係；靜態掃描（SAST）則兩種架構都能支援，差別主要在報告與品質門檻要統一管理或分開管理。

## 選擇 Nx 後，先訂好共用邊界

採用 Nx 時，最常見的問題不是工具設定，而是把所有東西都放進 `shared`。

較健康的原則是：

- `apps/` 放各應用程式專屬的頁面、feature 與流程。
- `libs/` 只放真正跨應用程式共用、且有穩定介面的程式碼。
- 依領域或用途區分 library，例如 `shared-api`、`shared-ui`、`shared-utils`。
- 用 tags 搭配 `@nx/enforce-module-boundaries` 限制依賴方向。

例如，`frontend` 與 `admin` 都可引用 `shared-api`，但 `frontend` 不應直接引用 `admin` 內的 feature。這樣才能保留程式碼共用的好處，同時避免應用程式彼此糾結。

## 結論

如果前後台由同一團隊長期維護、共享程式碼比例高，而且 CI/CD 效率與架構治理是重點，Nx Workspace 通常是最平衡的選擇。

如果專案規模小、應用程式不多，Angular CLI Workspace 已能滿足需求，不必為了 Monorepo 而增加工具複雜度。

如果有嚴格的原始碼隔離、不同維護團隊或不同技術棧，雙 Repository 會是更清楚且風險更低的選擇。架構沒有絕對答案；先釐清共用程度、交付邊界與團隊協作方式，才能選到適合長期維護的方案。
