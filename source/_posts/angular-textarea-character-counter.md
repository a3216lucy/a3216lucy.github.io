---
title: Angular Reactive Forms：textarea 顯示目前字數與上限
date: 2024-12-27 16:21:00
categories:
  - 技術筆記
tags:
  - Angular
  - Reactive Forms
  - textarea
---

表單中的文字說明欄位常需要限制輸入長度，並即時顯示目前字數。Angular Reactive Forms 可以直接透過 `FormControl` 綁定狀態，不需要手動操作 DOM 或以固定 id 尋找元素。

<!-- more -->

## 建立表單控制項

在元件中設定最大長度與非 nullable 的控制項：

```ts
import { Component } from '@angular/core';
import { FormControl, ReactiveFormsModule, Validators } from '@angular/forms';

@Component({
  selector: 'app-remark-field',
  imports: [ReactiveFormsModule],
  templateUrl: './remark-field.component.html',
  styleUrl: './remark-field.component.scss',
})
export class RemarkFieldComponent {
  readonly maxLength = 200;

  readonly remarkControl = new FormControl('', {
    nonNullable: true,
    validators: [Validators.maxLength(this.maxLength)],
  });

  get currentLength(): number {
    return this.remarkControl.value.length;
  }
}
```

## 在範本顯示字數

將計數器放在欄位容器內，並以 Angular binding 顯示目前長度：

```html
<div class="textarea-field">
  <label for="remark">說明</label>

  <textarea
    id="remark"
    rows="4"
    [formControl]="remarkControl"
    [attr.maxlength]="maxLength"
    placeholder="請輸入說明"
  ></textarea>

  <span class="character-count" aria-live="polite">
    {{ currentLength }} / {{ maxLength }}
  </span>
</div>
```

```scss
.textarea-field {
  position: relative;
}

textarea {
  box-sizing: border-box;
  min-height: 8rem;
  padding-bottom: 1.5rem;
  width: 100%;
}

.character-count {
  bottom: 0.5rem;
  color: #6b7280;
  font-size: 0.75rem;
  pointer-events: none;
  position: absolute;
  right: 0.75rem;
}
```

顯示內容為「目前字數／最大字數」。若要顯示剩餘字數，可改成 `{{ maxLength - currentLength }}`，並將文案明確寫成「剩餘字數」。

## 為什麼不要直接操作 DOM？

使用 `document.getElementById()` 與 `innerHTML` 會讓資料狀態分散在 Angular 之外，也容易因重複 id、多個元件實例或伺服器端渲染而出現問題。將 UI 完全由 FormControl 與 template binding 驅動，可讓驗證、畫面與提交資料保持同步。

## 小結

`maxlength` 負責限制輸入，`Validators.maxLength` 負責表單驗證，template binding 則負責顯示狀態。三者搭配即可完成可重用且易維護的文字字數提示欄位。

