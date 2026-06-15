# 修正月相灰色陰影圓形剪裁、小測驗隨機化與標題美化

## 新增修改內容

1. **「地球上看到的月相」標題文字改為鮮豔色**：
   原本 `.preview-title` 顏色為暗深灰色 (`#333`)，在深色太空主題中辨識度不佳。我們將其改為鮮豔的亮黃色 (`#ffb74d` 到 `#ffd700`)，以呼應月亮本體色，並在深藍色背景中顯得非常醒目。

2. **圓形陰影剪裁與隨月相移動**：
   在 `drawMoonPhaseHelper` 繪製時，使用 `ctx.clip()` 使任何比例的灰色陰影在邊界處皆為正圓形，並隨著軌道角度 $\theta$ 動態平滑推移。

3. **小測驗題目每次隨機洗牌**：
   每次初始化隨機挑選 5 題，且不重複。

---

## Proposed Changes

### 1. 樣式調整 (標題字體改為鮮豔色)
#### [MODIFY] [index.html](file:///c:/Users/USER/Downloads/Earth-Science/index.html) (約 L149 - L154)
將 `.preview-title` 的文字顏色從 `#333` 修改為鮮豔醒目的金色/黃色 `#ffd700`，並加上文字發光效果：
```diff
         .preview-title {
             font-size: 1.2em;
             font-weight: 600;
-            color: #333;
+            color: #ffd700;
+            text-shadow: 0 0 10px rgba(255, 215, 0, 0.5);
         }
```

### 2. 測驗題目資料集 (新增 `angle` 屬性)
#### [MODIFY] [index.html](file:///c:/Users/USER/Downloads/Earth-Science/index.html) (約 L1223 - L1304)
為所有 `type: 'image'` 的題目補上公轉角度 `angle`。

### 3. 月相繪圖輔助函數 (重構並加入圓形剪裁)
#### [MODIFY] [index.html](file:///c:/Users/USER/Downloads/Earth-Science/index.html) (約 L931 - L1021)
重構 `drawMoonPhase`，並建立通用的 `drawMoonPhaseHelper`。使用 `ctx.clip()` 讓灰色陰影在拖曳轉動時，其輪廓完美限制在月球正圓形邊界內。

### 4. 圖鑑與小測驗畫布調用更新
#### [MODIFY] [index.html](file:///c:/Users/USER/Downloads/Earth-Science/index.html) (約 L1178 - L1220) 與 (約 L1350 - L1400)
修改 `drawPhasePreview` 與 `drawQuizPhase`，使其調用 `drawMoonPhaseHelper` 進行圓形陰影繪製。

### 5. 小測驗隨機抽題與洗牌邏輯
#### [MODIFY] [index.html](file:///c:/Users/USER/Downloads/Earth-Science/index.html) (約 L780 - L787)
新增隨機狀態變數：
```javascript
        let activeQuestions = [];
```

#### [MODIFY] [index.html](file:///c:/Users/USER/Downloads/Earth-Science/index.html) (約 L1306 - L1529)
* 新增 `shuffleArray` 洗牌函數。
* 重構 `initializeQuiz`，每次從洗牌後的題庫中抽取 5 題存入 `activeQuestions`。
* 將 `selectAnswer`、`updateQuizDisplay`、`submitQuiz`、`resetQuiz` 中的 `QUIZ_QUESTIONS` 全數替換為當次測驗專用的 `activeQuestions`。
