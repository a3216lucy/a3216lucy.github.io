---
title: 前端地圖工具怎麼選？從 2D、3D、圖資到空間分析整理
date: 2025-04-14 10:28:00
categories:
  - 技術筆記
tags:
  - GIS
  - Map
  - Angular
  - GeoJSON
---

地圖功能不只是「在頁面放一張地圖」。從一般標記、路線與行政區圖層，到地形、3D 模型與空間分析，不同需求適合的工具差異很大。
本文從前端渲染、圖資服務與後端空間資料三個層次整理常見選項。

<!-- more -->

## 先分清楚工具的角色

地圖系統常見的角色如下：

```text
空間資料庫／分析          圖資服務與資料格式              前端渲染與互動
PostGIS                 WMS / WMTS                  Leaflet
SpatiaLite      →       Vector Tiles / GeoJSON  →   OpenLayers
                        3D Tiles                    MapLibre GL JS
                                                    CesiumJS
```

- **GeoJSON**：交換點、線、多邊形等向量地理資料的格式。
- **WMS／WMTS**：向伺服器取得地圖影像或地圖瓦片的服務標準。
- **Vector Tiles**：以向量資料傳送地圖圖徵，適合前端動態樣式與大量資料渲染。
- **PostGIS**：PostgreSQL 的空間資料擴充，適合查詢範圍、距離、相交與包含關係。
- **Turf.js**：可在瀏覽器或 Node.js 端處理 GeoJSON 的空間運算工具。

## 常見前端地圖函式庫

| 工具 | 強項 | 適合情境 | 留意事項 |
| --- | --- | --- | --- |
| Leaflet | 輕量、學習門檻低 | 標記、簡單圖層、資料視覺化 | 大量向量資料與複雜 GIS 功能需額外設計 |
| OpenLayers | GIS 格式與投影支援完整 | WMS、WMTS、專業圖層與 2D GIS | API 較多，學習曲線較陡 |
| MapLibre GL JS | 向量瓦片與樣式客製 | 高互動 2D 地圖、客製底圖與圖層 | 需規劃 vector tile 與樣式來源 |
| CesiumJS | 真 3D 地球、地形、時間軸 | 3D 地形、模型、飛行軌跡與大範圍視覺化 | 專案體積與效能需求較高 |
| ArcGIS Maps SDK for JavaScript | 完整 ArcGIS 生態系 | 已使用 ArcGIS 平台的專業 GIS 系統 | 授權與平台整合成本需先評估 |
| Google Maps JavaScript API | 地點搜尋、路線與商業地點資料 | 導航、店家搜尋、一般商業地圖 | 用量與費用應依官方方案評估 |

## 2D、偽 3D 與真 3D 的差別

MapLibre GL JS 這類向量地圖函式庫主要處理平面地圖，可透過傾斜視角與建物高度營造 3D 效果，適合街道、商圈與資料圖層互動。

CesiumJS 則以球面地球與 3D 場景為核心，可處理地形高程、3D Tiles、模型、時間軸與相機漫遊。若需求只是顯示行政區、標記或熱區圖，直接使用 Cesium 往往會增加不必要的複雜度。

## Angular 整合原則

不論選擇哪套函式庫，都建議將地圖初始化與資源釋放封裝在元件中：

```ts
import { AfterViewInit, Component, OnDestroy } from '@angular/core';
import * as L from 'leaflet';

@Component({
  selector: 'app-map',
  template: '<div id="map" class="map"></div>',
  styles: '.map { height: 400px; }',
})
export class MapComponent implements AfterViewInit, OnDestroy {
  private map?: L.Map;

  ngAfterViewInit(): void {
    this.map = L.map('map').setView([25.033, 121.565], 13);
  }

  ngOnDestroy(): void {
    this.map?.remove();
  }
}
```

實際專案中，應改用 `ViewChild` 取得地圖容器，避免多個元件實例使用相同 id；並將圖層、事件與資料轉換邏輯拆到 service，減少元件複雜度。

## 依需求選擇

- **標記、簡單互動地圖**：優先評估 Leaflet。
- **WMS／WMTS、多投影或 GIS 圖層管理**：優先評估 OpenLayers。
- **高度客製化的向量地圖樣式**：評估 MapLibre GL JS。
- **地形、3D 模型、時間序列或球面視覺化**：評估 CesiumJS。
- **需要地點搜尋、導航與商業地圖資料**：評估 Google Maps 或其他商業地圖服務。
- **需要空間查詢與資料治理**：將 PostGIS 與後端 API 納入架構，而不是把所有計算放在瀏覽器。

## 小結

選擇地圖工具時，先回答資料從哪裡來、需要 2D 還是 3D、是否涉及專業 GIS 格式，以及預計的流量與授權成本。
前端函式庫只是其中一層；圖資授權、瓦片服務、資料量與後端空間分析能力，往往才是影響專案成敗的關鍵。

