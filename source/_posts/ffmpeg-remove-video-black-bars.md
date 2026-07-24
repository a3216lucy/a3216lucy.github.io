---
title: 使用 FFmpeg 偵測並裁切影片黑邊
date: 2026-03-25 11:37:00
categories:
  - 技術筆記
tags:
  - FFmpeg
  - Video
---

影片被嵌入不同尺寸的版面時，若來源檔本身含有上下或左右黑邊，使用 CSS 的 `object-fit: cover` 也無法真正移除它們。
這時可先用 FFmpeg 偵測可裁切範圍，再輸出新的影片檔。

<!-- more -->

## 1. 偵測黑邊範圍

對來源影片執行 `cropdetect`：

```shell
ffmpeg -i input.mp4 -vf "cropdetect" -f null - 2>&1 | tail -20
```

輸出中會出現類似以下的建議：

```text
crop=1616:1072:152:4
```

四個數值依序是：

```text
crop=寬度:高度:左側偏移:上側偏移
```

建議多觀察幾行輸出，選擇反覆出現且合理的數值。
若影片畫面本身有長時間的全黑場景，可能需要從不同時間點取樣確認。

## 2. 輸出裁切後的影片

先輸出成新檔，保留來源影片以便驗證：

```shell
ffmpeg -i input.mp4 \
  -vf "crop=1616:1072:152:4" \
  -c:v libx264 \
  -crf 18 \
  -preset slow \
  -c:a copy \
  output-crop.mp4
```

- `crop`：使用偵測出的裁切範圍。
- `libx264`：以 H.264 重新編碼影片。
- `crf 18`：品質與檔案大小的常用平衡值；數字愈小品質愈高、檔案也愈大。
- `preset slow`：以較長處理時間換取較佳壓縮效率。
- `-c:a copy`：直接複製原始音軌，不重新編碼。

若影片確定是無聲的背景影片，或不需要保留音軌，才改用 `-an`：

```shell
ffmpeg -i input.mp4 -vf "crop=1616:1072:152:4" -c:v libx264 -crf 18 -an output-crop.mp4
```

## 3. 確認後再取代檔案

先播放 `output-crop.mp4`，確認畫面比例、音訊與檔案大小都符合預期；確認後再以版本控制或備份方式取代原檔，避免直接覆寫而無法復原。

## 小結

`cropdetect` 用來取得裁切建議，`crop` 負責實際裁切，先輸出新檔並確認結果，是處理影音素材時最安全的流程。
