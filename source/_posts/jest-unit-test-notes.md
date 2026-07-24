---
title: Jest 單元測試入門：生命週期與 Mock Function
date: 2026-05-12 09:32:00
categories:
  - 技術筆記
tags:
  - Jest
  - Unit Test
  - TypeScript
---

Jest 是常見的 JavaScript 與 TypeScript 測試框架。
除了撰寫斷言外，最常用的能力是安排測試生命週期，以及用 mock 隔離 API、資料庫或其他外部依賴。

<!-- more -->

## 測試案例與群組

`test` 與 `it` 功能相同，可搭配 `describe` 將相關案例分組：

```ts
describe('sum', () => {
  test('adds two numbers', () => {
    expect(1 + 2).toBe(3);
  });

  it('returns zero when both inputs are zero', () => {
    expect(0 + 0).toBe(0);
  });
});
```

開發期間可用 `test.only` 暫時只跑某個案例，或用 `test.skip` 暫時略過案例；提交前應移除它們，避免 CI 少跑測試。

## 測試生命週期

Jest 提供四個常用 Hook：

```ts
beforeAll(() => {
  // 整個 describe 區塊開始前執行一次
});

beforeEach(() => {
  // 每個測試案例前執行
});

afterEach(() => {
  // 每個測試案例後執行
});

afterAll(() => {
  // 整個 describe 區塊結束後執行一次
});
```

大多數單元測試應在 `beforeEach` 建立新的測試資料，避免前一個案例修改狀態後影響下一個案例。

## 使用 `jest.fn()` 建立 Mock

假設畫面需要透過 service 取得產品清單，可在測試中建立假的 service，而不呼叫真正的 API：

```ts
import { Observable, of } from 'rxjs';

type Product = { id: number; name: string };

interface ProductService {
  getProducts(): Observable<Product[]>;
}

const products: Product[] = [{ id: 1, name: 'Notebook' }];

const productService: jest.Mocked<ProductService> = {
  getProducts: jest.fn().mockReturnValue(of(products)),
};
```

接著可以驗證方法是否被呼叫，以及收到的結果是否符合預期：

```ts
productService.getProducts().subscribe((result) => {
  expect(result).toEqual(products);
});

expect(productService.getProducts).toHaveBeenCalledTimes(1);
```

若 mock 的是 Promise，可使用 `mockResolvedValue`：

```ts
const getProfile = jest.fn().mockResolvedValue({ id: 1, name: 'Kimi' });

await expect(getProfile()).resolves.toEqual({ id: 1, name: 'Kimi' });
```

## 清除 Mock 狀態

同一個 mock 被多個案例使用時，可在每次測試後清除呼叫紀錄：

```ts
afterEach(() => {
  jest.clearAllMocks();
});
```

`clearAllMocks` 只清除呼叫資訊；若還要移除 mock implementation，使用 `resetAllMocks`。
若以 `jest.spyOn` 取代原始方法，則用 `restoreAllMocks` 還原。

## 小結

好的單元測試應只驗證一個明確行為，並以 mock 隔離外部依賴。
先掌握 `describe`、`test`、`expect`、生命週期 Hook 與 `jest.fn()`，就足以覆蓋大部分元件與 service 的基礎測試情境。
