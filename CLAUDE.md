# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 專案概述

純前端專案（無 build 流程），由三個獨立 HTML 頁面組成，透過 Firebase Realtime Database 作為中介層，實現從手機遙控 **Meta Ray-Ban 智慧眼鏡**在內建顯示器上播放 YouTube 影片。

## 架構說明

```
remote.html  ──寫入──▶  Firebase RTDB  ──讀取──▶  index.html (眼鏡端)
   ▲                     currentVideo{             (YouTube IFrame API)
   └────讀取 position─────  id, timestamp,
                            position, seek       ──── index 回報 position ──┐
                          }                                                 │
                            ▲───────────────────────────────────────────────┘
```

- **index.html** — 在 Meta 眼鏡瀏覽器內執行，黑底（眼鏡內等於透明）。監聽 `currentVideo/id` 切換影片、監聽 `currentVideo/seek` 即時跳轉；播放中每 5 秒（及頁面隱藏時）回報 `currentVideo/position`，載入影片時以 `startSeconds` 續播。
- **remote.html** — 手機或電腦的遙控介面。點播影片（寫 `id`）、內嵌 YouTube 預覽播放器可在手機端先看/拖曳，再送出「跳轉時間」（同片寫 `seek`，新片寫 `set({id, timestamp, position})`）。
- **testKey.html** — 偵測 Meta 眼鏡鏡腿手勢對應的瀏覽器事件（keydown / click / scroll，以及 visibilitychange / blur / focus / pagehide 可見性事件），用於開發階段的輸入訊號與失焦暫停診斷。

## Firebase 設定

Firebase 使用 CDN v8 SDK（compat 模式），只用到 Realtime Database，無需 Auth。
Firebase 專案：`ytonmetadisplay-default-rtdb.firebaseio.com`

資料結構 `/currentVideo`：

| 欄位 | 型別 | 寫入者 | 說明 |
|------|------|--------|------|
| `id` | string | remote | 11 碼 YouTube 影片 ID |
| `timestamp` | number | remote | 點播時間（`Date.now()`） |
| `position` | number | **index（眼鏡）** | 目前播放秒數，播放中每 5 秒 / 頁面隱藏時回報；用於重開續播，remote 送新片時亦用作起始秒數 |
| `seek` | `{ to: number, at: number }` | remote | 即時跳轉指令；index 監聽並 `seekTo(to)`。`at` 為時間戳，確保跳到同一秒也會觸發；index 只讀不寫，且開機時跳過初始快照避免誤跳 |

> 播放/暫停由眼鏡端敲鏡腿（Enter）本地控制，**不經 Firebase**（舊版的 `command` / `status` 欄位已移除）。

安全規則：`currentVideo` 節點需開放讀寫（無 Auth）。測試模式規則會過期，過期後讀寫回傳 401 Permission denied，需至 Firebase Console 更新規則。

## 部署

靜態網站，透過 GitHub Pages 提供服務：
- 眼鏡端 HUD：`https://gino8246.github.io/ytOnMetaDisplay/`
- 遙控器：`https://gino8246.github.io/ytOnMetaDisplay/remote.html`
- 輸入測試：`https://gino8246.github.io/ytOnMetaDisplay/testKey.html`

無需 build，直接修改 HTML 並推送到 `main` branch 即可更新。
