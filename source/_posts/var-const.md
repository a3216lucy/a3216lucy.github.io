---
title: JS 宣告變數：var、let、const 差異與使用時機
date: 2021-10-10 04:36:50
categories:
  - 技術筆記
tags:
  - JavaScript
  - Studies
---

JavaScript 可以用 `var`、`let`、`const` 宣告變數，但在現代 JavaScript 中，最實用的原則其實很簡單：**預設使用 `const`；確定需要重新賦值時才用 `let`；新程式碼避免使用 `var`。**

理解三者的作用域、提升與重複宣告差異後，就能知道這個原則從何而來。

<!-- more -->

## 快速結論

```ts
const apiUrl = '/api'; // 預設使用 const

let page = 1; // 值需要重新賦值時使用 let
page += 1;

var legacyValue = 'old code'; // 只在維護舊程式碼時遇到
```

| 特性 | `var` | `let` | `const` |
| --- | --- | --- | --- |
| 作用域 | 函式作用域 | 區塊作用域 | 區塊作用域 |
| 可重新賦值 | 可以 | 可以 | 不可以 |
| 可在同一作用域重複宣告 | 可以 | 不可以 | 不可以 |
| 宣告前存取 | 得到 `undefined` | TDZ，拋出錯誤 | TDZ，拋出錯誤 |
| 新程式碼建議 | 不建議 | 需要變動時使用 | 預設使用 |

## 1. 作用域：`var` 是函式作用域，`let` 與 `const` 是區塊作用域

區塊作用域指的是 `{}` 所包住的範圍，例如 `if`、`for`、`while` 或單純的大括號。

```ts
if (true) {
  var varMessage = 'var 不受區塊限制';
  let letMessage = 'let 只存在於區塊內';
  const constMessage = 'const 只存在於區塊內';
}

console.log(varMessage); // 'var 不受區塊限制'
console.log(letMessage); // ReferenceError
console.log(constMessage); // ReferenceError
```

`var` 不會被 `if` 區塊限制，而是以最近的函式為範圍：

```ts
function getStatus() {
  if (true) {
    var status = 'ready';
  }

  return status; // 'ready'
}

console.log(status); // ReferenceError：函式外無法存取
```

區塊作用域可縮小變數可被存取與修改的範圍，因此更容易閱讀，也能降低同名變數互相影響的機率。

## 2. `const` 不能重新賦值，不代表物件完全不可變

`const` 限制的是「變數綁定」不能改指向另一個值：

```ts
const userName = 'Kimi';
userName = 'Alex'; // TypeError
```

若值是物件或陣列，仍可以修改其內部內容：

```ts
const user = { name: 'Kimi' };

user.name = 'Alex'; // 可以
user = { name: 'Mina' }; // TypeError：不能重新指派 user
```

因此，`const` 不等於 immutable。若需要不可變資料，仍要避免直接修改物件，或使用 `Object.freeze`、immutable update 等方式。

## 3. `for` 迴圈與非同步 callback

以下是 `var` 最常見的陷阱之一：

```ts
for (var i = 0; i < 3; i++) {
  setTimeout(() => {
    console.log(i);
  }, 100);
}

// 3
// 3
// 3
```

`setTimeout` 的 callback 會在迴圈結束後才執行，而三個 callback 讀到的是同一個 `i`。迴圈結束時，`i` 已經是 `3`。

改用 `let` 後，JavaScript 會為每一次迭代建立各自的綁定：

```ts
for (let i = 0; i < 3; i++) {
  setTimeout(() => {
    console.log(i);
  }, 100);
}

// 0
// 1
// 2
```

舊程式碼可用 IIFE 解決 `var` 的問題，但新程式碼直接使用 `let` 更清楚。

## 4. Hoisting 與暫時死區（TDZ）

`var`、`let` 與 `const` 的宣告都與 JavaScript 的執行階段建立流程有關，常被統稱為 hoisting。不過它們在宣告前被存取時，行為不同。

```ts
console.log(count); // undefined
var count = 1;
```

上例可概念化為：

```ts
var count;
console.log(count); // undefined
count = 1;
```

這讓程式看起來像是「尚未宣告卻可使用」，容易隱藏錯誤。

`let` 與 `const` 也會在進入作用域時建立綁定，但在執行到宣告前不可存取；這段區間稱為暫時死區（Temporal Dead Zone，TDZ）：

```ts
console.log(greeting); // ReferenceError
let greeting = 'hello';
```

TDZ 促使變數先宣告、後使用，讓控制流程更容易理解。實務上不需要刻意依賴任何一種 hoisting 行為，應將宣告放在使用位置之前。

## 5. 重複宣告

`var` 允許在同一作用域用相同名稱重複宣告：

```ts
var theme = 'light';
var theme = 'dark';

console.log(theme); // 'dark'
```

這可能在大型檔案或多人協作時意外覆蓋變數。`let` 與 `const` 會直接阻止這類錯誤：

```ts
const theme = 'light';
const theme = 'dark'; // SyntaxError: Identifier 'theme' has already been declared
```

## 實務使用準則

可以按照下面順序選擇：

1. 先使用 `const`。
2. 如果變數在同一作用域內需要重新賦值，改為 `let`。
3. 不在新程式碼中使用 `var`；只有維護舊專案時才需要理解它的行為。

```ts
const maxRetries = 3;
let retries = 0;

while (retries < maxRetries) {
  retries += 1;
}
```

這個原則能讓「哪些值會變動」在閱讀程式碼時一目了然，也能避開 `var` 的函式作用域、重複宣告與非同步迴圈陷阱。
