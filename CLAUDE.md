# CLAUDE.md — 日本名所。實地打卡

## 項目概覽
單一 `index.html` 的日本景點 GPS 打卡 web app，靈感來自御朱印帳。
用戶需親身到達景點範圍內（500m），用相機拍照才可打卡。

## 技術棧
- **純 HTML/CSS/JS**，無框架，無 build step
- **Leaflet.js** — 互動地圖
- **localStorage** — 本地資料持久化
- **Canvas API** — 相片圓形裁剪 + 壓縮
- **navigator.geolocation** — GPS 驗證
- **LXGW WenKai TC** — 字體（廣東話字符支援 + 文青感）

## 部署
- 開發：ngrok 做 HTTPS tunnel（相機 / GPS 需要 HTTPS）
- 生產目標：GitHub Pages（`https://用戶名.github.io/japan47/`）
  - 固定 domain 解決 iOS ITP 7天自動清除 localStorage 問題

## 資料結構

### SPOTS
```js
{ id, name, nameEn, pref, prefLabel, emoji, lat, lng, radiusMeters: 500 }
```

### localStorage checkins
```js
{ [spot.id]: { dataUrl, ts: Date.now() } }
```

### spotLayers（地圖物件）
```js
{ spot, marker, circle, photoMarker: null, photoDataUrl: null }
```

## 相片壓縮
- 160px 正方形，JPEG quality 0.42，Canvas 圓形 arc clip
- 每張約 10–18KB（base64）
- iOS Safari localStorage 上限 5MB，約可容納 250+ 景點

## 地圖 Marker 系統

### 三種 marker 類型
- **makeSimpleIcon** — zoom < 9 時顯示，14px 小圓點
- **makeNamedIcon** — zoom ≥ 9 未打卡，50px 空圓圈 + 景點名 chip
- **makePhotoIcon** — 已打卡，60px 相片圓圈 + 景點名 chip

### 縮放平滑化
- `map.on('zoom', applyZoomScale)` — 每 frame 更新 CSS transform scale
- `map.on('zoomend', updateMarkerScale)` — 切換 marker 類型（有 mode tracking 避免不必要 setIcon）
- `transform-origin: 50% 100%` — 從地理錨點縮放
- Scale 公式：`Math.max(0.25, Math.min(1, (zoom - 8) / 6))`

### 重要：setIcon 後 opacity 會 reset
每次 `setIcon` 之後必須重新 call `setOpacity`。

### 唔好用 marker.remove()
會破壞 switchSpot，改用 `setOpacity(0)` 隱藏。

## UI 結構
- 底部卡片（`#card`）— 景點資訊、打卡狀態、拍照按鈕
- 印花收集面板（`#stamp-panel`）— 右上角書本 icon 開啟，可向下拉收起
- 成功 modal — 打卡後彈出
- DEV panel — fake GPS 測試用

## 打卡流程
1. 用戶選景點 → `switchSpot(spot)`
2. GPS 定位 → Haversine 距離計算
3. 進入 500m 範圍 → 解鎖相機
4. 拍照 → Canvas 圓形裁剪壓縮
5. `processCheckin(dataUrl)` → 儲存 + 更新地圖 + 彈出 modal
6. 可重新拍照（仍需 GPS 驗證）

## 進度系統
- 縣進度：紅（0%）、黃（部分）、綠（100% = 制覇）
- 全局：都道府県 X/47、全日本名所 X/總數
- `getPrefProgress(prefName)` — 單縣進度 bar
- `getGlobalStatsHtml()` — 全局兩條進度 bar

## 待辦 / 計劃中
- [ ] Export / Import JSON 備份功能（防用戶主動清除資料）
- [ ] GitHub Pages 部署
- [ ] PWA meta tags（加入 iOS 主畫面）
- [ ] 新增更多景點（現有 3 個，目標 188 個）

## 設計原則
- 功能優先，景點未定案前唔加新景點
- 唔用錢：GitHub Pages 免費，唔買 domain
- 以 iOS Safari 為基準（localStorage 5MB 上限）
