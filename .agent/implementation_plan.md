# 修正月相灰色陰影圓形剪裁與小測驗隨機機制

為了解決月相灰色陰影在邊界處無法完美貼合圓形的問題，並滿足測驗題目每次不重複的需求，我們將進行以下擴充與優化。

## 問題原因與優化方案

1. **陰影邊緣溢出與不夠圓潤**：
   在原本的 Canvas 繪圖中，灰色陰影是使用橢圓直接繪製。由於橢圓沒有被限制在月球的半徑範圍內，在某些亮暗比例下，橢圓的邊緣會超出月球的圓形輪廓，溢出到外部背景中，顯得不夠圓潤。
   * **解決方案**：在繪製月相亮暗面之前，先在 Canvas 上設定一個半徑為 `radius` 的**圓形剪裁路徑 (`ctx.clip()`)**。如此一來，後續繪製的灰色橢圓陰影將會被完美截斷在月球邊緣，與月球邊界百分之百貼合，呈現出完美的圓球陰影效果。

2. **測驗題目每次皆相同**：
   原本的測驗固定以相同的順序顯示全部 8 題，缺乏挑戰性與隨機性。
   * **解決方案**：引入 Fisher-Yates 洗牌演算法，在每次測驗初始化時，將 8 題題庫進行隨機洗牌，並**隨機抽選 5 題**作為本次測驗題目。所有計分與進度條均會根據當次隨機選出的題目動態計算，實現每次測驗皆不重複。

---

## Proposed Changes

### 1. 測驗題目資料集 (新增 `angle` 屬性)
#### [MODIFY] [index.html](file:///c:/Users/USER/Downloads/Earth-Science/index.html) (約 L1223 - L1304)
為所有 `type: 'image'` 的題目補上公轉角度 `angle`：
* 第一題（新月）：`angle: 0`
* 第二題（上弦月）：`angle: 90`
* 第三題（滿月）：`angle: 180`
* 第六題（虧凸月）：`angle: 225`

### 2. 月相繪圖輔助函數 (重構並加入圓形剪裁)
#### [MODIFY] [index.html](file:///c:/Users/USER/Downloads/Earth-Science/index.html) (約 L931 - L1021)
重構 `drawMoonPhase`，並建立通用的 `drawMoonPhaseHelper`。使用 `ctx.clip()` 將亮面與暗面（灰色陰影）完美約束在圓形內。

### 3. 圖鑑與小測驗畫布調用更新
#### [MODIFY] [index.html](file:///c:/Users/USER/Downloads/Earth-Science/index.html) (約 L1178 - L1220) 與 (約 L1350 - L1400)
修改 `drawPhasePreview` 與 `drawQuizPhase`，使其調用 `drawMoonPhaseHelper` 進行圓形陰影繪製。

### 4. 小測驗隨機抽題與洗牌邏輯
#### [MODIFY] [index.html](file:///c:/Users/USER/Downloads/Earth-Science/index.html) (約 L780 - L787)
新增隨機狀態變數：
```javascript
        let activeQuestions = [];
```

#### [MODIFY] [index.html](file:///c:/Users/USER/Downloads/Earth-Science/index.html) (約 L1306 - L1529)
* 新增 `shuffleArray` 洗牌函數。
* 重構 `initializeQuiz`，每次從洗牌後的題庫中抽取 5 題存入 `activeQuestions`。
* 將 `selectAnswer`、`updateQuizDisplay`、`submitQuiz`、`resetQuiz` 中的 `QUIZ_QUESTIONS` 全數替換為當次測驗專用的 `activeQuestions`。

---

## Verification Plan

### Manual Verification
1. **陰影圓形貼合測試**：
   開啟模擬器，緩慢拖曳月球。確認地球月相中，灰色的暗面陰影在月球圓形邊界處完全沒有溢出，邊緣呈現完美的球形圓弧。
2. **小測驗每次不重複測試**：
   * 切換到「小測驗」分頁，點選幾題後按「下一題」，觀察題目內容與類型。
   * 完成測驗或點選「重新作答」，確認新一輪的題目順序與題目內容與上一輪不同，且每次總題數為 5 題。
