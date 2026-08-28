# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 專案概述

純前端專案（無 build 流程），由三個獨立 HTML 頁面組成，透過 Firebase Realtime Database 作為中介層，實現從手機遙控 **Meta Ray-Ban 智慧眼鏡**在內建顯示器上播放 YouTube 影片。

## 架構說明

```
remote.html  ──寫入──▶  Firebase RTDB  ──讀取──▶  index.html (眼鏡端)
                         currentVideo{             (YouTube IFrame API)
                           id, command
                         }
```

- **index.html** — 在 Meta 眼鏡瀏覽器內執行，黑底（眼鏡內等於透明），監聽 Firebase `currentVideo` 節點變化，透過 YouTube IFrame API 切換/控制影片。
- **remote.html** — 手機或電腦的遙控介面，寫入影片 ID 與 play/pause 指令到 Firebase。
- **testKey.html** — 偵測 Meta 眼鏡鏡腿手勢對應的瀏覽器事件（keydown / click / scroll），用於開發階段的輸入訊號測試。

## Firebase 設定

Firebase 使用 CDN v8 SDK（compat 模式），只用到 Realtime Database，無需 Auth。
Firebase 專案：`ytonmetadisplay-default-rtdb.firebaseio.com`
資料結構：`/currentVideo { id: string, command: "play"|"pause" }`

## 部署

靜態網站，透過 GitHub Pages 提供服務：
- 眼鏡端 HUD：`https://gino8246.github.io/ytOnMetaDisplay/`
- 遙控器：`https://gino8246.github.io/ytOnMetaDisplay/remote.html`
- 輸入測試：`https://gino8246.github.io/ytOnMetaDisplay/testKey.html`

無需 build，直接修改 HTML 並推送到 `main` branch 即可更新。
