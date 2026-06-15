# 修正月相灰色陰影與亮暗方向變化

將月相繪製演算法重構，修復原本模擬器中「灰色陰影無法隨公轉角度正確變化」以及「上弦月與下弦月亮面方向相同（皆為右側亮）」的教學邏輯錯誤。

## 問題原因分析

1. **亮暗面公式錯誤與方向性缺失**：
   在原本的 [index.html](file:///c:/Users/USER/Downloads/Earth-Science/index.html) 中，`drawMoonPhase` 的照亮比例計算為：
   `const illumination = (Math.cos(phaseAngle) + 1) / 2 * 100;`
   這導致在新月（0°）時算出了 100% 亮面，在滿月（180°）時算出了 0% 亮面，與邏輯完全顛倒。
   此外，繪圖邏輯中並未區分月球公轉軌道是在上半部（0°~180°，右側亮）還是下半部（180°~360°，左側亮），這導致不論是上弦月（90°）還是下弦月（270°），畫面中一律顯示為「右半邊亮」，這在天文教學上是嚴重的錯誤（下弦月應為左半邊亮，即 C 形）。

2. **小測驗圖片與答案不符**：
   由於 `drawQuizPhase` 僅使用 `illumination` 百分比參數繪圖，而沒有傳入軌道角度，導致「虧凸月（左半邊亮）」在題目圖片中被畫成了「盈凸月（右半邊亮）」。

---

## 提案修改內容

### 1. 建立通用的月相繪圖輔助函數 `drawMoonPhaseHelper`
將月相繪製邏輯抽象出一個通用函數，依據「軌道公轉角度」來判斷亮面應該朝左還是朝右，並正確繪製晨昏線橢圓：
* **上半月 (0° 到 180°)**：亮面在右側，暗面在左側。依據角度在中央疊加對應寬度的橢圓（灰色或黃色）。
* **下半月 (180° 到 360°)**：亮面在左側，暗面在右側。依據角度在中央疊加對應寬度的橢圓（灰色或黃色）。

### 2. 重構三個主要繪圖入口
* `drawMoonPhase()` (主預覽)：傳入當前公轉角度 `moonAngle`。
* `drawPhasePreview()` (月相圖鑑)：改為傳入 `phase.angle` 代替原本的 `illumination`。
* `drawQuizPhase()` (小測驗)：改為傳入題目定義的 `q.angle`，並更新題目資料集加上 `angle` 屬性。

---

## Proposed Changes

### 1. 擴展小測驗題目資料
#### [MODIFY] [index.html](file:///c:/Users/USER/Downloads/Earth-Science/index.html) (約 L1223 - L1280)
為 `type: 'image'` 的測驗題目加上 `angle` 屬性：
* 第一題（新月）：`angle: 0`
* 第二題（上弦月）：`angle: 90`
* 第三題（滿月）：`angle: 180`
* 第六題（虧凸月）：`angle: 225`

### 2. 重構 JavaScript 繪圖邏輯
#### [MODIFY] [index.html](file:///c:/Users/USER/Downloads/Earth-Science/index.html) (約 L931 - L1021)
重寫 `drawMoonPhase` 並新增 `drawMoonPhaseHelper` 函數。

#### [MODIFY] [index.html](file:///c:/Users/USER/Downloads/Earth-Science/index.html) (約 L1178 - L1220)
修改 `drawPhasePreview` 及其在 `initializePhasesGrid` 中的呼叫方式。

#### [MODIFY] [index.html](file:///c:/Users/USER/Downloads/Earth-Science/index.html) (約 L1345 - L1400)
修改 `drawQuizPhase` 及其在 `initializeQuiz` 中的呼叫方式。

---

## Verification Plan

### Manual Verification
1. 開啟網頁，並切換至「軌道模擬器」分頁。
2. 拖曳月球，觀察地球上看到的月相：
   * 在 **90°** 時，確認為 **右半邊亮 (D形上弦月)**，左半邊為灰色陰影。
   * 在 **270°** 時，確認為 **左半邊亮 (C形下弦月)**，右半邊為灰色陰影。
   * 從 180° 到 360° 拖曳時，確認灰色陰影從右側逐漸擴展。
3. 切換至「月相圖鑑」分頁，確認「下弦月」、「殘月」卡片中的預覽圖亮面在左側。
4. 切換至「小測驗」分頁，確認第 6 題「虧凸月」繪製出來的亮面在左側。
